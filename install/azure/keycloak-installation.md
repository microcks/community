# Deploying Keycloak on Azure Kubernetes Service (AKS)

## Overview
This guide will walk you through the process of deploying **Keycloak** on **Azure Kubernetes Service (AKS)** and connecting it to **Azure Database for PostgreSQL**. You will also create and configure a **Service Principal (SP)** to securely manage Azure resources from your Kubernetes environment.

---

## Prerequisites
Before you begin, ensure you have the following installed:

1. **Azure CLI**: For managing Azure resources: [Azure CLI Installation Guide](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)

2. **kubectl**: For interacting with your Kubernetes cluster: [kubectl Installation Guide](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

3. **Helm**: A package manager for Kubernetes that will help deploy Keycloak and other resources: [Helm Installation Guide](https://helm.sh/docs/intro/install/)


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
  --node-count 1 \
  --node-vm-size Standard_D4s_v3 \
  --node-osdisk-size 50 \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 2 \
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

## 5. Deploy Keycloak on AKS with Azure Database for PostgreSQL

### Add Keycloak Helm Repository
```sh
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm repo update
```

### Install NGINX Ingress Controller
```sh
$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
$ helm repo update
$ helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"=/healthz \
  --set controller.service.type=LoadBalancer \
  --set controller.config."proxy-buffer-size"="128k"
```

### Get External IP of Ingress Controller
```sh
$ kubectl get svc -n ingress-nginx ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
```

### Custom Domain
1. If You Have a Custom Domain Create and A record in your DNS provider to point your domain/subdomain to the INGRESS IP. 
For example:
- keycloak.YOUR-DOMAIN.com pointing to <INGRESS_IP>
- microcks.YOUR-DOMAIN.com pointing to <INGRESS_IP>

2. If you don't have a custom domain, you can use a free domain by using nip.io for your domain names, such as:
- keycloak.<INGRESS_IP>.nip.io
- microcks.<INGRESS_IP>.nip.io

### Install cert-manager for SSL Certificates
```sh
$ helm repo add jetstack https://charts.jetstack.io
$ helm repo update
$ helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set installCRDs=true
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
    email: <your-email@example.com>
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
EOF
```

### Prepare keycloak.yaml Configuration File
```sh
$ cat > keycloak.yaml <<EOF
auth:
  adminUser: admin
  adminPassword: "microcks123"  # Set a strong password

postgresql:
  enabled: false  # Disable embedded PostgreSQL

externalDatabase:
  host: "microcks-postgres.postgres.database.azure.com"
  port: 5432
  database: "keycloak"
  user: "<admin-username>"       # Your PostgreSQL username
  password: "<admin-password>"   # Your PostgreSQL password
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
  hostname: keycloak.<YOUR-DOMAIN>.nip.io  # Replace with your actual domain
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"
    cert-manager.io/cluster-issuer: letsencrypt-prod
  tls: true
EOF
```

### Install Keycloak and Check Pod Status
```sh
$ helm install keycloak bitnami/keycloak -f keycloak.yaml --namespace microcks
$ kubectl get pods -n microcks
--- OUTPUT  ---
NAME         READY   STATUS    RESTARTS   AGE
keycloak-0   1/1     Running   0          2m2s
```

### Verify Ingress and Get the Keycloak URL
$ kubectl get ingress -n microcks
--- OUTPUT ---
NAME       CLASS   HOSTS                           ADDRESS         PORTS     AGE
keycloak   nginx   keycloak.YOUR-DOMAIN.com   34.28.102.122      80, 443   3m

Keycloak is available at `http://keycloak.<YOUR-DOMAIN>.com`

## 6. Configure Keycloak for your Application

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

🎉 **Congratulations! You have successfully deployed Keycloak on Azure Kubernetes Service (AKS) with with Azure Database for PostgreSQL**
