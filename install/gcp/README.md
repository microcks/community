# Deploying Microcks on Google Kubernetes Engine (GKE)

## Overview
This guide provides a comprehensive, step-by-step process for deploying **Microcks** on **Google Kubernetes Engine (GKE)**, leveraging **Google Cloud SQL (PostgreSQL)** for database management. It also covers the integration of **External MongoDB** for data storage and **Keycloak** for authentication, enabling a secure and scalable API testing environment in the cloud.

## Prerequisites
Before deploying Microcks on GKE, ensure the following tools are installed and configured:

- **Google Cloud SDK (gcloud):** Required for interacting with GCP resources such as GKE, Cloud SQL, and Firestore.  [Install Google Cloud SDK](https://cloud.google.com/sdk/docs/install)

- **Helm:** Used for deploying Microcks via Helm charts.  [Install Helm](https://helm.sh/docs/intro/install/)

- **Kubernetes CLI (kubectl):** Required for managing Kubernetes resources on GKE.  [Install kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

- **Docker (Optional):** Useful for local testing and development using Docker Compose.   [Install Docker](https://docs.docker.com/get-docker/)


## 1. Authenticate and Set Up GCP Project

### Authenticate and Configure GCP Project
Run the following commands to log in, create a project, and set it as active. Replace `<PROJECT-ID>` and `<PROJECT-NAME>` with your details.
```sh
$ gcloud auth login
$ gcloud projects create <PROJECT-ID> --name="<PROJECT-NAME>"
$ gcloud projects list
$ gcloud config set project <PROJECT-ID>
```
For example, if your project name is **cncf-microcks** and project ID is **microcks333**, use these values accordingly.

### Enable Billing for Your Project
Your Google Cloud project must be linked to a billing account. If you haven’t set up billing, follow these steps:
- Go to **Google Cloud Console** → **Billing**.
- Link an existing billing account or create a new one.
- Once set up, link the billing account to your project.

Alternatively, enable billing using the command-line:
```sh
$ gcloud beta billing accounts list
$ gcloud beta billing projects link <PROJECT-ID> --billing-account=<BILLING-ACCOUNT-ID>
$ gcloud beta billing projects describe <PROJECT-ID>
```
Once billing is enabled, you're ready to proceed with the next steps.

## 2. Create Service Account and Assign Required Roles

If you're working in an enterprise or company environment where you don't have access to an admin account, request the admin user to create the service account and grant the following roles. Once the service account is created and the key file (secret) is provided by the admin/root user, you can authenticate and proceed with the setup as the service account.

```sh
$ gcloud iam service-accounts create <SERVICE-ACCOUNT-NAME> \
  --display-name "<SERVICE-ACCOUNT-DESCRIPTION>" \
  --project <PROJECT-ID>
```

### Assign required roles to the service account:

```sh
$ gcloud projects add-iam-policy-binding <PROJECT-ID> \
  --member="serviceAccount:<SERVICE-ACCOUNT-NAME>@<PROJECT-ID>.iam.gserviceaccount.com" \
  --role="roles/cloudsql.client"

$ gcloud projects add-iam-policy-binding <PROJECT-ID> \
  --member="serviceAccount:<SERVICE-ACCOUNT-NAME>@<PROJECT-ID>.iam.gserviceaccount.com" \
  --role="roles/cloudsql.admin"

$ gcloud projects add-iam-policy-binding <PROJECT-ID> \
  --member="serviceAccount:<SERVICE-ACCOUNT-NAME>@<PROJECT-ID>.iam.gserviceaccount.com" \
  --role="roles/serviceusage.serviceUsageAdmin"

$ gcloud projects add-iam-policy-binding <PROJECT-ID> \
  --member="serviceAccount:<SERVICE-ACCOUNT-NAME>@<PROJECT-ID>.iam.gserviceaccount.com" \
  --role="roles/container.developer"

$ gcloud projects add-iam-policy-binding <PROJECT-ID> \
  --member="serviceAccount:<SERVICE-ACCOUNT-NAME>@<PROJECT-ID>.iam.gserviceaccount.com" \
  --role="roles/iam.serviceAccountUser"
```

### Create a Service Account Key
Generate a key file for the service account. Replace `<PROJECT-ID>` with your actual GCP project ID before running the command.

```sh
$ gcloud iam service-accounts keys create /path/to/microcks-deployer-key.json \
  --iam-account=<SERVICE-ACCOUNT-NAME>@<PROJECT-ID>.iam.gserviceaccount.com
```

## 3. Create a GKE Cluster

### Enable Kubernetes Engine API

Before creating a GKE cluster, enable the Kubernetes Engine API. This process takes 1-2 minutes.

```sh
$ gcloud services enable container.googleapis.com --project=<PROJECT-ID>
```

For example, if your project ID is **microcks333**, replace `<PROJECT-ID>` accordingly.

### Create a GKE Cluster

Create a Kubernetes cluster in the specified region with autoscaling enabled.

```sh
$ gcloud container clusters create <CLUSTER-NAME> \
  --zone us-central1-a \
  --machine-type e2-standard-4 \
  --num-nodes 3 \
  --disk-size 50 \
  --enable-autoscaling \
  --min-nodes 2 \
  --max-nodes 4 \
  --enable-ip-alias \
  --enable-network-policy \
  --service-account <SERVICE-ACCOUNT>@<PROJECT-ID>.iam.gserviceaccount.com
```

Wait for 7-8 minutes for the cluster to be provisioned and configure node count and disk size based on your usage or team size.

> **Note:** If you are deploying Microcks **with asynchronous options enabled**, only then run the following two commands:

```bash
# Authenticate your local `kubectl` with the GKE cluster

$ gcloud container clusters get-credentials microcks-cluster \
  --zone us-central1-a \
  --project microcks123

# Grant cluster-admin permissions to the Microcks deployer service account
$ kubectl create clusterrolebinding microcks-deployer-admin \
  --clusterrole=cluster-admin \
  --user="<SERVICE-ACCOUNT-NAME>@microcks123.iam.gserviceaccount.com"
```

### Authenticate with the Service Account

```sh
$ gcloud auth activate-service-account --key-file=/path/to/microcks-deployer-key.json
```

### Configure Access to Your GKE Cluster

```sh
$ gcloud container clusters get-credentials <CLUSTER-NAME> --zone <CLUSTER-ZONE>
$ gcloud config set project <PROJECT-ID>
```

## 4. Create a Cloud SQL Instance and Database

### Create a Cloud SQL Instance

```sh
$ gcloud sql instances create <INSTANCE-NAME> \
  --database-version=POSTGRES_14 \
  --tier=db-f1-micro \
  --region=us-central1 \
  --root-password=<ROOT-PASSWORD> \
  --project=<PROJECT-ID>
```

Wait for 10-11 minutes for the instance to be provisioned.

### Create a Database

```sh
$ gcloud sql databases create <DATABASE-NAME> --instance=<INSTANCE-NAME> --project=<PROJECT-ID>
```

### Create a User

```sh
$ gcloud sql users create <USERNAME> \
  --instance=<INSTANCE-NAME> \
  --password=<USER-PASSWORD> \
  --project=<PROJECT-ID>
```

For example, project ID microcks333, instance name microcks-cloudsql, database name keycloak_db and username keycloak_user.

## 5. Setting Up Private Connectivity Between GKE and Cloud SQL

### Enable Service Networking API

```sh
$ gcloud services enable servicenetworking.googleapis.com --project=<PROJECT-ID>
```

### Allocate IP Range

```sh
$ gcloud compute addresses create google-managed-services-default \
  --global \
  --purpose=VPC_PEERING \
  --prefix-length=16 \
  --network=default \
  --project=<PROJECT-ID>
```

### Create VPC Peering

```sh
$ gcloud services vpc-peerings connect \
  --service=servicenetworking.googleapis.com \
  --ranges=google-managed-services-default \
  --network=default \
  --project=<PROJECT-ID>
```

### Assign Private IP to Cloud SQL

```sh
$ gcloud sql instances patch <INSTANCE-NAME> \
  --network=default \
  --assign-ip \
  --project=<PROJECT-ID>
```

### Get Private IP of Cloud SQL Instance

```sh
$ gcloud sql instances describe <INSTANCE-NAME> \
  --format="value(ipAddresses)" \
  --project=<PROJECT-ID>
```

## 6. Deploy Keycloak on GKE with Cloud SQL

Follow the [Keycloak deployment guide for GCP GKE](https://github.com/microcks/community/blob/main/install/gcp/keycloak-installation.md) provided by the Microcks community.

Start from **Step 6** of the document and continue through the remaining steps to deploy Keycloak on your GKE cluster.

Once Keycloak is successfully deployed, complete the following configuration:

- **Create a `microcks` realm** in Keycloak.
- **Set up a `microcks` client** within that realm.
- **Add a `microcks` user** and assign appropriate roles for accessing the Microcks dashboard.

## 7. Deploy External MongoDB on GKE

### Add Bitnami Helm Chart Repository and Install MongoDB

```sh
$ helm repo add bitnami https://charts.bitnami.com/bitnami

$ helm install mongodb bitnami/mongodb -n microcks \
  --set architecture=standalone \
  --set persistence.enabled=true \
  --set persistence.size=10Gi \
  --set persistence.storageClass=standard \
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

### Verify MongoDB Deployment

```sh
$ kubectl get pods -n microcks
--- OUTPUT ---
NAME                                        READY   STATUS    RESTARTS        AGE
keycloak-0                                  1/1     Running   0               1d2h
mongodb-69d8b98df-q5vl2                     1/1     Running   0               2m
```

### Create the MongoDB Connection Secret

```sh
$ kubectl create secret generic microcks-mongodb-connection -n microcks \
  --from-literal=username=<USERNAME> \
  --from-literal=password=<PASSWORD>
```

## 8. Deploy Microcks on GKE with Default Options

### Add the Microcks Helm repository

```sh
$ helm repo add microcks https://microcks.io/helm
$ helm repo update
```

> **Note:** Make sure you've followed the steps to deploy Keycloak, installed NGINX Ingress Controller, configured cert-manager for SSL certificates, and created a ClusterIssuer.

### Prepare microcks.yaml Configuration File
Create the microcks.yaml file with the configuration below. Replace the placeholders like YOUR-DOMAIN, USERNAME, and PASSWORD with your actual values.

```sh
$ cat > microcks.yaml <<EOF
appName: microcks
ingresses: true

microcks:
  url: microcks.<YOUR-DOMAIN>.com
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
    - name: SPRING_DATA_MONGODB_URI
      value: "mongodb://<USERNAME>:<PASSWORD>@mongodb.microcks.svc.cluster.local:27017/microcks?authSource=microcks"
    - name: CORS_REST_ALLOWED_ORIGINS
      value: "https://keycloak.<YOUR-DOMAIN>.com"
    - name: CORS_REST_ALLOW_CREDENTIALS
      value: "true"    

keycloak:
  enabled: true
  install: false
  url: keycloak.<YOUR-DOMAIN>.com
  privateUrl: https://keycloak.<YOUR-DOMAIN>.com
  realm: microcks
  client:
    id: <YOUR-CLIENT-ID>  
    secret: <CLIENT-SECRET>

mongodb:
  install: false  
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
$ helm install microcks microcks/microcks -n microcks --create-namespace -f microcks.yaml
```

### Verify Pod Status

```sh
$ kubectl get pods -n microcks
--- OUTPUT ---
NAME                                        READY   STATUS    RESTARTS        AGE
keycloak-0                                  1/1     Running   0               2h8m
microcks-5dcc6b6c8c-wxrr6                   1/1     Running   0               3m2s
microcks-postman-runtime-76c4c86775-v66mb   1/1     Running   0               2m4s
mongodb-69d8b98df-q5vl2                     1/1     Running   0               1h2m
```

### Get Microcks Ingress and Access URL

```sh
$ kubectl get ingress -n microcks
--- OUTPUT ---
NAME            CLASS   HOSTS                                 ADDRESS         PORTS     AGE
keycloak        nginx   keycloak.<YOUR-DOMAIN>.com            <INGRESS-IP>    80, 443   2h9m
microcks        nginx   microcks.<YOUR-DOMAIN>.com            <INGRESS-IP>    80, 443   3m17s
microcks-grpc   nginx   microcks-grpc.<YOUR-DOMAIN>.com       <INGRESS-IP>    80, 443   3m17s
```
Microcks is available at: `https://microcks.<YOUR-DOMAIN>.com` gRPC mock service is available at: `microcks-grpc.<YOUR-DOMAIN>.com`

🎉 **Congratulations! You have successfully deployed Microcks on GKE with default options. Now, you can start using Microcks to mock and test your APIs seamlessly in your cloud environment.**


## 9. Deploy Microcks on GKE with Asynchronous Options

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

### Prepare microcks.yaml Configuration File
Create the microcks-kafka.yaml file with the configuration below. Replace the placeholders like YOUR-DOMAIN, USERNAME, and PASSWORD with your actual values.


```sh
$ cat > microcks-kafka.yaml <<EOF
appName: microcks
ingresses: true

microcks:
  url: microcks.<YOUR-DOMAIN>.com
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
    - name: SPRING_DATA_MONGODB_URI
      value: "mongodb://<USERNAME>:<PASSWORD>@mongodb.microcks.svc.cluster.local:27017/microcks?authSource=microcks"
    - name: CORS_REST_ALLOWED_ORIGINS
      value: "https://keycloak.<YOUR-DOMAIN>.com"
    - name: CORS_REST_ALLOW_CREDENTIALS
      value: "true"    

keycloak:
  enabled: true
  install: false
  url: keycloak.<YOUR-DOMAIN>.com
  privateUrl: https://keycloak.<YOUR-DOMAIN>.com
  realm: microcks
  client:
    id: <YOUR-CLIENT-ID>  
    secret: <CLIENT-SECRET>

mongodb:
  install: false  
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
      url: kafka.<YOUR-DOMAIN>.com
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

### Deploy Microcks with Kafka

```sh
$ helm install microcks microcks/microcks -n microcks --create-namespace -f microcks-kafka.yaml
```

### Verify Pod Status

```sh
$ kubectl get pods -n microcks
--- OUTPUT ---
NAME                                              READY   STATUS    RESTARTS       AGE
keycloak-0                                        1/1     Running   0              1h2m
microcks-65dfc96686-bl5lv                         1/1     Running   0              99s
microcks-async-minion-5c8cc4d4db-ttbst            1/1     Running   3 (62s ago)    99s
microcks-kafka-entity-operator-79c49b679b-xkssp   1/2     Running   0              19s
microcks-kafka-kafka-0                            1/1     Running   0              51s
microcks-kafka-zookeeper-0                        1/1     Running   0              95s
microcks-postman-runtime-5fffb86666-bx74t         1/1     Running   0              99s
mongodb-69d8b98df-q5vl2                           1/1     Running   0              6h3m
strimzi-cluster-operator-6bf566db79-bdjd8         1/1     Running   1 (7d1h ago)   4m2m
```

### Get the Ingress details for Microcks and Kafka.

```sh
$ kubectl get ingress -n microcks
```

Microcks is available at: `https://microcks.<YOUR-DOMAIN>.com` gRPC mock service is available at: `microcks-grpc.<YOUR-DOMAIN>.com` Kafka broker is available at: `microcks-kafka.kafka.<YOUR-DOMAIN>.com`

🎉 **Congratulations! You have successfully deployed Microcks on GKE with asynchronous options. Now, you can start using Microcks to mock and test your APIs, with enhanced asynchronous capabilities, in your cloud environment.**
