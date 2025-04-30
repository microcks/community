# Community-Driven Guidelines for Deploying Microcks in Cloud Production Environments

## Overview
This document provides a common guideline for deploying Microcks in production-grade cloud environments using managed Kubernetes services and leveraging cloud-native backing services. It serves as a framework adaptable to various cloud providers (**AWS**, **GCP**, **Azure**, **OVH**, **Oracle**, **Scaleway**, or **Koyeb**, etc.).

The goal is to guide users through the deployment process by emphasizing the use of external Keycloak for authentication, native PostgreSQL-compatible databases for data management, and the deployment of Microcks via Helm charts. While the specific cloud provider implementations may vary (e.g., cloud provider-managed services like databases), the document focuses on the essential steps required, leaving the precise configuration and commands to be adapted based on the user’s cloud provider’s tools, documentation, and community-contributed examples.

## Key Deployment Architecture

This guideline assumes a deployment architecture utilizing:

* **Managed Kubernetes Service:** Your cloud provider's managed Kubernetes offering (e.g., Amazon EKS, Google GKE, Azure AKS).
* **External Keycloak:** Keycloak deployed separately (ideally on the managed Kubernetes cluster) using a managed PostgreSQL-compatible database.
* **Managed PostgreSQL-Compatible Database:** Your cloud provider's managed relational database service configured for PostgreSQL (e.g., Amazon RDS for PostgreSQL, Google Cloud SQL for PostgreSQL, Azure Database for PostgreSQL). This will be used by Keycloak.
* **MongoDB Solution:** A suitable MongoDB solution. This could be:
    * Your cloud provider's managed MongoDB-compatible service (e.g., Amazon DocumentDB for MongoDB Compatibility, Azure Cosmos DB for MongoDB API), *if* fully compatible with Microcks.
    * An external MongoDB instance deployed on your managed Kubernetes cluster (e.g., using a Helm chart like Bitnami's MongoDB chart), if a suitable managed service is not available or preferred.
      
## Prerequisites

Before starting, ensure you have the following tools installed and configured locally:

* **Cloud Provider CLI / SDK:** Necessary for interacting with your specific cloud provider's services (e.g., AWS CLI, gcloud CLI, Azure CLI, etc.). **Authenticate and configure it for your target project/account.**
* **Helm:** Used for deploying Microcks and other components via Helm charts. [Install Helm](https://helm.sh/docs/intro/install/)
* **Kubernetes CLI (kubectl):** Required for managing resources on your Kubernetes cluster. [Install kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
* **Docker (Optional):** Useful for local testing and development using Docker Compose setups. [Install Docker](https://www.docker.com/products/docker-desktop/)

## Deployment Steps

Follow these steps, adapting the specific cloud provider commands and procedures as needed.

## 1. Prepare Cloud Infrastructure and Backing Services

In this section, you will set up the foundational cloud resources required for Microcks using your cloud provider's specific tools and console.

1.  **Authenticate and Set Up Cloud Account/Project:**
    * Authenticate with your cloud provider's CLI/SDK.
    * Create or select the project/account where you will deploy resources.
    * Ensure that billing is enabled for your project/account.

2.  **Create Service Account/Identity and Assign Required Permissions:**
    * Create a dedicated service account or identity that will be used for deploying and managing resources (e.g., Kubernetes cluster creation, database interactions).
    * Assign this identity the necessary permissions to provision and manage:
        * Managed Kubernetes Clusters
        * Managed Database Services (PostgreSQL, MongoDB if used)
        * Networking resources (VPC, subnets, security groups, private endpoints)
        * IAM roles/policies

3.  **Provision Managed Kubernetes Cluster:**
    * Provision a managed Kubernetes cluster in your desired region (e.g., EKS, GKE, AKS).
    * Configure the cluster size, node types, and autoscaling based on your expected load.
    * Ensure the cluster's nodes are associated with the service account/identity created in the previous step, or configure appropriate network access for the nodes.
    * **Configure `kubectl` to connect to your newly created cluster.**

4.  **Provision Managed PostgreSQL-Compatible Database (for Keycloak):**
    * Provision a managed PostgreSQL-compatible database instance (e.g., RDS for PostgreSQL, Cloud SQL for PostgreSQL).
    * Create a dedicated database and user credentials for Keycloak.
    * **Note the database host, port, database name, username, and password.**

5.  **Provision MongoDB Solution (for Microcks):**
    * **Option A: Using a Managed MongoDB-Compatible Service:**
        * Provision your cloud provider's managed MongoDB-compatible database service (e.g., Amazon DocumentDB for MongoDB Compatibility, Azure Cosmos DB for MongoDB API).
        * Ensure it's configured to be accessible from your Kubernetes cluster (see Networking step below).
        * **Note the connection string, database name, username, and password.**
    * **Option B: Deploying External MongoDB on Kubernetes:**
        - If a suitable managed service is not available or preferred, you will deploy a MongoDB instance directly onto your Kubernetes cluster using a standard Helm chart (like Bitnami's). You will configure it to use persistent storage suitable for your cloud provider.
        - Add Bitnami Helm Chart Repository and Install MongoDB

         ```bash
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

6.  **Create MongoDB Connection Secret:**

    Create a Kubernetes `Secret` containing the username and password for your MongoDB instance. Replace the placeholder values with the actual credentials obtained from your cloud provider (for managed     service) or your MongoDB deployment process (for on-K8s deployment).

    ```bash
    $ kubectl create secret generic microcks-mongodb-connection -n microcks \
      --from-literal=username=<YOUR-MONGODB-USERNAME> \
      --from-literal=password=<YOUR-MONGODB-PASSWORD>
    ```

7.  **Establish Private Network Connectivity:**
    * Configure networking to ensure secure, private communication between:
        * Your Kubernetes cluster nodes and the Managed PostgreSQL instance.
        * Your Kubernetes cluster nodes and the Managed MongoDB instance (if using Option A in step 5).
    * This typically involves configuring VPCs, subnets, private endpoints, peering, and security groups/firewalls using your cloud provider's networking tools.

## 2. Deploy Shared Services on Kubernetes

In this section, you deploy common components necessary for Microcks and Keycloak within your Kubernetes cluster using standard Helm and kubectl commands.

1.  **Add Required Helm Repositories:**

    ```bash
    $ helm repo add bitnami 
    $ helm repo add ingress-nginx
    $ helm repo add jetstack 
    $ helm repo add microcks
    
    # Add Strimzi only if planning to deploy asynchronous features with Kafka
    $ helm repo add strimzi
    $ helm repo update
    ```

2.  **Create Kubernetes Namespace:**

    ```bash
    $ kubectl create namespace microcks
    ```
    
3.  **Install Ingress Controller:**
   
    Install an Ingress Controller (e.g., NGINX Ingress Controller) onto your cluster using its standard Helm chart. This will route external traffic to your services. You may need to adjust `controller.service.type` or other parameters based on your cloud.

    ```bash
    $ helm install ingress-nginx ingress-nginx/ingress-nginx \
      --namespace ingress-nginx \
      --create-namespace \
      --set controller.service.type=LoadBalancer \
      --set controller.config."proxy-buffer-size"="128k"
    ```
    * **Note the external IP or hostname assigned to the Ingress Controller's LoadBalancer service.** You can get this using `$ kubectl get svc -n ingress-nginx ingress-nginx-controller`.

5.  **Configure DNS / Domains:**

    Set up DNS records in your domain provider (or use a service like nip.io for testing) to point desired hostnames (e.g., `keycloak.<YOUR-DOMAIN>`, `microcks.<YOUR-DOMAIN>`, `microcks-grpc.<YOUR-DOMAIN>`, `kafka.<YOUR-DOMAIN>` if async is used) to the Ingress Controller's external IP/hostname.

7.  **Install Certificate Manager:**

    ```bash
    $ helm install cert-manager jetstack/cert-manager \
      --namespace cert-manager \
      --create-namespace \
      --set installCRDs=true
    ```

8.  **Create ClusterIssuer:**

    ```bash
    $ cat <<EOF | kubectl apply -f -
    apiVersion: cert-manager.io/v1
    kind: ClusterIssuer
    metadata:
      name: letsencrypt-prod
    spec:
      acme:
        server: https://acme-v02.api.letsencrypt.org/directory
        email: <your-email@example.com>  # Update with your email address
        privateKeySecretRef:
          name: letsencrypt-prod-private-key
        solvers:
        - http01:
            ingress:
              class: nginx # Or the class of your Ingress Controller
    EOF
    ```

## 3. Deploy and Configure External Keycloak

1.  **Prepare Keycloak Helm Values:**

    Create a Helm values file (e.g., `keycloak-values.yaml`) to configure the Bitnami Keycloak Helm chart. Replace the placeholder values with your actual details.

    ```bash
    $ cat > keycloak-values.yaml <<EOF
    auth:
      adminUser: admin
      adminPassword: <KEYCLOAK-ADMIN-PASSWORD> # Set a strong password

    postgresql:
      enabled: false

    externalDatabase:
      host: "<YOUR-MANAGED-POSTGRES-HOST>" # Your Managed PostgreSQL-Compatible Database endpoint or IP
      port: 5432
      database: "<DATABASE-NAME>"
      user: "<USER-PASSWORD>" 
      password: "<USER-PASSWORD>"
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
      ingressClassName: nginx # Or the class of your Ingress Controller
      hostname: keycloak.<YOUR-DOMAIN/IP.nip.io> # Replace with your desired Keycloak hostname
      annotations:
        nginx.ingress.kubernetes.io/backend-protocol: "HTTP"
        cert-manager.io/cluster-issuer: letsencrypt-prod # Or the name of your chosen ClusterIssuer/Issuer
      tls: true
    EOF
    ```

3.  **Install Keycloak using Helm:**

    ```bash
    $ helm install keycloak bitnami/keycloak -n microcks -f keycloak-values.yaml
    ```

4.  **Verify Keycloak Deployment Get Keycloak Ingress UR:**

    ```bash
    $ kubectl get pods -n microcks
    $ kubectl get ingress -n microcks
    ```
    * Access Keycloak via the configured hostname to ensure it's reachable.

5.  **Configure Keycloak for Microcks:**
    * Once Keycloak is successfully deployed and accessible via its domain, complete the following configuration within the Keycloak administration console using the admin credentials defined in your `keycloak-values.yaml`:
        * Create a **`microcks`** realm.
        * Set up a client for Microcks within that realm (e.g., **`microcks-app-js`**). Configure Valid Redirect URIs and Web Origins for your Microcks hostname (e.g., `https://microcks.<YOUR-DOMAIN/IP.nip.io>/*`).
        * Create a user (or users) and assign them appropriate roles (e.g., `user`, `admin`) for accessing the Microcks dashboard within the **`microcks`** realm.

## 4. Deploy Microcks with Default Options

1.  **Prepare Microcks Helm Values:**

    Create a Helm values file (e.g., `microcks-values.yaml`) to configure the Microcks Helm chart. This template uses the `mongodb.secretRef` pattern to securely pass credentials. Replace the placeholder values with your actual details.

    ```bash
    $ cat > microcks-values.yaml <<EOF
    appName: microcks
    ingresses: true

    microcks:
      url: microcks.<YOUR-DOMAIN/IP.nip.io>     # Replace with your desired Microcks hostname
      ingressClassName: nginx       # Or the class of your Ingress Controller
      ingressAnnotations:
        cert-manager.io/cluster-issuer: "letsencrypt-prod"
        nginx.ingress.kubernetes.io/proxy-body-size: "50m"

      grpcEnableTLS: true
      grpcIngressClassName: nginx 
      grpcIngressAnnotations:
        cert-manager.io/cluster-issuer: "letsencrypt-prod"      # Or the name of your chosen ClusterIssuer/Issuer
        nginx.ingress.kubernetes.io/backend-protocol: "GRPC"
        nginx.ingress.kubernetes.io/ssl-passthrough: "true"

      env:
        - name: CORS_REST_ALLOWED_ORIGINS
          value: "https://keycloak.<YOUR-DOMAIN/IP.nip.io>"     # Replace with your Keycloak hostname
        - name: CORS_REST_ALLOW_CREDENTIALS
          value: "true"

    keycloak:
      enabled: true
      install: false
      url: keycloak.<YOUR-DOMAIN/IP.nip.io>     # Replace with your Keycloak hostname (external)
      privateUrl: https://keycloak.<YOUR-DOMAIN/IP.nip.io>
      realm: microcks
      client:
        id: <YOUR-KEYCLOAK-CLIENT-ID> 
        secret: <YOUR-KEYCLOAK-CLIENT-SECRET>
    
    mongodb:
      install: false        # Disable embedded MongoDB
      uri: <YOUR-URI>       # MongoDB-compatible service URI (Replace with your cloud provider's URI)
      uriParameters: <YOUR-URI-PARAMETERS>      # URI Parameters (Modify as needed for your service)
      database: <DATABASE-NAME>
      secretRef:
        secret: microcks-mongodb-connection 
        usernameKey: username
        passwordKey: password 


    ingress:
      enabled: true
      tls: true
    EOF
    ```

2.  **Deploy Microcks:**

    ```bash
    $ helm install microcks microcks/microcks -n microcks -f microcks-values.yaml
    ```

3.  **Verify Pod Status and Get Microcks Ingress URL:**
   
    ```bash
    $ kubectl get pods -n microcks
    $ kubectl get ingress -n microcks
    ```
    
    * Microcks is available at: `microcks.<YOUR-DOMAIN/IP.nip.io>` gRPC mock service is available at: `microcks-grpc.<YOUR-DOMAIN/IP.nip.io>`
    * Access the Microcks UI via your configured hostname and log in using the user created in Keycloak.

## 5. Deploy Microcks with Asynchronous Options

If you require support for asynchronous protocols (like Kafka), follow these additional steps.

1.  **Enable SSL Passthrough on Ingress Controller:**

    Depending on your Ingress Controller configuration and setup, you might need to enable SSL Passthrough for async protocols like Kafka/gRPC that require it. Consult your Ingress Controller's documentation. For NGINX Ingress Controller, this often involves patching the deployment.

    ```bash
    $ kubectl patch -n ingress-nginx deployment/ingress-nginx-controller --type='json' \
      -p '[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--enable-ssl-passthrough"}]'
    ```

2.  **Install Strimzi Operator for Kafka Support:**

    ```bash
    $ kubectl apply -f 'https://strimzi.io/install/latest?namespace=microcks' -n microcks
    ```

3.  **Prepare Microcks Helm Values (for Async):**

    Create a new Helm values file named microcks-async-values.yaml to configure the Microcks Helm chart for asynchronous features This file will contain the complete configuration, including settings from the default deployment and the Kafka configuration.

    ```bash
    $ cat > microcks-async-values.yaml <<EOF
    appName: microcks
    ingresses: true

    microcks:
      url: microcks.<YOUR-DOMAIN/IP.nip.io>     # Replace with your desired Microcks hostname
      ingressClassName: nginx       # Or the class of your Ingress Controller
      ingressAnnotations:
        cert-manager.io/cluster-issuer: "letsencrypt-prod"
        nginx.ingress.kubernetes.io/proxy-body-size: "50m"

      grpcEnableTLS: true
      grpcIngressClassName: nginx 
      grpcIngressAnnotations:
        cert-manager.io/cluster-issuer: "letsencrypt-prod"      # Or the name of your chosen ClusterIssuer/Issuer
        nginx.ingress.kubernetes.io/backend-protocol: "GRPC"
        nginx.ingress.kubernetes.io/ssl-passthrough: "true"

      env:
        - name: CORS_REST_ALLOWED_ORIGINS
          value: "https://keycloak.<YOUR-DOMAIN/IP.nip.io>"     # Replace with your Keycloak hostname
        - name: CORS_REST_ALLOW_CREDENTIALS
          value: "true"

    keycloak:
      enabled: true
      install: false
      url: keycloak.<YOUR-DOMAIN/IP.nip.io>     # Replace with your Keycloak hostname (external)
      privateUrl: https://keycloak.<YOUR-DOMAIN/IP.nip.io>
      realm: microcks
      client:
        id: <YOUR-KEYCLOAK-CLIENT-ID> 
        secret: <YOUR-KEYCLOAK-CLIENT-SECRET>
    
    mongodb:
      install: false        # Disable embedded MongoDB
      uri: <YOUR-URI>       # MongoDB-compatible service URI (Replace with your cloud provider's URI)
      uriParameters: <YOUR-URI-PARAMETERS>      # URI Parameters (Modify as needed for your service)
      database: <DATABASE-NAME>
      secretRef:
        secret: microcks-mongodb-connection 
        usernameKey: username
        passwordKey: password 

    features:
      async:
        enabled: true
        kafka:
          install: true
          url: kafka.<YOUR-DOMAIN/IP.nip.io>        # Replace with your desired Kafka hostname 
          ingressClassName: nginx       # Or the class of your Ingress Controller
          ingressAnnotations:
            cert-manager.io/cluster-issuer: "letsencrypt-prod" 
            nginx.ingress.kubernetes.io/proxy-body-size: "50m"
            # Note: ssl-passthrough is often required for Kafka protocols via Ingress
            nginx.ingress.kubernetes.io/ssl-passthrough: "true"

    ingress:
      enabled: true
      tls: true    
    EOF
    ```

4.  **Deploy Microcks:**
   
    ```bash
    # If upgrading an existing Microcks deployment:
    $ helm upgrade microcks microcks/microcks -n microcks -f microcks-async-values.yaml

    If deploying Microcks for the first time with async enabled:
    $ helm install microcks microcks/microcks -n microcks -f microcks-async-values.yaml
    ```

5.  **Verify Pod Status and Get Microcks Ingress URL:**

    ```bash
    $ kubectl get pods -n microcks
    $ kubectl get ingress -n microcks
    ```

    * Microcks is available at: `microcks.<YOUR-DOMAIN/IP.nip.io>` gRPC mock service is available at: `microcks-grpc.<YOUR-DOMAIN/IP.nip.io>` Kafka broker is available at: `microcks-kafka.kafka.<YOUR-DOMAIN/IP.nip.io>`
    * Access the Microcks UI and confirm asynchronous mocking/testing capabilities.

## Contributing to the Community: Post-Deployment Insights
Sharing your cloud provider-specific deployment scripts, configurations, and lessons learned. Enhance the shared documentation and engage in community discussions to improve the experience for all users.
