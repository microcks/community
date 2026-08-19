# Deploy Microcks on OVHcloud

## Overview

This guide provides a step-by-step approach to deploying **Microcks** on an **OVHcloud Managed Kubernetes Service (MKS)** cluster. It includes setting up **Ingress with Public Cloud Load Balancer**.

## Prerequisites

Ensure the following tools are installed on your local system:

1. **OVHcloud CLI:** [Install Guide](https://github.com/ovh/ovhcloud-cli/#installation)
2. **kubectl (Kubernetes CLI):** [Install Guide](https://kubernetes.io/docs/tasks/tools/#kubectl)
3. **Helm:** [Install Guide](https://helm.sh/docs/intro/install/)
4. An active OVHcloud account
5. An OVHcloud Public Cloud project
6. An OVHcloud API credential with sufficient permissions
7. A Domain Name (optional) if you want to set up a custom DNS

## 1. Authenticate and configure the OVHcloud Public Cloud project

OVHcloud CLI requires authentication to be able to make API calls. Run the following commands to export the needed enviroment variables. Replace <application-key>, <application-secret>, <consumer-key> and <public-cloud-project-id> with your information.

```sh
export OVH_ENDPOINT="ovh-eu"
export OVH_APPLICATION_KEY="<application-key>"
export OVH_APPLICATION_SECRET="<application-secret>"
export OVH_CONSUMER_KEY="<consumer-key>"
export OVH_CLOUD_PROJECT_SERVICE="<public-cloud-project-id>"
```

Alternatively, you can use the `ovhcloud login` command to authenticate interactively or [use a configuration file](https://github.com/ovh/ovhcloud-cli/blob/main/doc/authentication.md#configuration-file).

## 2. Create and configure an OVHcloud MKS cluster

### 2.1 Create an OVHcloud MKS cluster

Configure the Kubernetes cluster information:

For example:

```sh
export CLUSTER_NAME="microcks"
export REGION="GRA9"
export PLAN="free"
```

Create the Kubernetes cluster:

```sh
CLUSTER_ID=$(ovhcloud cloud mks create --name $CLUSTER_NAME --region $REGION --plan $PLAN | grep -oE '[0-9a-f-]{36}')
```

Wait for 2-4 minutes for the cluster to be provisioned.

Check the status of the Kubernetes cluster:

```sh
ovhcloud cloud mks get $CLUSTER_ID
```

Note that for production usage, you should consider using a "standard" plan instead of the free plan.

### 2.2 Create the MKS node pool

Microcks is composed of several Kubernetes workloads, including the Microcks application, Keycloak, MongoDB and the Postman runtime.

For a small installation, a node pool with three general-purpose nodes is a reasonable starting point.

For example:

```sh
export NODEPOOL_NAME="microcks-np"
export NODE_FLAVOR="b3-8"
```

Create the node pool:

```sh
NP_ID=$(ovhcloud cloud mks nodepool create $CLUSTER_ID --flavor-name $NODE_FLAVOR --name $NODEPOOL_NAME --desired-nodes 3 --min-nodes 2 --max-nodes 3 | grep -oE '[0-9a-f-]{36}')
```

Wait for 3-4 minutes for the node pool to be provisioned.

Check the status of the node pool:

```sh
ovhcloud cloud mks nodepool get $CLUSTER_ID $NP_ID
```

### 2.3 Generate the kubeconfig and configure kubectl

Once the cluster and node pool are ready, generate the Kubernetes configuration:

```sh
ovhcloud cloud mks kubeconfig generate $CLUSTER_ID > microcks.yaml
```

Configure the kubectl CLI with the generated kubeconfig:

```sh
export KUBECONFIG=$(pwd)/microcks.yaml
```

Display the node pool and the nodes information:

```sh
kubectl get np
kubectl get nodes
```

You should see several nodes in the `Ready` state.

## 3. Deploy and configure Ingress Controller

### 3.1 Ingress Controller deployment

Install NGINX Ingress Controller:

```sh
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.service.type=LoadBalancer \
  --set controller.config."proxy-buffer-size"="128k"
```

Get External IP of Ingress Controller once available:

```sh
kubectl get svc -n ingress-nginx ingress-nginx-controller
--- OUTPUT ---
NAME                       TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller   LoadBalancer   10.3.252.218   <INGRESS_IP>   80:30624/TCP,443:30574/TCP   3m39s
```

Note: The OVHcloud Public Cloud Load Balancer is creating. It may take a few minutes for the EXTERNAL-IP to be assigned.

### 3.2 Configure DNS for Ingress Controller

If you have a `Custom Domain` create an `A` record in your DNS provider to point your domain/subdomain to the INGRESS IP. For example:

```
keycloak.YOUR-DOMAIN.com pointing to <INGRESS_IP>
microcks.YOUR-DOMAIN.com pointing to <INGRESS_IP>
```

You can do it easily at OVHcloud in the Domain names management console if you have your domain registered with OVHcloud:

![OVHcloud DNS Domain Name](ovhcloud_dns_domain_name.png)

After the creation, wait a little bit for the DNS propagation to be completed. You can check it with the following command:

```sh
dig keycloak.YOUR-DOMAIN.com +noall +answer
dig microcks.YOUR-DOMAIN.com +noall +answer
```

Or, if you don't have a custom domain, you can use a free domain by using `nip.io` for your domain names, such as:

```
keycloak.<INGRESS_IP>.nip.io
microcks.<INGRESS_IP>.nip.io
```

### 3.3 Install cert-manager for SSL Certificates

```sh
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --set installCRDs=true
```

Create `ClusterIssuer` for Let's Encrypt:

```sh
$ cat <<EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: <your-email@example.com>   # Update with your email address
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
EOF
```

## 4. Install Microcks using Helm

### 4.1 Add Microcks Helm Repository

```sh
helm repo add microcks https://microcks.io/helm/
helm repo update
```

### 4.2 Create the configuration file for Microcks

Create the `microcks_values.yaml` file with the configuration below. Replace the YOUR-DOMAIN placeholder with your actual value.

```sh
cat > microcks_values.yaml <<EOF
appName: microcks
ingresses: true

microcks:
  url: microcks.<YOUR_DOMAIN>.com
  ingressClassName: nginx
  ingressAnnotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/proxy-body-size: "50m"
   
  grpcEnableTLS: true
  grpcIngressClassName: nginx
  grpcIngressAnnotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/backend-protocol: "GRPC"
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"  

keycloak:
  url: keycloak.<YOUR_DOMAIN>.com
  privateUrl: https://keycloak.<YOUR_DOMAIN>.com
  ingressClassName: nginx
  ingressAnnotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/proxy-body-size: "50m"
    
ingress:
  enabled: true
  tls: true
EOF
```

### 4.3 Deploy Microcks

```sh
helm install microcks microcks/microcks -n microcks -f microcks_values.yaml --create-namespace
```

### 4.4 Verify Microcks Pod Status

Check that the Microcks pods are running:

```sh
kubectl get pods -n microcks
--- OUTPUT ---
NAME                                           READY   STATUS    RESTARTS       AGE
microcks-7f9f994fbc-jd7pb                      1/1     Running   0              19m
microcks-keycloak-5cf68c6b65-xjr6n             1/1     Running   3 (3m1s ago)   19m
microcks-keycloak-postgresql-6665b755f-zjdrl   1/1     Running   0              19m
microcks-mongodb-7ddff9f544-8rdcx              1/1     Running   0              19m
microcks-postman-runtime-5699859b86-58mr7      1/1     Running   0              19m
```

Wait until all pods are in the `Running` state.


### 4.5 Get Microcks Ingress and Access URL

```sh
kubectl get ingress -n microcks
```

Microcks is now available at: https://microcks.YOUR-DOMAIN.com  gRPC mock service is available at: https://microcks-grpc.YOUR-DOMAIN.com

🎉 Congratulations! You have successfully deployed Microcks on OVHcloud MKS. Now, you can start using Microcks to mock and test your APIs seamlessly in your cloud environment.

## Cleanup (If Needed)

To clean up resources:

1. Delete the MKS cluster:

```sh
ovhcloud cloud mks delete $CLUSTER_ID
```
It will delete the cluster and all associated resources, including node pools, load balancers, and ingress controllers.


Improvement:
* gateway api instead of nginx ingress controller + cert-manager for SSL certificates
* deploy the postgreSQL DB on an OVHcloud managed database service instead of using the default PostgreSQL deployment in the Microcks Helm chart. This will provide better performance, scalability, and reliability for your Microcks installation.
* deploy the mongodb on an OVHcloud managed database service instead of using the default MongoDB deployment in the Microcks Helm chart. This will provide better performance, scalability, and reliability for your Microcks installation.