# Deploying Microcks with Helm Chart using External MongoDB (Bitnami)

This guide walks you through the steps to deploy Microcks using an external MongoDB instance instead of the default built-in MongoDB. Using an external MongoDB provides better scalability, easier maintenance, and more control over your database configuration.

## Prerequisites

- A Kubernetes cluster (this guide uses Minikube)
- Helm 3+ installed
- kubectl configured to access your cluster

## Step 1: Start Minikube and Enable Ingress

Start your Minikube cluster:

```shell
$ minikube start
```

Enable the ingress addon if it's not already enabled:

```shell
$ minikube addons enable ingress
```

Verify that the ingress controller is running:

```shell
$ kubectl get pods -n ingress-nginx
```

Then, we’ll need to prepare the /etc/hosts file to access Microcks using an Ingress. Add the line containing microcks.m.minikube.local address. You need to declare 2 host names for both Microcks and Keycloak.

```shell
$ cat /etc/hosts
--- OUTPUT --- 
##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1 microcks.m.minikube.local keycloak.m.minikube.local
255.255.255.255 broadcasthost
::1 localhost
```

## Step 2: Prepare the Environment

Enable the metrics-server addon in Minikube:

```shell
$ minikube addons enable metrics-server
```

Create a dedicated namespace for your Microcks deployment:

```shell
$ kubectl create namespace microcks-external
```

## Step 3: Deploy External MongoDB using Bitnami Helm Chart

Add the Bitnami Helm repository:

```shell
$ helm repo add bitnami https://charts.bitnami.com/bitnami
```

Deploy MongoDB with authentication enabled:

```shell
$ helm install my-mongodb bitnami/mongodb --namespace microcks-external --set auth.enabled=true --set auth.rootPassword=microcks123 --set auth.username=admin --set auth.password=microcks123 --set auth.database=microcks-db
```

Wait for the MongoDB pod to be ready:

```shell
$ kubectl get pods -n microcks-external 

NAME                          READY   STATUS    RESTARTS   AGE                                                  
my-mongodb-7f9885588f-7mbxj   1/1     Running   0          95s
```

## Step 4: Create MongoDB Credentials Secret for Microcks

Create a Kubernetes Secret to store MongoDB credentials that Microcks will use to connect to the database:

```shell
cat << EOF > mongodb-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: external-mongodb-connection
  namespace: microcks-external
  labels:
    app: microcks
    container: mongodb
    group: microcks
type: kubernetes.io/basic-auth
stringData:
  username: "admin"
  password: "microcks123"
EOF
```

Apply the secret:

```shell
$ kubectl apply -f mongodb-secret.yaml -n microcks-external
```

## Step 5: Deploy Microcks with External MongoDB

Add the Microcks Helm repository if you haven't already:

```shell
$ helm repo add microcks https://microcks.io/helm
$ helm repo update
```

Install Microcks with external MongoDB configuration:

```shell
$ helm install microcks microcks/microcks --namespace microcks-external --set mongodb.install=false --set mongodb.uri=my-mongodb.microcks-external.svc.cluster.local:27017 --set mongodb.database=microcks-db --set mongodb.secretRef.secret=external-mongodb-connection --set mongodb.secretRef.usernameKey=username --set mongodb.secretRef.passwordKey=password --set microcks.url=microcks.m.minikube.local --set keycloak.url=keycloak.m.minikube.local --set keycloak.privateUrl=http://microcks-keycloak.microcks-external.svc.cluster.local:8080
```


## Step 6: Enable Minikube Ingress Access

Start the Minikube tunnel to enable access to ingress resources:

```shell
$ minikube tunnel
```

Keep this terminal window open while accessing Microcks.

## Step 7: Verify the Installation

Check that all pods are running correctly:

```shell
$ kubectl get pods -n microcks-external

NAME                                          READY   STATUS    RESTARTS      AGE
microcks-5cf86dc654-47r6q                     1/1     Running   0             4m17s
microcks-keycloak-5cfb6fc9c-phbx5             1/1     Running   1             4m17s
microcks-keycloak-postgresql-8b6bddc5-7n2sf   1/1     Running   0             4m17s
microcks-postman-runtime-b848f4489-8wxz4      1/1     Running   0             4m17s
my-mongodb-7f9885588f-7mbxj                   1/1     Running   0             6m5s
```


## Step 8: Access Microcks

1. First, navigate to `https://keycloak.m.minikube.local` to accept any self-signed certificates
2. Then access Microcks at `https://microcks.m.minikube.local`
3. Log in using the default credentials:
   - Username: `admin`
   - Password: `microcks123`

## Accessing External MongoDB

To access MongoDB directly from within the cluster:

```shell
$ kubectl exec -it -n microcks-external my-mongodb-[pod-id] -- mongosh admin --authenticationDatabase admin -u root -p microcks123
```

Replace `[pod-id]` with the actual pod identifier.


## Troubleshooting

If Microcks cannot connect to MongoDB:

1. Verify the MongoDB URI is correct:
   ```shell
   $ kubectl get svc -n microcks-external my-mongodb
   ```

2. Check that the secret was created correctly:
   ```shell
   $ kubectl describe secret external-mongodb-connection -n microcks-external
   ```

3. Check the Microcks logs for connection errors:
   ```shell
   $ kubectl logs -n microcks-external deployment/microcks
   ```

## Uninstalling

To uninstall Microcks and the external MongoDB:

```shell
# Uninstall Microcks
$ helm uninstall microcks --namespace microcks-external

# Uninstall MongoDB
$ helm uninstall my-mongodb --namespace microcks-external

# Delete the namespace to clean up everything
$ kubectl delete namespace microcks-external
```
