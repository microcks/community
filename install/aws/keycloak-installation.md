# Deploying Keycloak on Amazon Elastic Kubernetes Service (EKS)

## Overview
This guide walks you through deploying **Keycloak** on **Amazon EKS (Elastic Kubernetes Service)** using an **Amazon Aurora (PostgreSQL)** instance as the database. Keycloak is an open-source identity and access management solution that enables secure authentication and authorization for applications.

---

## Prerequisites

Ensure the following tools are installed on your local system:

1. **AWS CLI:** [Install Guide](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
2. **kubectl (Kubernetes CLI):** [Install Guide](https://kubernetes.io/docs/tasks/tools/)
3. **Helm:** [Install Guide](https://helm.sh/docs/intro/install/)
4. **eksctl:** [Install Guide](https://eksctl.io/installation/)
5. **jq (optional, for JSON parsing):** [Install Guide](https://stedolan.github.io/jq/)

---

## 1. Configure AWS Credentials

```sh
aws configure
```
Enter your Access Key, Secret Key, region (e.g., us-east-1),

## 2. Create IAM Role for EKS and ECR Access
### IAM Policy JSON

Create a policy named `KeycloakEKSFullAccessPolicy.json` with the following permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "eks:*",
        "iam:CreateRole",
        "iam:AttachRolePolicy",
        "iam:PutRolePolicy",
        "iam:PassRole",
        "iam:GetOpenIDConnectProvider",
        "iam:CreateOpenIDConnectProvider",
        "iam:GetRole",
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage",
        "rds:CreateDBCluster",
        "rds:CreateDBInstance",
        "rds:CreateDBSubnetGroup",
        "rds:DescribeDBClusters",
        "rds:DescribeDBInstances",
        "rds:ModifyDBCluster",
        "rds:DeleteDBCluster",
        "ec2:DescribeSubnets",
        "ec2:DescribeSecurityGroups",
        "ec2:DescribeVpcs"
      ],
      "Resource": "*"
    }
  ]
}
```
Apply the policy:
```sh
eksctl utils associate-iam-oidc-provider --region=<REGION> --cluster=<CLUSTER-NAME> --approve

### Create IAM policy for CloudWatch, ECR, etc.
aws iam create-policy \
  --policy-name AmazonEKSClusterPolicy \
  --policy-document file://KeycloakEKSFullAccessPolicy.json
```

## 3. Create an EKS Cluster with eksctl

### Step 1: Create EKS Cluster
```sh
eksctl create cluster \
  --name <CLUSTER-NAME> \
  --region <REGION> \
  --version <LATEST-VERSION> \
  --vpc-public-subnets <PUBLIC-SUBNETS> \  # Comma separated
  --nodegroup-name microcks-nodes \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3 \
  --managed
```
Wait for 10-12 minutes for the cluster to be provisioned and configure node group and disk size based on your usage or team size.

### Step 2: Associate IAM OIDC Provider
```sh
eksctl utils associate-iam-oidc-provider \
  --region=<REGION> \
  --cluster=<CLUSTER-NAME> \
  --approve
```
### Step 3: Update kubeconfig to Interact with the Cluster
```sh
aws eks update-kubeconfig \
  --name <CLUSTER-NAME> \
  --region <REGION>
```

### Step 4: Update VPC CNI Add-on (optional but recommended)
```sh
eksctl update addon \
  --name vpc-cni \
  --cluster <CLUSTER-NAME> \
  --region <REGION>
```

### Step 5: Verify EKS Cluster Status
```sh
eksctl get cluster --name <CLUSTER-NAME>
```

### Step 6: Confirm Node Connectivity
```sh
kubectl get nodes
```
If the nodes show Ready, your EKS cluster is up and kubectl is properly configured.

## 4. Create Amazon Aurora (PostgreSQL) Instance
### Step 1: Create a Subnet Group (if not using default VPC)
```sh
aws rds create-db-subnet-group \
  --db-subnet-group-name microcks-subnet-group \
  --db-subnet-group-description "Microcks-Keycloak DB Subnet Group" \
  --subnet-ids <subnet-1-id> <subnet-2-id>
```

### Step 2: Create PostgreSQL Aurora Cluster
```sh
aws rds create-db-cluster \
  --db-cluster-identifier microcks-db-cluster \
  --engine aurora-postgresql \
  --engine-version 15.10 \
  --master-username microcks \
  --master-user-password microcks123 \
  --vpc-security-group-ids <VPC-SECURITY-GROUP-ID> \
  --db-subnet-group-name microcks-subnet-group \
  --serverless-v2-scaling-configuration MinCapacity=0.5,MaxCapacity=2 \
  --backup-retention-period 7 \
  --enable-http-endpoint \
  --region <REGION>
```
### Step 3: Create PostgreSQL RDS Instance
```sh
aws rds create-db-instance \
  --db-instance-identifier microcks-db-instance \
  --db-cluster-identifier microcks-db-cluster \
  --engine aurora-postgresql \
  --db-instance-class db.serverless \
  --region <REGION>
```
Wait for 10-11 minutes for the instance to be provisioned.

### Step 4: Create a Database
Connect to the RDS PostgreSQL instance: 
(You may need to make Aurora Cluster `publically accessible` through console.)
```sh
psql -h <Aurora-endpoint> -U microcks -d postgres
```
Create Database:
```sh
CREATE DATABASE keycloak_db;
```
## 5. Deploy Keycloak on EKS
### Add Keycloak Helm Repository
```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```
### Prepare keycloak.yaml Configuration File
```sh
cat > keycloak-values.yaml <<EOF
auth:
  adminUser: admin
  adminPassword: "microcks123"

postgresql:
  enabled: false

externalDatabase:
  host: "<Aurora-endpoint>"
  port: 5432
  database: "keycloak_db"
  user: "microcks"
  password: "microcks123"
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
EOF
```
### Install Keycloak and check Pod Status
```sh
helm install keycloak bitnami/keycloak -f keycloak-values.yaml
kubectl get pods -l app.kubernetes.io/name=keycloak
```

### Install NGINX Ingress Controller
```sh
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.service.type=LoadBalancer \
  --set controller.config."proxy-buffer-size"="128k"
```
### Get External IP of Ingress Controller Once available, export it
```sh
kubectl get svc -n ingress-nginx ingress-nginx-controller -w
export INGRESS_IP=$(kubectl -n ingress-nginx get svc ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
```

### Create and Apply Ingress Resource to Expose Keycloak
```sh
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: keycloak-ingress
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"
    nginx.ingress.kubernetes.io/proxy-buffer-size: "128k"
spec:
  ingressClassName: nginx
  rules:
  - host: keycloak.$INGRESS_IP.nip.io
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: keycloak
            port:
              number: 80
EOF
```

### Verify Ingress and Access Keycloak
```sh
kubectl get ingress
```
Keycloak is available at `http://keycloak.<EXTERNAL-IP>.nip.io`

## 6. Configure Keycloak for your Application
### Step 1: Create a Microcks Realm
- Login to the Keycloak dashboard at `http://keycloak.<YOUR-DOMAIN>.com`
- Click on `Create Realm` in the top left corner and name it `microcks`.

### Step 2: Add a Client for Microcks
- From the left menu, go to `Clients`.
- Click `Create` and enter the C`lient ID` (e.g., `microcks-app-js`).
- Enable `Client authentication` and Click `Next`.
- In `Valid Redirect URIs`, enter your application URL (e.g., `http://microcks.<YOUR-DOMAIN>.com/*`).
- In `Web Origins`, enter `http://microcks.<YOUR-DOMAIN>.com`.
- If you have not deployed your application yet, you can configure this later.

### Step 3: Add a User
- From the left menu, go to `Users`.
- Click `Add User` and enter a `Username`, `Email` and Click `Save`.
- Go to the `Credentials` tab, set a `password`, and save it.

🎉 Congratulations! You have successfully deployed Keycloak on AWS Elastic Kubernetes Service (EKS) with AWS Aurora (PostegreSQL) Database.

---

## 🤝 Community Contributions

We welcome community contributions to enhance this guide and keep it up to date with best practices.

Whether you're an experienced DevOps engineer, a security enthusiast, or someone passionate about documentation, your input can make this deployment guide even more valuable for the community.

### Areas Where You Can Contribute:

- 🔐 **Security Improvements** – Replace hardcoded credentials with secure secret management (e.g., AWS Secrets Manager).
- 🏗️ **Infrastructure as Code** – Convert manual steps into Terraform, AWS CDK, or CloudFormation modules.
- 📈 **Observability Enhancements** – Add integration guidance for monitoring tools like Prometheus, Grafana, or AWS CloudWatch.
- ⚙️ **Helm Configuration Tuning** – Suggest production-optimized values or best practices for Keycloak.
- 🌐 **High Availability** – Propose improvements for multi-AZ setups or failover strategies.
- 🌍 **Localization** – Translate this guide into other languages for broader accessibility.
- 🤖 **Automation Scripts** – Provide shell or Python scripts to automate key deployment steps.

If you’d like to contribute, please open a [Pull Request](https://github.com/microcks/community/tree/main/install/aws) or file an [Issue](https://github.com/microcks/community/issues) with your suggestions.

Together, we can make this deployment guide more reliable, secure, and accessible to everyone.

---
