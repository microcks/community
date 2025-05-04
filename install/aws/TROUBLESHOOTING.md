# 🚀 Troubleshooting 

## Overview
This document provides a comprehensive troubleshooting guide for deploying **Microcks** on Amazon Web Services (AWS) with **External Keycloak** and **MongoDB**. It includes common errors encountered during the deployment process and their solutions. Use this guide to resolve issues and ensure smooth deployment.

If you face issues not covered in this document, please feel free to contribute by adding your own experiences and solutions to this guide. Your contributions help the community!

## 1. **CloudFormation Stack Creation Error**

### Issue:  
AWS CloudFormation stack creation fails with the error:  
`Stack creation failed: The stack name you specified is already in use. Please try an alternative name.`

### Solution:
This error occurs when the **stack name** is not unique. Ensure that the stack name is globally unique within your AWS account and region.

#### Create a New Stack:
```sh
aws cloudformation create-stack --stack-name <unique-stack-name> --template-body file://template.json --region <aws-region>
```

## 2.  Keycloak Pod Not Starting / CrashLoopBackOff

### Diagnosis:
```sh
kubectl get pods -n microcks -l app.kubernetes.io/name=keycloak
kubectl describe pod <pod-name> -n microcks
kubectl logs -n microcks <pod-name> --previous
```

### If Error is "Unable to Connect to Host":
```sh
# Enable public IP connectivity or set up private IP.
EKS_NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="ExternalIP")].address}')
aws rds modify-db-instance --db-instance-identifier <db-instance-id> --vpc-security-group-ids <security-group-id> --region <aws-region>
```

### If Error is "Password Authentication Failed for User":
Ensure the values for DB_USER, DB_PASSWORD, and DB_NAME in your keycloak.yaml file exactly match the credentials you used when creating the RDS database and user.

## 3. Microcks Installation Failure

### Issue:
Helm installation fails due to invalid Ingress URL format:
```sh
$ helm install microcks microcks/microcks -f microcks.yaml --namespace microcks
ERROR: Invalid Ingress URL format.
```

### Solution:
Remove the https:// from the URL in the microcks.yaml file:
```sh
microcks:
  url: microcks.<your-domain>.nip.io
```
```sh
# Upgrade Helm release:
helm upgrade microcks microcks/microcks -n microcks -f microcks.yaml
```

## 4. Microcks Pod Not Starting / CrashLoopBackOff
Get pod logs and describe the pod:
```sh
kubectl logs <pod-name> -n microcks
kubectl describe pod <pod-name> -n microcks
```
### Error: "Quota Limit Exceeded":
```sh
# Enable autoscaling
$ aws eks update-cluster-config --name <cluster-name> --region <aws-region> --autoscaling-min-nodes 1 --autoscaling-max-nodes 5

# Or upgrade machine type
$ aws eks update-nodegroup-config --cluster-name <cluster-name> --nodegroup-name <nodegroup-name> --instance-types m5.large --region <aws-region>
```
If the issue persists, check AWS Console → IAM & Admin → Quotas, and review resource limits like CPUs, IPs, and disk size. If necessary, request a quota increase.

### MongoDB Secret Not Found or Connection Refused:
Ensure that the MongoDB Kubernetes secret is created with the correct credentials:
```sh
kubectl create secret generic microcks-mongodb-connection -n microcks \
  --from-literal=username=<USERNAME> \
  --from-literal=password=<PASSWORD>
```
Also, confirm that the SPRING_DATA_MONGODB_URI environment variable is correctly set in the microcks.yaml file:
```sh
SPRING_DATA_MONGODB_URI: "mongodb://<USERNAME>:<PASSWORD>@mongodb.microcks.svc.cluster.local:27017/microcks?authSource=microcks"
```

## 5. NGINX Not Found or Not Secure Warning
### Issue:
NGINX is not found or there is a "Not Secure" warning on Microcks and Keycloak URLs.

### Solution:
- Ensure that cert-manager is installed for SSL certificates.
- Follow the necessary steps in [Keycloak Deployment Documentation](https://github.com/Vaishnav88sk/community/blob/main/install/aws/keycloak-installation.md).
- Add the correct annotations to the keycloak.yaml and microcks.yaml configuration files for proper SSL/TLS configuration.

---

## 📝 Contributions from the Community 🤝
If you’ve encountered an issue that isn’t listed in this guide, please contribute by adding your troubleshooting solutions! Here’s how you can help:
1. Identify the Problem 🔍: Clearly describe the problem you encountered, including error messages, AWS region, and any related logs.
2. Provide a Solution 💡: If you've found a resolution, please provide detailed steps for others to follow.
3. Submit Your Contribution ✍️: For contributions, please open an Issue or a Pull Request with your findings, and we’ll incorporate them into the guide.

If you're an adopter who's already installed Microcks, sharing your experience and troubleshooting steps can significantly help others in the community. 🌱

Your input will help other developers and users who are facing similar issues! 🌟