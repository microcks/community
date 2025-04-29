# Community-Driven Guidelines for Deploying Microcks in Cloud Production Environments

## Overview
This document provides a common guideline for deploying Microcks in production-grade cloud environments using managed Kubernetes services and leveraging cloud-native backing services. It serves as a framework adaptable to various cloud providers (**AWS**, **GCP**, **Azure**, **OVH**, **Oracle**, **Scaleway**, or **Koyeb**, etc.).

The goal is to guide users through the deployment process by emphasizing the use of external Keycloak for authentication, native PostgreSQL-compatible databases for data management, and the deployment of Microcks via Helm charts. While the specific cloud provider implementations may vary (e.g., cloud provider-managed services like databases), the document focuses on the essential steps required, leaving the precise configuration and commands to be adapted based on the user’s cloud provider’s tools, documentation, and community-contributed examples.

## Prerequisites

Before starting, ensure you have the following tools installed and configured locally:

* **Cloud Provider CLI / SDK:** Necessary for interacting with your specific cloud provider's services (e.g., AWS CLI, gcloud CLI, Azure CLI, etc.). **Authenticate and configure it for your target project/account.**
* **Helm:** Used for deploying Microcks and other components via Helm charts. [Install Helm](https://helm.sh/docs/intro/install/)
* **Kubernetes CLI (kubectl):** Required for managing resources on your Kubernetes cluster. [Install kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
* **Docker (Optional):** Useful for local testing and development using Docker Compose setups. [Install Docker](https://www.docker.com/products/docker-desktop/)

## Deployment Steps

Follow these steps, adapting the specific commands and procedures to your chosen cloud provider.

## 1. Prepare Cloud Infrastructure and Backing Services

In this section, you will set up the foundational cloud resources required for Microcks.

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
        * If a suitable managed service is not available or preferred, you will deploy a MongoDB instance directly onto your Kubernetes cluster using a standard Helm chart (like Bitnami's).
        * Refer to the Microcks community documentation section on **Deploy External MongoDB on GKE** for detailed Helm command parameters and common configurations. You will adapt these instructions to your specific cloud provider's managed Kubernetes environment.

6.  **Establish Private Network Connectivity:**
    * Configure networking to ensure secure, private communication between:
        * Your Kubernetes cluster nodes and the Managed PostgreSQL instance.
        * Your Kubernetes cluster nodes and the Managed MongoDB instance (if using Option A in step 5).
    * This typically involves configuring VPCs, subnets, private endpoints, peering, and security groups/firewalls using your cloud provider's networking tools.

## 2. Deploy Shared Services on Kubernetes

In this section, you deploy common components necessary for Microcks and Keycloak within your Kubernetes cluster.

1.  **Add Required Helm Repositories:**
    * Add the Helm repositories for necessary charts like `bitnami`, `ingress-nginx`, `microcks`, and `strimzi` (if deploying asynchronous features).

2.  **Create Kubernetes Namespace:**
    * Create a dedicated Kubernetes namespace for your Microcks and related deployments (e.g., `microcks`).

3.  **Install Ingress Controller:**
    * Install an Ingress Controller (e.g., NGINX Ingress Controller) onto your cluster using its standard Helm chart. This will route external traffic to your services.
    * **Note the external IP or hostname assigned to the Ingress Controller's LoadBalancer service.**

4.  **Configure DNS / Domains:**
    * Set up DNS records in your domain provider (or use a service like nip.io for testing) to point desired hostnames (e.g., `keycloak.<YOUR-DOMAIN>`, `microcks.<YOUR-DOMAIN>`, `microcks-grpc.<YOUR-DOMAIN>`, `kafka.<YOUR-DOMAIN>` if async is used) to the Ingress Controller's external IP/hostname.

5.  **Install Certificate Manager:**
    * Install `cert-manager` using its standard Helm chart. This will automate the provisioning and management of TLS certificates.

6.  **Create ClusterIssuer:**
    * Configure a `ClusterIssuer` or `Issuer` resource for `cert-manager` (e.g., using Let's Encrypt) to automatically issue certificates for your domains.

## 3. Deploy and Configure External Keycloak

In this section, you deploy Keycloak using the managed PostgreSQL database provisioned earlier.

1.  **Deploy Keycloak using Helm:**
    * Follow the [Keycloak deployment guide](https://github.com/microcks/community/blob/main/install/gcp/keycloak-installation.md#6-deploy-keycloak-on-gke-with-cloud-sql) provided by the Microcks community. Start from Step 6 of that document and continue through the remaining deployment steps to deploy Keycloak on your cluster using the managed PostgreSQL database and the shared services (Ingress, Cert-Manager) installed in the previous section. You will adapt the Helm values and commands to your specific cloud setup.

2.  **Verify Keycloak Deployment:**
    * Check the status of the Keycloak pods to ensure they are running.
        ```bash
        kubectl get pods -n microcks
        ```
    * Verify the Keycloak Ingress resource and get its URL.
        ```bash
        kubectl get ingress -n microcks
        ```
    * Access Keycloak via the configured hostname to ensure it's reachable.

3.  **Configure Keycloak for Microcks:**
    * Once Keycloak is successfully deployed and accessible via its domain, complete the following configuration within the Keycloak administration console:
        * Create a `microcks` realm.
        * Set up a client for Microcks within that realm (e.g., `microcks-app-js`). Configure redirect URIs and web origins for your Microcks hostname.
        * Create a user (or users) and assign them appropriate roles for accessing the Microcks dashboard within the `microcks` realm.

## 4. Deploy Microcks with Default Options

Deploy Microcks, connecting it to the external Keycloak and your MongoDB solution.

1.  **Create MongoDB Connection Secret (If deploying MongoDB on Kubernetes):**
    * If you chose Option B for your MongoDB solution (deploying on Kubernetes in section 1, step 5), create a Kubernetes `Secret` containing the credentials for your MongoDB instance. This secret will be referenced in the Microcks Helm values.

2.  **Prepare Microcks Helm Values:**
    * Create a Helm values file (e.g., `microcks-values.yaml`) to configure the Microcks Helm chart.
    * Configure the `microcks.url`, `ingressClassName`, and ingress annotations (referencing your cert-manager ClusterIssuer).
    * Disable the embedded Keycloak and MongoDB (`keycloak.install: false`, `mongodb.install: false`).
    * Configure the external Keycloak details (`keycloak.enabled: true`, `keycloak.url`, `keycloak.privateUrl`, `keycloak.realm`, `keycloak.client.id`, `keycloak.client.secret`).
    * Configure the MongoDB connection:
        * If using a managed service (Option A in section 1, step 5), provide the connection string using `env: SPRING_DATA_MONGODB_URI`.
        * If using MongoDB on Kubernetes (Option B in section 1, step 5), reference the connection secret using `mongodb.secretRef`.
    * Configure CORS settings if necessary (`CORS_REST_ALLOWED_ORIGINS`).
    * Configure gRPC ingress if needed (`grpcEnableTLS`, `grpcIngressClassName`, `grpcIngressAnnotations`).

3.  **Deploy Microcks with Default Options:**
    * Deploy Microcks using the Helm chart and your `microcks-values.yaml`.
    * Refer to the Microcks community documentation section on [Deploying Microcks with Default Options](https://github.com/microcks/community/blob/main/install/gcp/README.md#8-deploy-microcks-on-gke-with-default-options) for detailed Helm command parameters and common configurations (e.g., similar to Step 8 in the GKE example). You will adapt these instructions to your specific cloud environment.

4.  **Verify Microcks Deployment (Default):**
    * Check the status of the Microcks and Postman Runtime pods.
        ```bash
        kubectl get pods -n microcks
        ```
    * Verify the Microcks Ingress resource and get its URL.
        ```bash
        kubectl get ingress -n microcks
        ```
    * Access the Microcks UI via your configured hostname and log in using the user created in Keycloak.

## 5. Deploy Microcks with Asynchronous Options

If you require support for asynchronous protocols (like Kafka), follow these additional steps.

1.  **Enable SSL Passthrough on Ingress Controller (If applicable):**
    * Depending on your Ingress Controller and setup, you might need to enable SSL Passthrough for async protocols like Kafka/gRPC. Consult your Ingress Controller's documentation. A common way involves patching the Ingress Controller deployment using `kubectl patch`.

2.  **Install Strimzi Operator for Kafka Support:**
    * Install the Strimzi Kafka Operator onto your Kubernetes cluster using its provided manifest or Helm chart. This operator manages Kafka clusters within Kubernetes.

3.  **Prepare Microcks Helm Values (for Async):**
    * Create a new Helm values file (e.g., `microcks-async-values.yaml`), starting from your default values file.
    * Add or modify the `features.async` section to enable asynchronous features (`features.async.enabled: true`).
    * Configure the Kafka settings (`features.async.kafka`). You can choose to have the Microcks chart deploy Kafka via Strimzi (`features.async.kafka.install: true`) or connect to an existing Kafka (`features.async.kafka.install: false` and configure connection details). If installing Kafka via the chart, configure the Kafka Ingress settings.

4.  **Deploy Microcks with Asynchronous Options:**
    * Deploy or upgrade your Microcks deployment using the Helm chart and your `microcks-async-values.yaml`.
    * Refer to the Microcks community documentation section on [Deploying Microcks with Asynchronous Options](https://github.com/microcks/community/blob/main/install/gcp/README.md#9-deploy-microcks-on-gke-with-asynchronous-options) for detailed Helm command parameters and configurations (e.g., similar to Step 9 in the GKE example). You will adapt these instructions to your specific cloud environment.

5.  **Verify Microcks Deployment (Async):**
    * Check the status of all Microcks-related pods, including the Async Minion and Kafka components if installed.
        ```bash
        kubectl get pods -n microcks
        ```
    * Verify the Ingress resources for Microcks and Kafka (if applicable) and get their URLs.
        ```bash
        kubectl get ingress -n microcks
        ```
    * Confirm asynchronous mocking/testing capabilities within the Microcks UI.

## Contributing to the Community: Post-Deployment Insights
Sharing your cloud provider-specific deployment scripts, configurations, and lessons learned. Enhance the shared documentation and engage in community discussions to improve the experience for all users.