# Deploying Microcks on AWS EKS

## Overview
This guide provides a step-by-step approach to deploying **Microcks** on **AWS EKS** with **Amazon Aurora (PostgreSQL)** and **Amazon DocumentDB**. It includes setting up **Ingress with ALB, API Gateway, and Load Balancer**.

## Prerequisites
- **AWS CLI** installed and configured ([Install AWS CLI](https://aws.amazon.com/cli/))
- **kubectl** installed ([Install kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/))
- **Helm** installed ([Install Helm](https://helm.sh/docs/intro/install/))
- **An active AWS account** with necessary permissions
- **IAM permissions** for managing EKS, RDS, DocumentDB, and ALB
- **A VPC with at least two public and two private subnets**
- **Domain Name (optional)** if you want to set up a custom DNS

## Step 1: Create an EKS Cluster
### 1.1 Create EKS Cluster
```sh
aws eks create-cluster \
    --name microcks-cluster \
    --role-arn arn:aws:iam::<AWS_ACCOUNT_ID>:role/EKSClusterRole \
    --resources-vpc-config subnetIds=<SUBNET_IDS>,securityGroupIds=<SECURITY_GROUP_ID>
```

### 1.2 Create a Node Group
```sh
aws eks create-nodegroup \
    --cluster-name microcks-cluster \
    --nodegroup-name microcks-nodegroup \
    --subnets <SUBNET_IDS> \
    --instance-types t3.medium \
    --scaling-config minSize=1,maxSize=3,desiredSize=2 \
    --node-role arn:aws:iam::<AWS_ACCOUNT_ID>:role/EKSWorkerRole
```

### 1.3 Update kubeconfig
```sh
aws eks update-kubeconfig --name microcks-cluster
kubectl get nodes  # Verify nodes are ready
```

## Step 2: Setup RDS for PostgreSQL
### 2.1 Create an Aurora PostgreSQL Instance
```sh
aws rds create-db-cluster \
    --db-cluster-identifier microcks-db-cluster \
    --engine aurora-postgresql \
    --master-username admin \
    --master-user-password <YOUR_PASSWORD> \
    --vpc-security-group-ids <SECURITY_GROUP_ID>
```

### 2.2 Retrieve the RDS Endpoint
```sh
aws rds describe-db-clusters --query "DBClusters[0].Endpoint" --output text
```

## Step 3: Setup DocumentDB
### 3.1 Create a DocumentDB Cluster
```sh
aws docdb create-db-cluster \
    --db-cluster-identifier microcks-docdb-cluster \
    --engine docdb \
    --master-username admin \
    --master-user-password <YOUR_PASSWORD> \
    --vpc-security-group-ids <SECURITY_GROUP_ID>
```

### 3.2 Retrieve the DocumentDB Endpoint
```sh
aws docdb describe-db-clusters --query "DBClusters[0].Endpoint" --output text
```

## Step 4: Deploy Microcks using Helm
### 4.1 Add Microcks Helm Repository
```sh
helm repo add microcks https://microcks.io/helm/
helm repo update
```

### 4.2 Deploy Microcks
```sh
helm install microcks microcks/microcks \
    --namespace microcks \
    --create-namespace \
    --set microcks.db.external=true \
    --set microcks.db.mongo.connectionString="mongodb://admin:<YOUR_PASSWORD>@<DOCDB_ENDPOINT>:27017/microcks" \
    --set microcks.db.postgres.host=<RDS_ENDPOINT> \
    --set microcks.db.postgres.user=admin \
    --set microcks.db.postgres.password=<YOUR_PASSWORD>
```

## Step 5: Setup AWS ALB Ingress Controller
### 5.1 Deploy ALB Ingress Controller
```sh
kubectl apply -f https://github.com/kubernetes-sigs/aws-load-balancer-controller/releases/latest/download/v2_3_1_full.yaml
```

### 5.2 Deploy Ingress for Microcks
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: microcks-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS": 443}]'
spec:
  rules:
  - host: microcks.<your-domain>
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: microcks
            port:
              number: 8080
```
```sh
kubectl apply -f microcks-ingress.yaml
```

## Step 6: Setup API Gateway
### 6.1 Create an API Gateway for Microcks
```sh
aws apigatewayv2 create-api \
    --name "MicrocksAPI" \
    --protocol-type HTTP \
    --target "http://<ALB-DNS-NAME>"
```

## Step 7: Verify Deployment
### 7.1 Get Microcks URL
```sh
kubectl get ingress -n microcks
```
Visit the displayed **ALB or API Gateway URL** in a browser.

### 7.2 Test API Access
```sh
curl http://<ALB-DNS-NAME>/microcks
```

## Cleanup (If Needed)
```sh
aws eks delete-cluster --name microcks-cluster
aws rds delete-db-cluster --db-cluster-identifier microcks-db-cluster --skip-final-snapshot
aws docdb delete-db-cluster --db-cluster-identifier microcks-docdb-cluster --skip-final-snapshot
```

Microcks is now deployed on **AWS EKS** with **Aurora RDS, DocumentDB, ALB, and API Gateway**. 🚀 Enjoy your SaaS deployment!

