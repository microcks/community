# Deploying Microcks on Google Kubernetes Engine (GKE)

## Overview
This guide provides a step-by-step approach to deploying **Microcks** on **Google Kubernetes Engine (GKE)** using **Google Cloud SQL (PostgreSQL)**, **Firestore**, and integrating with **Keycloak** for authentication.
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

Create a service account for deploying Microcks. Replace `<SERVICE-ACCOUNT-NAME>`, `<SERVICE-ACCOUNT-DESCRIPTION>`, and `<PROJECT-ID>` with your values. For example, the service account name could be `microcks-deployer`, and the project ID could be `microcks333`.

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
  --role="roles/datastore.user"

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
  --iam-account=microcks-deployer@<PROJECT-ID>.iam.gserviceaccount.com
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

### Authenticate with the Service Account

Authenticate using the service account.

```sh
$ gcloud auth activate-service-account --key-file=/path/to/microcks-deployer-key.json
```

## 4. Enable Firestore API and Create a Firestore Database

### Get Kubernetes Credentials for the Cluster and Verify Kubernetes Access

```sh
$ gcloud container clusters get-credentials <CLUSTER-NAME> --zone <CLUSTER-ZONE>
$ kubectl get nodes
```

### Set the Project to Ensure the Correct Project is Used

```sh
$ gcloud config set project <PROJECT-ID>
```

### Enable Firestore API and Create Firestore Database in Native Mode

```sh
$ gcloud services enable firestore.googleapis.com --project=<PROJECT-ID>
$ gcloud firestore databases create --location=us-central1 --project=<PROJECT-ID>
```

## 5. Create a Cloud SQL Instance and Database

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

## 6. Setting Up Private Connectivity Between GKE and Cloud SQL

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

## 7. Deploy Keycloak on GKE with Cloud SQL

Follow the [Keycloak deployment guide for GCP GKE](https://github.com/microcks/community/blob/main/install/gcp/keycloak-installation.md) provided by the Microcks community.

Start from **Step 6** of the document and continue through the remaining steps to deploy Keycloak on your GKE cluster.

Once Keycloak is successfully deployed, complete the following configuration:

- **Create a `microcks` realm** in Keycloak.
- **Set up a `microcks` client** within that realm.
- **Add a `microcks` user** and assign appropriate roles for accessing the Microcks dashboard.


## 8 Deploy Microcks on GKE

### Add Microcks Helm Repository
```sh
$ helm repo add microcks https://microcks.io/helm
$ helm repo update
```

### Prepare `microcks.yaml` Configuration File
```sh
$ cat > microcks.yaml <<EOF
microcks:
  url: "microcks.autoip.nip.io"
  resources: {}

service:
  type: LoadBalancer
  port: 8080
  annotations:
    cloud.google.com/load-balancer-type: "External"

grpc:
  enabled: true
  service:
    type: LoadBalancer
    port: 9090
    annotations:
      cloud.google.com/load-balancer-type: "External"

mongodb:
  enabled: false
  persistence:
    enabled: false

keycloak:
  enabled: false  
  postgresql:
    enabled: false 

identity:
  provider: keycloak
  keycloak:
    url: "http://keycloak.<KEYCLOAK-EXTERNAL-IP>.nip.io" 
    realm: "<YOUR-REALM-NAME>"
    clientId: "<YOUR-CLIENT-ID>"
    clientSecret: "<CLIENT-SECRET>"

postman:
  enabled: true
  resources: {}

env:
  - name: MICROCKS_APP_STORAGE_TYPE
    value: "firestore"
  - name: GCP_PROJECT_ID
    value: "<GCP-PROJECT-ID>"
EOF
```

### Install Microcks on GKE
```sh
$ helm install microcks microcks/microcks -n microcks --create-namespace -f microcks.yaml
```

### Verify Deployment and Check Pod Status
```sh
$ kubectl get pods -n microcks
$ kubectl describe pod <POD-NAME> -n microcks
```

### Get External IPs for Microcks and gRPC
```sh
$ kubectl get svc -n microcks
```

### Update Keycloak Configuration
#### Login to Keycloak and Navigate to Client Configuration
1. Select **Realm Settings** → `microcks`
2. Go to **Clients** → `microcks-app`

### Update Security Settings
```yaml
Valid Redirect URIs:
  http://microcks.<MICROCKS-EXTERNAL-IP>.nip.io/*
  http://microcks-grpc.<GRPC-EXTERNAL-IP>.nip.io/*

Web Origins:
  http://microcks.<MICROCKS-EXTERNAL-IP>.nip.io
  http://microcks-grpc.<GRPC-EXTERNAL-IP>.nip.io
```

### Upgrade Helm Configuration
#### Using nip.io
```sh
$ helm upgrade microcks microcks/microcks -n microcks \
  --set microcks.url=microcks.<MICROCKS-EXTERNAL-IP>.nip.io \
  --set grpc.host=microcks-grpc.<GRPC-EXTERNAL-IP>.nip.io
```

#### Using a Custom Domain
If you have a custom domain, update your DNS provider to create an `A` record pointing to the external IP of Microcks.
```sh
$ helm upgrade microcks microcks/microcks -n microcks \
  --set microcks.url=microcks.<your-custom-domain.com> \
  --set grpc.host=microcks-grpc.<your-custom-domain.com>
```
Ensure your domain's DNS settings correctly map to the external IP of your Microcks service by creating an **A Record** in your DNS provider.

### Access URLs:
- **Microcks UI:** `http://microcks.<MICROCKS-EXTERNAL-IP>.nip.io`
- **gRPC:** `http://microcks-grpc.<GRPC-EXTERNAL-IP>.nip.io`

🎉 **Congratulations! You have successfully deployed Microcks on GKE. Now, you can start using Microcks to mock and test your APIs seamlessly in your cloud environment.**

