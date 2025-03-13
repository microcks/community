# Deploying Microcks on AWS EKS

This document outlines the steps for deploying **Microcks** on **Amazon Web Services (AWS) Elastic Kubernetes Service (EKS)**. Microcks is a cloud-native tool for mocking and testing APIs and microservices, enabling efficient simulation of your distributed environment.

## Prerequisites

Before you start deploying Microcks on AWS EKS, ensure that the following prerequisites are in place:

### 1. **AWS Account**
   - An active AWS account with access to EKS, IAM, and other required services.
   - AWS CLI installed and configured with necessary credentials.

### 2. **Command Line Tools**
   - **AWS CLI**: Install and configure the AWS CLI to manage your AWS resources.
     - [AWS CLI Installation](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
   - **kubectl**: Kubernetes command-line tool to interact with your EKS cluster.
     - [kubectl Installation](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
   - **Helm**: Package manager for Kubernetes that simplifies deployments.
     - [Helm Installation](https://helm.sh/docs/intro/install/)
   - **AWS IAM Authenticator**: IAM Authenticator for Kubernetes to authenticate to EKS.
     - [IAM Authenticator Installation](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html)

### 3. **EKS Cluster**
   - An EKS cluster up and running in your AWS account.
   - EKS worker nodes configured with the correct IAM role for accessing AWS services.
     - [EKS Cluster Setup Guide](https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html)


### 5. **IAM Roles and Permissions**
   - Ensure proper IAM roles and policies are set up for EKS and Microcks services.
   - Ensure that your EKS nodes and Microcks service have appropriate permissions to interact with AWS services (e.g., RDS, S3, CloudWatch).

### 7. **Docker**
   - Docker installed to build and run containers.
     - [Docker Installation](https://docs.docker.com/get-docker/)

## Steps to Deploy Microcks on AWS EKS

### Step 1: **Set Up EKS Cluster**


