# Deploying Microcks on Azure Kubernetes Service (AKS)

## Overview
This documentation provides a step-by-step guide to deploying **Microcks**, an API mocking and testing solution, on **Azure Kubernetes Service (AKS)**. It also integrates **Keycloak** for authentication via **Azure Database for PostgreSQL**, and utilizes **MongoDB** for storage. This solution allows you to mock APIs, manage microservices effectively, and test with a reliable cloud infrastructure.

---

## Prerequisites
Before you begin, ensure you have the following installed:

1. **Azure CLI**: For managing Azure resources: [Azure CLI Installation Guide](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)

2. **kubectl**: For interacting with your Kubernetes cluster: [kubectl Installation Guide](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

3. **Helm**: A package manager for Kubernetes that will help deploy Keycloak and other resources: [Helm Installation Guide](https://helm.sh/docs/intro/install/)

4. **Docker** (Optional): Useful for local testing and development using Docker Compose: [Install Docker](https://docs.docker.com/get-docker/)

---

## Deployment Steps

## 1. Authenticate and Set Up Azure Resources

### Login to Azure and Create Resource Group
```sh
# Login to Azure
$ az login

# Follow the device login flow (visit https://microsoft.com/devicelogin and enter the provided code).
$ az account set --subscription "<subscription-id>"

# Check your subscription ID
$ az account show --query id -o tsv

# Create the resource group
$ az group create --name microcks-prod-rg --location centralus
```

## 2. Create Service Principal and Assign Required Roles

### Create Service Principal
If you're working in an enterprise or company environment, request the admin user to create the Service Principal and grant the required roles. Once created, note the appId, password, and tenant for the Service Principal.
```sh
$ az ad sp create-for-rbac --name microcks-prod-sp \
    --role contributor \
    --scopes /subscriptions/<subscription-id>/resourceGroups/microcks-prod-rg
```

### Assign Required Roles to the Service Principal
```sh
# Assign Contributor Role at Resource Group Level
$ az role assignment create \
  --assignee <appId> \
  --role "Contributor" \
  --scope /subscriptions/<subscription-id>/resourceGroups/microcks-prod-rg

# Assign Azure Kubernetes Service Contributor Role
$ az role assignment create \
  --assignee <appId> \
  --role "Azure Kubernetes Service Contributor" \
  --scope /subscriptions/<subscription-id>/resourceGroups/microcks-prod-rg
```

### Register the necessary provider for AKS:
```sh
$ az provider register --namespace Microsoft.ContainerService
```

### Login with Service Principal
```sh
$ az login --service-principal \
  --username <appId> \
  --password <password> \
  --tenant <tenant-id>
```

## 3. Create an AKS Cluster

### Create the AKS cluster and wait for 4–5 minutes while the cluster is being provisioned.
```sh
$ az aks create \
  --resource-group microcks-prod-rg \
  --name microcks-prod-aks \
  --location centralus \
  --node-count 2 \
  --node-vm-size Standard_D4s_v3 \
  --node-osdisk-size 50 \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 4 \
  --generate-ssh-keys
```

### Get AKS Credentials
```sh
$ az aks get-credentials \
  --resource-group microcks-prod-rg \
  --name microcks-prod-aks
```

### Check the status of your AKS nodes
```sh
$ kubectl get nodes
```

## 4. Create Azure Database for PostgreSQL for Keycloak

### Create a PostgreSQL server and wait for 3–4 minutes while the server is provisioned
```sh
$ az postgres flexible-server create \
  --resource-group microcks-prod-rg \
  --name microcks-postgres \
  --location centralus \
  --admin-user <admin-username> \
  --admin-password <admin-password> \
  --sku-name Standard_D4s_v3 \
  --tier GeneralPurpose \
  --storage-size 32 \
  --version 13
```

### Create Database
```sh
$ az postgres flexible-server db create \
  --resource-group microcks-prod-rg \
  --server-name microcks-postgres \
  --database-name keycloak
```

### Configure Firewall Rules
```sh
# Allow Azure services to access the PostgreSQL server:
$ az postgres flexible-server firewall-rule create \
  --resource-group microcks-prod-rg \
  --name microcks-postgres \
  --rule-name allow-azure-services \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 0.0.0.0

# Allow your IP to access PostgreSQL:
$ az postgres flexible-server firewall-rule create \
  --resource-group microcks-prod-rg \
  --name microcks-postgres \
  --rule-name allow-my-ip \
  --start-ip-address $(curl -s http://checkip.amazonaws.com) \
  --end-ip-address $(curl -s http://checkip.amazonaws.com)
```

## 5. Deploy MongoDB on AKS

### Add the Bitnami Helm chart repository and install MongoDB:
```sh
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm repo update
$ kubectl create namespace microcks
$ helm install mongodb bitnami/mongodb -n microcks \
  --set architecture=standalone \
  --set persistence.enabled=true \
  --set persistence.size=10Gi \
  --set persistence.storageClass=default \
  --set resources.requests.cpu=500m \
  --set resources.requests.memory=1Gi \
  --set resources.limits.cpu=1 \
  --set resources.limits.memory=2Gi \
  --set auth.enabled=true \
  --set auth.rootPassword=<ROOT_PASSWORD> \
  --set auth.username=<USERNAME> \
  --set auth.password=<PASSWORD> \
  --set auth.database=<DATABASE>
```

### Verify MongoDB deployment:
```sh
$ kubectl get pods -n microcks
--- OUTPUT ---
NAME                       READY   STATUS    RESTARTS   AGE
mongodb-844cbfb466-mvc8g   1/1     Running   0          7m5s
```

### Create the MongoDB connection secret
```sh
$ kubectl create secret generic microcks-mongodb-connection -n microcks \
  --from-literal=username=<USERNAME> \
  --from-literal=password=<PASSWORD>
```

## 6. Deploy Keycloak on AKS with Azure Database for PostgreSQL

Follow the [Keycloak deployment guide for Azure AKS](https://github.com/microcks/community/blob/main/install/azure/keycloak-installation.md#5-deploy-keycloak-on-aks-with-azure-database-for-postgresql) provided by the Microcks community.  
Start from **Step 5** of the document and continue through the remaining steps to deploy Keycloak on your AKS cluster.

Once Keycloak is successfully deployed, complete the following configuration:

- **Create a `microcks` realm** in Keycloak.
- **Set up a `microcks` client** within that realm.
- **Add a `microcks` user** and assign appropriate roles for accessing the Microcks dashboard.

## 7. Deploy Microcks on AKS

## Add the Microcks Helm repository:
```sh
$ helm repo add microcks https://microcks.io/helm
$ helm repo update
```

> **Note:** Make sure you've followed the steps to deploy Keycloak, installed NGINX Ingress Controller, configured cert-manager for SSL certificates, and created a ClusterIssuer.

### Create the configuration file for Microcks
```sh
$ cat > microcks.yaml <<EOF
appName: microcks
ingresses: true

microcks:
  url: microcks.<your-domain>.com
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

  env:
    - name: CORS_REST_ALLOWED_ORIGINS
      value: "https://keycloak.<your-domain>.com"
    - name: CORS_REST_ALLOW_CREDENTIALS
      value: "true"

keycloak:
  enabled: true
  install: false
  url: keycloak.<your-domain>.com   
  realm: microcks
  client:
    id: microcks
    secret: <keycloak-client-secret>

mongodb:
  install: false
  uri: mongodb.microcks.svc.cluster.local
  uriParameters: ?authSource=microcks
  database: microcks
  secretRef:
    secret: microcks-mongodb-connection
    usernameKey: username
    passwordKey: password

ingress:
  enabled: true
  tls: true
EOF
```

### Deploy Microcks
```sh
$ helm install microcks microcks/microcks -n microcks -f microcks.yaml
```

### Verify Pod Status
```sh
$ kubectl get pods -n microcks
--- OUTPUT ---
NAME                                        READY   STATUS    RESTARTS   AGE
mongodb-844cbfb466-lwwx4                    1/1     Running   0          3h2m
keycloak-0                                  1/1     Running   0          2h8m
microcks-69fdd5f4b9-fqpd7                   1/1     Running   0          26s
microcks-postman-runtime-586db88ff9-5djtg   1/1     Running   0          26s
```

### Get Microcks Ingress and Access URL:
```sh
$ kubectl get ingress -n microcks
--- OUTPUT ---
NAME            CLASS   HOSTS                               ADDRESS       PORTS     AGE
keycloak        nginx   keycloak.YOUR-DOMAIN.com            INGRESS-IP    80, 443   2h9m
microcks        nginx   microcks.YOUR-DOMAIN.com            INGRESS-IP    80, 443   1m17s
microcks-grpc   nginx   microcks-grpc.YOUR-DOMAIN.com       INGRESS-IP    80, 443   1m17s
```

Microcks is available at: `https://microcks.<YOUR-DOMAIN>.com` gRPC mock service is available at: `microcks-grpc.<YOUR-DOMAIN>.com`

🎉 **Congratulations! You have successfully deployed Microcks on Azure AKS with PostgreSQL for Keycloak authentication and an external MongoDB. Now, you can start using Microcks to mock and test your APIs in the cloud.**


## 8. Deploy Microcks on GKE with Asynchronous Options

### Add the Microcks Helm repository

```sh
$ helm repo add microcks https://microcks.io/helm
$ helm repo update
```

> **Note:** Make sure you've followed the steps to deploy Keycloak, installed NGINX Ingress Controller, configured cert-manager for SSL certificates, and created a ClusterIssuer.

### Enable SSL Passthrough on NGINX Ingress Controller

```sh
$ kubectl patch -n ingress-nginx deployment/ingress-nginx-controller --type='json' \
  -p '[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--enable-ssl-passthrough"}]'
```

###  Install Strimzi Operator for Kafka Support

```sh
$ kubectl apply -f 'https://strimzi.io/install/latest?namespace=microcks' -n microcks
```

### Prepare microcks-async.yaml Configuration File
```sh
$ cat > microcks-async.yaml <<EOF
appName: microcks
ingresses: true

microcks:
  url: microcks.<your-domain>.com
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
    
  env:
    - name: CORS_REST_ALLOWED_ORIGINS
      value: "https://keycloak.<your-domain>.com"
    - name: CORS_REST_ALLOW_CREDENTIALS
      value: "true"    

keycloak:
  enabled: true
  install: false
  url: https://keycloak.<your-domain>.com
  privateUrl: https://keycloak.<your-domain>.com
  realm: microcks
  client:
    id: microcks  
    secret: <keycloak-client-secret>

mongodb:
  install: false
  uri: mongodb.microcks.svc.cluster.local 
  uriParameters: ?authSource=microcks
  database: microcks
  secretRef:
    secret: microcks-mongodb-connection
    usernameKey: username
    passwordKey: password

features:
  async:
    enabled: true
    kafka:
      install: true
      url: kafka.<your-domain>.com
      ingressClassName: nginx
      ingressAnnotations:
        cert-manager.io/cluster-issuer: "letsencrypt-prod"
        nginx.ingress.kubernetes.io/proxy-body-size: "50m"
        nginx.ingress.kubernetes.io/ssl-passthrough: "true"

ingress:
  enabled: true
  tls: true
EOF
```

### Deploy Microcks
```sh
$ helm install microcks microcks/microcks -n microcks -f microcks-async.yaml
```

### Check the status of the deployed pods
```sh
$ kubectl get pods -n microcks
--- OUTPUT ---
NAME                                             READY   STATUS    RESTARTS    AGE
mongodb-844cbfb466-lwwx4                         1/1     Running   0           4h1m
keycloak-0                                       1/1     Running   0           3h2m
microcks-async-minion-75f456b899-2www8           1/1     Running   4 (28s ago) 2m27s
microcks-f49fc97f9-hfwn7                         1/1     Running   0           2m27s
microcks-kafka-entity-operator-69c49b679b-xkggp  1/2     Running   0           19s
microcks-kafka-kafka-0                           1/1     Running   0           51s
microcks-kafka-zookeeper-0                       1/1     Running   0           95s
microcks-postman-runtime-5c4f488f58-q8fvz        1/1     Running   0           2m27s
strimzi-cluster-operator-5dd46b9985-ct2r9        1/1     Running   0           6m4s
```

### Get Ingress Details
```sh
$ kubectl get ingress -n microcks
```

Microcks is available at: `https://microcks.<YOUR-DOMAIN>.com` gRPC mock service is available at: `microcks-grpc.<YOUR-DOMAIN>.com` Kafka broker is available at: `microcks-kafka.kafka.<YOUR-DOMAIN>.com`

🎉 **Congratulations! You've successfully deployed Microcks on Azure AKS with asynchronous capabilities. You can now begin using Microcks to mock and test your APIs with enhanced asynchronous features in your cloud environment.**

## Cleanup (If Needed)
```sh
## Delete AKS Cluster
$ az aks delete \
  --resource-group microcks-prod-rg \
  --name microcks-prod-aks \
  --yes

# Delete PostgreSQL
$ az postgres flexible-server delete \
  --resource-group microcks-prod-rg \
  --name microcks-postgres \
  --yes
```
> **Note:** Be sure to confirm that you no longer need the resources before performing cleanup.

## Post-Deployment: Contributing to the Community 🤝
We welcome community contributions to improve this guide and keep it updated. If you're an expert in any relevant area, your input can make this guide more valuable for everyone.

**Contribution Areas:**
- Security Best Practices – Replace hardcoded credentials with secure secret management solutions.
- Infrastructure as Code – Convert manual steps into Terraform or other IaC templates.
- Observability – Add guidance for integrating monitoring tools like Prometheus or Grafana.
- Helm Optimization – Share production-ready Helm values for Keycloak.
- High Availability – Suggest multi-region setups or failover strategies.
- Automation – Provide scripts to automate deployment steps.

If you'd like to contribute, please open a [Pull Request](https://github.com/microcks/community/pulls) or submit an [Issue](https://github.com/microcks/community/issues) with your suggestions. Let's work together to make this guide even better! 🙌
