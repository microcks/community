# 🚀  Troubleshooting 

## Overview
This document provides a detailed troubleshooting guide for deploying Microcks on Azure Kubernetes Service (AKS) with integrated Keycloak (via Azure PostgreSQL) and MongoDB. It outlines issues encountered during setup and deployment, and provides actionable solutions to help ensure a successful deployment.

## 1. Cluster Creation Fails with Forbidden Error

### Issue:
Operation returned an invalid status 'Forbidden'. 
This occurs when the service principal lacks necessary permissions to create AKS resources.

### Solution:
Login as admin and assign required role:

```sh
$ az login
$ az role assignment create \
  --assignee <appId> \
  --role "Contributor" \
  --scope /subscriptions/<subscription-id>/resourceGroups/microcks-prod-rg
```

Login back using service principal and re-run the cluster creation command:

```sh
$ az login --service-principal \
  --username <appId> \
  --password <password> \
  --tenant <tenant-id>
```

## 2. QuotaExceeded for VM Size in Region

### Issue:
"message": "The requested size for resource 'Standard_D4s_v3' exceeds the quota for the region 'centralus'. Available: 0, Required: 4."

### Solution:
Check available quotas for all regions:

```sh
$ for region in $(az account list-locations --query "[].name" -o tsv); do
    echo "Region: $region"
    az vm list-usage --location $region --query "[?name.value=='Total Regional vCPUs']" -o table
  done
```

Deploy AKS in a region with available quota:

```sh
$ az aks create \
  --resource-group microcks-prod-rg \
  --name microcks-prod-aks \
  --location <available-region> \
  --node-count 2 \
  --node-vm-size Standard_D4s_v3 \
  --node-osdisk-size 50 \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 4 \
  --generate-ssh-keys
```

## 3. PostgreSQL Flexible Server Not Supported in Region

### Issue:
"message": "The requested resource type 'Microsoft.DBforPostgreSQL/flexibleServers' is not available in location 'eastus'."
OR
"message": "Currently, there is no capacity to provision flexible servers in the selected region 'eastus'."

### Solution:
List regions supporting PostgreSQL flexible servers:

```sh
$ az postgres flexible-server list-skus --location <region> -o table
```

Choose supported region like centralus, westus, or eastus2.


## 4. CrashLoopBackOff for Keycloak Pod

### Steps to Diagnose:

```sh
$ kubectl get pods -n microcks -l app.kubernetes.io/name=keycloak
$ kubectl describe pod <pod-name> -n microcks
$ kubectl logs -n microcks <pod-name> --previous
```

### Causes & Fixes:

#### DB Connection Issues:
Ensure that PostgreSQL credentials in keycloak.yaml match what was set during DB provisioning.

#### Network Restrictions:
Check if firewall rules allow Azure services and your IP to access PostgreSQL server:

```sh
$ az postgres flexible-server firewall-rule create \
  --resource-group microcks-prod-rg \
  --name microcks-postgres \
  --rule-name allow-azure-services \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 0.0.0.0
```

## 5. CrashLoopBackOff for Microcks Pod

### Step 1: Get Logs

```sh
$ kubectl logs <pod-name> -n microcks
$ kubectl describe pod <pod-name> -n microcks
```

### Possible Causes & Fixes:

#### Quota Limits:
Upgrade node type or enable autoscaling:

```sh
$ az aks nodepool update \
  --resource-group microcks-prod-rg \
  --cluster-name microcks-prod-aks \
  --name nodepool1 \
  --node-count 3
```

### MongoDB Secret Missing or Misconfigured:

#### Ensure secret was created:

```sh
$ kubectl create secret generic microcks-mongodb-connection -n microcks \
  --from-literal=username=<USERNAME> \
  --from-literal=password=<PASSWORD>
```
