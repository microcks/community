# Deploying Keycloak on Oracle Kubernetes Engine (OKE)

## Overview
This guide walks you through deploying **Keycloak** on **Oracle Container Engine for Kubernetes (OKE)** using an **Oracle PostgreSQL Database Service** instance as the database. Keycloak is an open-source identity and access management solution that enables secure authentication and authorization for applications.

---

## Prerequisites

Ensure the following tools are installed on your local system:

1. **OCI CLI:** [Install Guide](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cliconcepts.htm)
2. **kubectl (Kubernetes CLI):** [Install Guide](https://kubernetes.io/docs/tasks/tools/)
3. **Helm:** [Install Guide](https://helm.sh/docs/intro/install/)
4. **Optional: jq (for JSON parsing):** [Install Guide](https://stedolan.github.io/jq/)

Additionally, ensure you have a Virtual Cloud Network (VCN) set up in OCI with public and private subnets. The OKE cluster should be deployed in public subnets for the ingress controller to work properly, and the PostgreSQL DB System should be in a subnet accessible from the OKE cluster (e.g., within the same VCN).

---

## 1. Configure OCI Credentials
Configure your OCI CLI credentials by running:
```sh
oci setup config
```
Follow the prompts to enter your:

- Tenancy OCID
- User OCID
- Region (e.g., `us-ashburn-1`)
- API key details

This creates a configuration file (typically at `~/.oci/config`) that allows you to interact with OCI services.

## 2. Create Necessary Permissions
Create a policy in OCI Identity and Access Management (IAM) to grant permissions for managing OKE clusters, PostgreSQL Database Service, networking, and load balancers. Example policy:
```json
{
  "statements": [
    "Allow group Administrators to manage container-engine in tenancy",
    "Allow group Administrators to manage autonomous-database-family in tenancy",
    "Allow group Administrators to manage virtual-network-family in tenancy",
    "Allow group Administrators to manage load-balancers in tenancy"
  ]
}
```

- Replace `Administrators` with your actual group name.
- Apply this policy via the OCI Console under **Identity & Security > Policies** or using the OCI CLI.

This ensures your user or group has the necessary permissions to create and manage resources.

## 3. Create an OKE Cluster

### Step 1: Create OKE Cluster
Create a basic OKE cluster using the OCI CLI:
```sh
oci ce cluster create \
  --compartment-id <COMPARTMENT_OCID> \
  --name <CLUSTER_NAME> \
  --kubernetes-version <K8S_VERSION> \
  --vcn-id <VCN_OCID> \
  --type BASIC_CLUSTER
```
- Replace `<COMPARTMENT_OCID>` with your compartment ID.
- Replace `<CLUSTER_NAME>` with a name (e.g., `keycloak-cluster`).
- Replace `<K8S_VERSION>` with a supported Kubernetes version (e.g., `v1.33.1` – check availability in your region).
- Replace `<VCN_OCID>` with your VCN’s OCID.

### Step 2: Create Node Pool
Add a node pool to the cluster:
```sh
oci ce node-pool create \
  --compartment-id <COMPARTMENT_OCID> \
  --cluster-id <CLUSTER_OCID> \
  --name <NODE_POOL_NAME> \
  --node-shape <NODE_SHAPE> \
  --node-image-name <IMAGE_NAME> \
  --size 2
```

- Replace `<CLUSTER_OCID>` with the OCID of the cluster created in Step 1.
- Replace `<NODE_POOL_NAME>` with a name (e.g., `keycloak-nodes`).
- Replace `<NODE_SHAPE>` with a shape (e.g., `VM.Standard.E4.Flex`).
- Replace `<IMAGE_NAME>` with an available image (e.g., `Oracle-Linux-8.x-<date>` – list images with `oci ce node-pool-options get --cluster-option-id all`).

Wait for the cluster and node pool to provision (this may take 10-15 minutes).

### Step 3: Configure kubectl
Generate a kubeconfig file to interact with the OKE cluster:
```sh
oci ce cluster create-kubeconfig \
  --cluster-id <CLUSTER_OCID> \
  --file $HOME/.kube/config \
  --region <REGION> \
  --token-version 2.0.0
```

- Replace `<REGION>` with your region (e.g., `us-ashburn-1`).


### Step 4: Verify Cluster
Check that the nodes are ready:
```sh
kubectl get nodes
```
You should see your nodes listed with a status of `Ready`.

## 4. Set Up Oracle Database Service for PostgreSQL for Keycloak
### Step 1: Create a PostgreSQL Database Instance
Use the OCI Console or CLI to create a PostgreSQL database instance. For simplicity, use the Console steps:
1. Navigate to **Oracle Database > PostgreSQL > DB Systems**.
2. Click **Create DB System**.
3. Choose your compartment.
3. Select **PostgreSQL** as the database type.
4. Configure the DB system:
    - **Name**: e.g., `keycloak-postgres`
    - **Shape**: Choose an appropriate shape (e.g., `VM.Standard.E4.Flex`).
    - **Storage**: Set storage size (e.g., 50 GB).
    - **Networking**: Select your VCN and subnet.
    - **Administrator Credentials**: Set a username (e.g., `admin`) and password.

6. Click Create.

Wait for the DB System to provision (this may take 10-15 minutes). Note the endpoint (e.g., `10.0.1.x`) from the OCI Console.

### Step 2: Create Database for Keycloak
Connect to the PostgreSQL DB System using a PostgreSQL client (e.g., from a compute instance in the same VCN or via a bastion host):
```sh
psql -h <POSTGRES_ENDPOINT> -U <ADMIN_USERNAME> -d postgres
```
Run the following SQL commands to create a database and user for Keycloak:
```sh
CREATE DATABASE keycloak_db;
CREATE USER 'keycloak_user'@'%' IDENTIFIED BY 'keycloak_password';
GRANT ALL PRIVILEGES ON keycloak_db.* TO 'keycloak_user'@'%';
FLUSH PRIVILEGES;
```

- Replace `<POSTGRES_ENDPOINT>` with the DB System’s endpoint.
- Use the admin password when prompted.

## 5. Deploy Keycloak on OKE
### Add Keycloak Helm Repository
Add the Bitnami Helm repository and create a namespace:
```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
kubectl create namespace microcks
```
### Install NGINX Ingress Controller
Install the NGINX Ingress Controller to expose services:
```sh
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.service.type=LoadBalancer \
  --set controller.config."proxy-buffer-size"="128k"
```
### Get External IP of Ingress Controller Once Available
Wait for the LoadBalancer to provision, then retrieve the external IP:
```sh
kubectl get svc -n ingress-nginx ingress-nginx-controller
```
External-IP is your's INGRESS_IP.
If you don't have a custom domain, you can use a free domain by using nip.io for your domain names, such as:
- `keycloak.<INGRESS_IP>.nip.io`
- `microcks.<INGRESS_IP>.nip.io`

### Install cert-manager for SSL Certificates
Install cert-manager to manage TLS certificates:
```sh
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set installCRDs=true
```
### Create ClusterIssuer for Let's Encrypt
Create a `ClusterIssuer` for Let’s Encrypt:
```sh
cat <<EOF | kubectl apply -f -
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

### Update your ingress configuration
```sh
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: keycloak
  namespace: microcks
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/configuration-snippet: |
      proxy_set_header X-Forwarded-Proto $scheme;
      proxy_set_header X-Forwarded-Host $host;
spec:
  ingressClassName: nginx
  rules:
  - host: "keycloak.<YOUR-DOMAIN>.com"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: keycloak
            port:
              number: 80
  tls:
  - hosts:
    - "keycloak.<YOUR-DOMAIN>.com"
    secretName: keycloak-tls
EOF
```

### Prepare keycloak.yaml Configuration File
```sh
cat > keycloak.yaml <<EOF
auth:
  adminUser: admin
  adminPassword: "microcks123"   # Replace with a strong password

postgresql:
  enabled: false

externalDatabase:
  host: "<POSTGRES_ENDPOINT>"    # Replace with your POSTGRES DB System endpoint
  port: 5432
  database: "keycloak_db"
  user: "microcks"
  password: "microcks123"
  scheme: "postgres"

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

persistence:
  enabled: true
  storageClass: "oci-bv"       # Adjust based on your OKE storage class
  accessModes:
    - ReadWriteOnce
  size: 8Gi

ingress:
  enabled: true
  ingressClassName: nginx
  hostname: keycloak.<YOUR-DOMAIN>.com  # Replace <YOUR-DOMAIN> with your custom domain
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"
    cert-manager.io/cluster-issuer: letsencrypt-prod
  tls: true

EOF
```
- Replace `<POSTGRES_ENDPOINT>` with your POSTGRE SQL DB System’s endpoint.
- Update the `hostname` with your domain or `nip.io` address.
- Use Kuberenetes Secrets for variables like `adminUser`, `adminPassword`, `database`, `user`, `password` in production deployment to ensure security.
- Ensure persistence is enabled to retain Keycloak data across pod restarts. The `persistence` block in your values file handles this by provisioning a PersistentVolumeClaim.


### Custom Domain
1. If You Have a Custom Domain Create and A record in your DNS provider to point your domain/subdomain to the INGRESS IP. For example:
- keycloak.YOUR-DOMAIN.com pointing to <INGRESS_IP>
- microcks.YOUR-DOMAIN.com pointing to <INGRESS_IP>
2. If you don't have a custom domain, you can use a free domain by using nip.io for your domain names, such as:
- keycloak.<INGRESS_IP>.nip.io
- microcks.<INGRESS_IP>.nip.io

### Install Keycloak
Deploy Keycloak using Helm:
```sh
helm install keycloak bitnami/keycloak -f keycloak.yaml -n microcks
```

### Verify Deployment
Check that the Keycloak pod is running:
```sh
kubectl get pods -n microcks
```
Wait until the pod status is `Running`.

### Verify Ingress and Access Keycloak
```sh
kubectl get ingress -n microcks
```
Get host URL.
Keycloak is available at `https://keycloak.<YOUR-DOMAIN>.com`

## 📁 Deployment File Structure
Here’s a suggested structure for your deployment files:
```sh
.
├── keycloak.yaml   # Helm values for Keycloak (admin credentials, DB config, ingress, etc.)
├── keycloak-ingress.yaml   # Optional: Separate ingress if not using inline in values file
└── cert-manager-issuer.yaml    # ClusterIssuer YAML for Let's Encrypt ACME
```
Organizing your files this way improves maintainability and clarity for other contributors or automation scripts.

## 6. Configure Keycloak for your Application
### Step 1: Create a Microcks Realm
- Login to the Keycloak dashboard at `https://keycloak.<YOUR-DOMAIN>.com`
- Click on `Create Realm` in the top left corner and name it `microcks`.

### Step 2: Add a Client for Microcks
- From the left menu, go to `Clients`.
- Click `Create` and enter the `Client ID` (e.g., `microcks-app-js`).
- Enable `Client authentication` and Click `Next`.
- In `Valid Redirect URIs`, enter your application URL (e.g., `http://microcks.<YOUR-DOMAIN>.com/*`).
- In `Web Origins`, enter `http://microcks.<YOUR-DOMAIN>.com`.
- If you have not deployed your application yet, you can configure this later.

### Step 3: Add a User
- From the left menu, go to `Users`.
- Click `Add User` and enter a `Username`, `Email` and Click `Save`.
- Go to the `Credentials` tab, set a `password`, and save it.

🎉 Congratulations! You have successfully deployed Keycloak on Oracle Container Engine for Kubernetes (OKE) with Oracle PostgreSQL Database Service.

---

## 🤝 Community Contributions

We welcome community contributions to enhance this guide and keep it up to date with best practices.

Whether you're an experienced DevOps engineer, a security enthusiast, or someone passionate about documentation, your input can make this deployment guide even more valuable for the community.

### Areas Where You Can Contribute:

- 🔐 **Security Improvements** – Replace hardcoded credentials with secure secret management (e.g., OCI Vault).
- 🏗️ **Infrastructure as Code** – Convert manual steps into Terraform or OCI Resource Manager stacks.
- 📈 **Observability Enhancements** – Add integration guidance for monitoring tools like Prometheus, Grafana, or OCI Monitoring.
- ⚙️ **Helm Configuration Tuning** – Suggest production-optimized values or best practices for Keycloak.
- 🌐 **High Availability** – Propose improvements for multi-region setups or failover strategies.
- 🤖 **Automation Scripts** – Provide shell or Python scripts to automate key deployment steps.

If you’d like to contribute, please open a [Pull Request](https://github.com/microcks/community/tree/main/install/oracle) or file an [Issue](https://github.com/microcks/community/issues) with your suggestions. You can also contribute directly to the [Microcks documentation](https://microcks.io/documentation/) by improving guides, tutorials, or adding new content.

Together, we can make this deployment guide more reliable, secure, and accessible to everyone.

---
