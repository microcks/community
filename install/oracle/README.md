# Deploying Microcks on Oracle Kubernetes Engine (OKE)

## Overview
This guide provides a step-by-step approach to deploying **Microcks** on **Oracle Container Engine for Kubernetes (OKE)** with **Oracle Database Service for PostgreSQL** for Keycloak and a **self-managed MongoDB instance** for Microcks. It includes setting up **Ingress with OCI Load Balancer**.

## Prerequisites

Ensure the following tools are installed on your local system:

1. **OCI CLI:** [Install Guide](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cliconcepts.htm)
2. **kubectl (Kubernetes CLI):** [Install Guide](https://kubernetes.io/docs/tasks/tools/)
3. **Helm:** [Install Guide](https://helm.sh/docs/intro/install/)
4. **Optional: jq (for JSON parsing):** [Install Guide](https://stedolan.github.io/jq/)

Additionally, ensure you have:
- An active Oracle Cloud Infrastructure (OCI) account with necessary permissions.
- A Virtual Cloud Network (VCN) with at least two public and two private subnets.
- The OKE cluster should be deployed in public subnets for the ingress controller to work properly, and the PostgreSQL DB System should be in a subnet accessible from the OKE cluster (e.g., within the same VCN).

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
    "Allow group Administrators to manage instance-family in tenancy",
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
Follow the [Keycloak deployment guide for OKE](https://github.com/microcks/community/blob/main/install/oracle/keycloak-installation.md#5-deploy-keycloak-on-oke) provided by the Microcks community.

Start from **Step 5** of the document and continue through the remaining steps to deploy Keycloak on your EKS cluster.

Once Keycloak is successfully deployed, complete the following configuration:

- Create a `microcks` **realm** in Keycloak.
- Set up a `microcks` **client** within that realm.
- Add a `microcks` **user** and assign appropriate roles for accessing the Microcks dashboard.

## 6. Deploy External MongoDB on GKE
### 6.1 Add Bitnami Helm Chart Repository and Install MongoDB
```sh
helm repo add bitnami https://charts.bitnami.com/bitnami

helm install mongodb bitnami/mongodb -n microcks \
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

### 6.2 Verify MongoDB Deployment
```sh
$ kubectl get pods -n microcks
--- OUTPUT ---
NAME                                        READY   STATUS    RESTARTS        AGE
keycloak-0                                  1/1     Running   0               12m
mongodb-99cf82l7bb-h2pk09                   1/1     Running   0               4m
```

### 6.3 Create the MongoDB Connection Secret
```sh
kubectl create secret generic microcks-mongodb-connection -n microcks \
  --from-literal=username=<USERNAME> \
  --from-literal=password=<PASSWORD>
```

### 6.4 Allow Network Access
Ensure that the security list in your VCN allows traffic on port 27017 from the OKE cluster's subnet.

## 7. Deploy Microcks using Helm
### 7.1 Add Microcks Helm Repository
```sh
helm repo add microcks https://microcks.io/helm/
helm repo update
```

### 7.2 Prepare microcks.yaml Configuration File
Create the microcks.yaml file with the configuration below. Replace the placeholders like YOUR-DOMAIN, USERNAME, and PASSWORD with your actual values.
```sh
cat > microcks.yaml <<EOF
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

- Replace `<YOUR-DOMAIN>` with your domain or nip.io address.
- Replace `<YOUR-CLIENT-ID>` and `<CLIENT-SECRET>` with the values from your Keycloak client setup.

### 7.3 Deploy Microcks
```sh
helm install microcks microcks/microcks \
    -n microcks \
    -f microcks.yaml
```

### 7.4 Verify Pod Status
Check that the Microcks pods are running:
```sh
kubectl get pods -n microcks
```
Wait until all pods are in the `Running` state.

### 7.5 Get Microcks Ingress and Access URL
```sh
kubectl get ingress -n microcks
```
Microcks is available at: `https://microcks.YOUR-DOMAIN.com`
gRPC mock service is available at: `https://microcks-grpc.YOUR-DOMAIN.com`

🎉 Congratulations! You have successfully deployed Microcks on OKE. Now, you can start using Microcks to mock and test your APIs seamlessly in your cloud environment.

## Cleanup (If Needed)
To clean up resources:
1. Delete the OKE cluster:
```sh
oci ce cluster delete --cluster-id <CLUSTER_OCID> --force
```
2. Delete the PostgreSQL DB System via the OCI Console.
3. Terminate the MongoDB container via the OCI Console.
4. Delete any other resources created, such as load balancers or security lists.

Microcks is now deployed on **Oracle Container Engine for Kubernetes (OKE)** with **Oracle Database Service for PostgreSQL** for Keycloak and a **self-managed MongoDB container** for Microcks, using **Ingress with OCI Load Balancer**. 🚀 Enjoy your SaaS deployment!

---

## 📝 Contributions from the Community 🤝

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