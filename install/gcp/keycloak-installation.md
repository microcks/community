# Deploying Keycloak on Google Kubernetes Engine (GKE)

## Overview
This guide walks you through deploying Keycloak on Google Kubernetes Engine (GKE) using Google Cloud SQL database. Keycloak is an open-source identity and access management solution that enables secure authentication and authorization for applications.



## Prerequisites
Ensure the following tools are installed on your local system:

1. **Google Cloud SDK (gcloud):** [Install Guide](https://cloud.google.com/sdk/docs/install)
2. **Helm:** [Install Guide](https://helm.sh/docs/intro/install/)
3. **kubectl (Kubernetes CLI):** [Install Guide](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

---

## 1. Authenticate and Set Up GCP Project

### Authenticate with Google Cloud
```sh
$ gcloud auth login
```

### Create and Configure GCP Project
```sh
$ gcloud projects create <PROJECT-ID> --name="<PROJECT-NAME>"
$ gcloud config set project <PROJECT-ID>
```
For example, if your project name is **cncf-microcks** and project ID is **microcks333**, use these values accordingly.

### Enable Billing (if not already enabled)
```sh
$ gcloud beta billing accounts list
$ gcloud beta billing projects link <PROJECT-ID> --billing-account=<BILLING-ACCOUNT-ID>
```

---

## 2. Create Service Account and Assign Required Roles
If you're working in an enterprise or company environment where you don't have access to an admin account, request the admin user to create the service account and grant the following roles. Once the service account is created and the key file (secret) is provided by the admin/root user, you can authenticate and proceed with the setup as the service account.

### Create the service account for Keycloak deployment
```sh
$ gcloud iam service-accounts create keycloak-deployer \
  --display-name "Keycloak Deployer" \
  --project <PROJECT-ID>
```

### Assign Required Roles
```sh
$ gcloud projects add-iam-policy-binding <PROJECT-ID> \
  --member="serviceAccount:keycloak-deployer@<PROJECT-ID>.iam.gserviceaccount.com" \
  --role="roles/cloudsql.client"

$ gcloud projects add-iam-policy-binding <PROJECT-ID> \
  --member="serviceAccount:keycloak-deployer@<PROJECT-ID>.iam.gserviceaccount.com" \
  --role="roles/cloudsql.admin"

$ gcloud projects add-iam-policy-binding <PROJECT-ID> \
  --member="serviceAccount:keycloak-deployer@<PROJECT-ID>.iam.gserviceaccount.com" \
  --role="roles/container.developer"

$ gcloud projects add-iam-policy-binding <PROJECT-ID> \
  --member="serviceAccount:keycloak-deployer@<PROJECT-ID>.iam.gserviceaccount.com" \
  --role="roles/iam.serviceAccountUser"
```

### Create Service Account Key
```sh
$ gcloud iam service-accounts keys create /path/to/keycloak-deployer-key.json \
  --iam-account=keycloak-deployer@<PROJECT-ID>.iam.gserviceaccount.com
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
  --machine-type e2-medium \
  --num-nodes 2 \
  --disk-size 50 \
  --enable-autoscaling \
  --min-nodes 1 \
  --max-nodes 3 \
  --enable-ip-alias \
  --enable-network-policy \
  --service-account <SERVICE-ACCOUNT>@<PROJECT-ID>.iam.gserviceaccount.com
```

Wait for 7-8 minutes for the cluster to be provisioned and configure node count and disk size based on your usage or team size.

### Authenticate with the Service Accountb

Authenticate using the service account.

```sh
$ gcloud auth activate-service-account --key-file=/path/to/microcks-deployer-key.json
```

## 4. Create a Cloud SQL Instance and Database

### Set the Project to Ensure the Correct Project is Used

```sh
$ gcloud config set project <PROJECT-ID>
```

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

```sh
# Enable Service Networking API
$ gcloud services enable servicenetworking.googleapis.com --project=<PROJECT-ID>

# Allocate IP Range
$ gcloud compute addresses create google-managed-services-default \
  --global \
  --purpose=VPC_PEERING \
  --prefix-length=16 \
  --network=default \
  --project=<PROJECT-ID>

# Create VPC Peering
$ gcloud services vpc-peerings connect \
  --service=servicenetworking.googleapis.com \
  --ranges=google-managed-services-default \
  --network=default \
  --project=<PROJECT-ID>

# Assign Private IP to Cloud SQL
$ gcloud sql instances patch <INSTANCE-NAME> \
  --network=default \
  --assign-ip \
  --project=<PROJECT-ID>

# Get Private IP of Cloud SQL Instance
$ gcloud sql instances describe <INSTANCE-NAME> \
  --format="value(ipAddresses)" \
  --project=<PROJECT-ID>
```

## 6. Deploy Keycloak on GKE with Cloud SQL

### Add Keycloak Helm Repository

```sh
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm repo update
$ kubectl create namespace microcks
```

### Install NGINX Ingress Controller.
```sh
$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
$ helm repo update

$ helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.service.type=LoadBalancer \
  --set controller.config."proxy-buffer-size"="128k"
```

### Get External IP of Ingress Controller Once Available
```sh
$ kubectl get svc -n ingress-nginx ingress-nginx-controller
--- OUTPUT ---
NAME                       TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                      AGE
ingress-nginx-controller   LoadBalancer   34.118.229.104   <INGRESS_IP>   80:31477/TCP,443:31478/TCP    3m
```

### Custom Domain
1. If You Have a Custom Domain Create and A record in your DNS provider to point your domain/subdomain to the INGRESS IP. 
For example:
- keycloak.YOUR-DOMAIN.com pointing to <INGRESS_IP>
- microcks.YOUR-DOMAIN.com pointing to <INGRESS_IP>

2. If you don't have a custom domain, you can use a free domain by using nip.io for your domain names, such as:
- keycloak.<INGRESS_IP>.nip.io
- microcks.<INGRESS_IP>.nip.io

###  Install cert-manager for SSL Certificates
```sh
$ helm repo add jetstack https://charts.jetstack.io
$ helm repo update
$ helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --set installCRDs=true
```

### Create ClusterIssuer for Let's Encrypt
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
### Prepare `keycloak.yaml` Configuration File

```sh
$ cat > keycloak.yaml <<EOF
auth:
  adminUser: admin
  adminPassword: "microcks123"  # Set a strong password

postgresql:
  enabled: false  # Disable embedded PostgreSQL

externalDatabase:
  host: "<CLOUDSQL-PRIVATE-IP>"            # Your Cloud SQL private IP
  port: 5432
  database: "<DATABASE-NAME>"       # Your Cloud SQL database name
  user: "<USER-PASSWORD>"        # Your Cloud SQL username
  password: "<USER-PASSWORD>"      # Your Cloud SQL password
  scheme: "postgresql"

service:
  type: ClusterIP
  ports:
    http: 80  

resources:
  requests:
    cpu: "500m"
    memory: "512Mi"
  limits:
    cpu: "1"
    memory: "1Gi"

ingress:
  enabled: true
  ingressClassName: nginx
  hostname: keycloak.<YOUR-DOMAIN>.com   # Replace <YOUR-DOMAIN> with your custom domain
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"
    cert-manager.io/cluster-issuer: letsencrypt-prod
  tls: true
EOF
```

### Install Keycloak and check Pod Status
```sh
$ helm install keycloak bitnami/keycloak -f keycloak.yaml --namespace microcks
$ kubectl get pods -n microcks
--- OUTPUT ---
NAME                                        READY   STATUS    RESTARTS        AGE
keycloak-0                                  1/1     Running   0               1m
```

### Verify Ingress and Get the Keycloak URL
```sh
$ kubectl get ingress -n microcks
--- OUTPUT ---
NAME       CLASS   HOSTS                           ADDRESS         PORTS     AGE
keycloak   nginx   keycloak.<YOUR-DOMAIN>.com   34.28.102.121      80, 443   3m
```

Keycloak is available at `http://keycloak.<YOUR-DOMAIN>.com`


## 7. Configure Keycloak for your Application

### Step 1: Create a Microcks Realm
- Login to the Keycloak dashboard at `http://keycloak.<YOUR-DOMAIN>.com`
- Click on `Create Realm` in the top left corner and name it `microcks`.

### Step 2: Add a Client for Microcks
- From the left menu, go to `Clients`.
- Click `Create` and enter the `Client ID` (e.g., `microcks-app-js`).
- Enable `Client authentication` and Click `Next`
- In `Valid Redirect URIs`, enter your application URL (e.g., `http://microcks.<YOUR-DOMAIN>.com/*`).
- In `Web Origins`, enter `http://microcks.<YOUR-DOMAIN>.com`.
- If you have not deployed your application yet, you can configure this later.

### Step 3: Create a User
- From the left menu, go to `Users`.
- Click `Add User` and enter a `Username`, `Email` and Click `Save`.
- Go to the `Credentials` tab, set a password, and save it.

🎉 Congratulations! You have successfully deployed Keycloak on Google Kubernetes Engine (GKE) with Cloud SQL database.
