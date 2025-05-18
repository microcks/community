# Deploying Microcks on AWS Elastic Kubernetes Service (EKS)

## Overview
This guide provides a step-by-step approach to deploying **Microcks** on **AWS EKS (Elastic Kubernetes Service)** with **Amazon Aurora (PostgreSQL)** and **Amazon DocumentDB**. It includes setting up **Ingress with ALB**.

## Prerequisites

Ensure the following tools are installed on your local system:

1. **AWS CLI:** [Install Guide](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
2. **kubectl (Kubernetes CLI):** [Install Guide](https://kubernetes.io/docs/tasks/tools/)
3. **Helm:** [Install Guide](https://helm.sh/docs/intro/install/)
4. **eksctl:** [Install Guide](https://eksctl.io/installation/)
5. **An active AWS account** with necessary permissions
6. **IAM permissions** for managing EKS, RDS, DocumentDB, and ALB
7. **A VPC with at least two public and two private subnets**
8. **Domain Name (optional)** if you want to set up a custom DNS
9. **jq (optional, for JSON parsing):** [Install Guide](https://stedolan.github.io/jq/)

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
### 1.1 Create EKS Cluster
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

### 1.2: Associate IAM OIDC Provider
```sh
eksctl utils associate-iam-oidc-provider \
  --region=<REGION> \
  --cluster=<CLUSTER-NAME> \
  --approve
```
### 1.3: Update kubeconfig to Interact with the Cluster
```sh
aws eks update-kubeconfig \
  --name <CLUSTER-NAME> \
  --region <REGION>
```

### 1.4: Create VPC CNI Add-on (optional but recommended)
```sh
eksctl create addon \
  --name vpc-cni \
  --cluster <CLUSTER-NAME> \
  --region <REGION>
```

### 1.5: Verify EKS Cluster Status
```sh
eksctl get cluster --name <CLUSTER-NAME>
```

### 1.6: Confirm Node Connectivity
```sh
kubectl get nodes
```
If the nodes show Ready, your EKS cluster is up and kubectl is properly configured.

## 4. Setup Amazon Aurora for PostgreSQL
### 4.1 Create a Subnet Group (if not using default VPC)
```sh
aws rds create-db-subnet-group \
  --db-subnet-group-name microcks-subnet-group \
  --db-subnet-group-description "Microcks-Keycloak DB Subnet Group" \
  --subnet-ids <subnet-1-id> <subnet-2-id>
```

### 4.2 Create PostgreSQL Aurora Cluster
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
Retrieve the endpints:
```sh
aws rds describe-db-clusters --query "DBClusters[0].Endpoint" --output text
```

### 4.3 Create PostgreSQL RDS Instance
```sh
aws rds create-db-instance \
  --db-instance-identifier microcks-db-instance \
  --db-cluster-identifier microcks-db-cluster \
  --engine aurora-postgresql \
  --db-instance-class db.serverless \
  --region <REGION>
```
Wait for 10-11 minutes for the instance to be provisioned.

### 4.4 Create a Database
Connect to the RDS PostgreSQL instance: 
(You may need to make Aurora Cluster `publically accessible` through console.)
```sh
psql -h <Aurora-endpoint> -U microcks -d postgres
```

Make the Aurora cluster publicly accessible using the CLI (optional, for database creation). Be sure to disable public access after creating the database to ensure security:
```sh
aws rds modify-db-instance \
  --db-instance-identifier microcks-db-instance \
  --publicly-accessible \
  --apply-immediately \
  --region <REGION>
```

Create Database:
```sh
CREATE DATABASE keycloak_db;
```
## 5. Deploy Keycloak on EKS
Follow the [Keycloak deployment guide for EKS](https://github.com/microcks/community/blob/main/install/aws/keycloak-installation.md#5-deploy-keycloak-on-eks) provided by the Microcks community.

Start from **Step 5** of the document and continue through the remaining steps to deploy Keycloak on your EKS cluster.

Once Keycloak is successfully deployed, complete the following configuration:

- Create a `microcks` **realm** in Keycloak.
- Set up a `microcks` **client** within that realm.
- Add a `microcks` **user** and assign appropriate roles for accessing the Microcks dashboard.

## 6. Setup Amazon DocumentDB cluster
### 6.1 Create a Subnet Group for DocumentDB
```sh
aws docdb create-db-subnet-group \
  --db-subnet-group-name microcks-docdb-subnet-group \
  --db-subnet-group-description "Microcks DocumentDB Subnet Group" \
  --subnet-ids <subnet-1-id> <subnet-2-id> \
  --region <REGION>
```

### 6.2 Create a DocumentDB Cluster
```sh
aws docdb create-db-cluster \
  --db-cluster-identifier microcks-docdb-cluster \
  --master-username microcks \
  --master-user-password microcks123 \    # Set a strong password
  --vpc-security-group-ids <VPC-SECURITY-GROUP-ID> \
  --db-subnet-group-name microcks-docdb-subnet-group \
  --region <REGION>
```
Retrieve the endpoint for your Microcks configuration:
```sh
aws docdb describe-db-clusters \
  --db-cluster-identifier microcks-docdb-cluster \
  --query "DBClusters[0].Endpoint" \
  --output text \
  --region <REGION>
```

### 6.3 Create a DocumentDB Instance
```sh 
aws docdb create-db-instance \
  --db-instance-identifier microcks-docdb-instance \
  --db-cluster-identifier microcks-docdb-cluster \
  --db-instance-class db.t3.medium \
  --region <REGION>
```
⏳ Wait for about 10 minutes for the instance to be fully provisioned and ready.

### 6.4 Create the MongoDB Connection Secret
```sh
kubectl create secret generic microcks-mongodb-connection -n microcks \
  --from-literal=username=<USERNAME> \
  --from-literal=password=<PASSWORD>
```

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
      value: "mongodb://<USERNAME>:<PASSWORD>@<Your-DocumentDB-URL>:27017/microcks?authSource=microcks"
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

### 7.3 Deploy Microcks

```sh
helm install microcks microcks/microcks \
    -n microcks \
    -f microcks.yaml
```

### 7.4 Verify Pod Status

```sh
kubectl get pods -n microcks
```

### 7.5 Get Microcks Ingress and Access URL
```sh
kubectl get ingress -n microcks
```
Microcks is available at: `https://microcks.YOUR-DOMAIN.com` 
gRPC mock service is available at: `https://microcks-grpc.YOUR-DOMAIN.com`

🎉 Congratulations! You have successfully deployed Microcks on EKS. Now, you can start using Microcks to mock and test your APIs seamlessly in your cloud environment.

## Cleanup (If Needed)
```sh
aws eks delete-cluster --name microcks-cluster
aws rds delete-db-cluster --db-cluster-identifier microcks-db-cluster --skip-final-snapshot
aws docdb delete-db-cluster --db-cluster-identifier microcks-docdb-cluster --skip-final-snapshot
```

Microcks is now deployed on **AWS EKS** with **Aurora RDS, DocumentDB, and ALB**. 🚀 Enjoy your SaaS deployment!

---

## 📝 Contributions from the Community 🤝

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

If you’d like to contribute, please open a [Pull Request](https://github.com/microcks/community/tree/main/install/aws) or file an [Issue](https://github.com/microcks/community/issues) with your suggestions. You can also contribute directly to the [Microcks documentation](https://microcks.io/documentation/) by improving guides, tutorials, or adding new content.

Together, we can make this deployment guide more reliable, secure, and accessible to everyone.

---