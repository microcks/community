# Microcks Deployment Guides  

> **🚀 Community-Driven Deployment Effort**  
> This section is maintained by **community contributors**, not the core Microcks maintainers.  
> These guides aim to provide best practices for deploying Microcks on various cloud providers.  

## Overview  

Deploy **Microcks**, the **cloud-native API and microservices mocking and testing tool**, seamlessly on any cloud platform.  
Microcks integrates effortlessly with your existing cloud-native ecosystem, enabling you to simulate, test, and validate APIs in production-like environments.  

Microcks is compatible with **managed Kubernetes services** such as:  

- **Amazon Elastic Kubernetes Service (EKS)**  
- **Google Kubernetes Engine (GKE)**  
- **Microsoft Azure Kubernetes Service (AKS)**  
- **Other platforms**: OVH, Oracle Cloud, Scaleway, etc.  

This guide is part of a **collaborative community effort** to document deployment strategies for different cloud providers.  
If your preferred platform is not listed or lacks detailed instructions, feel free to **contribute** or **open an issue** in the [Microcks Community Repository](https://github.com/microcks/community).  

---

## Microcks Platform Compatibility Overview  

Microcks is designed to run on **Kubernetes**, making it compatible with a wide range of cloud providers and on-premises environments.  
Below is a list of supported platforms, along with links to their official documentation and detailed deployment guides for Microcks.  

| Vendor                     | Platform Name                          | Documentation                                                                                     | Microcks Deployment                                                                                   |
|----------------------------|----------------------------------------|--------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------|
| **Amazon Web Services**    | Amazon Elastic Kubernetes Service (EKS) | [EKS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html)       | [Deploy Microcks on EKS](https://github.com/microcks/community/tree/main/install/aws)                 |
| **Google Cloud Platform**  | Google Kubernetes Engine (GKE)          | [GKE Documentation](https://cloud.google.com/kubernetes-engine/docs/quickstart)                  | [Deploy Microcks on GKE](https://github.com/microcks/community/tree/main/install/gcp)                 |
| **Microsoft Azure**        | Azure Kubernetes Service (AKS)          | [AKS Documentation](https://learn.microsoft.com/en-us/azure/aks/)                                | [Deploy Microcks on AKS](https://github.com/microcks/community/tree/main/install/azure)               |
| **Oracle Cloud**           | Oracle Container Engine for Kubernetes (OKE) | [OKE Documentation](https://docs.oracle.com/en-us/iaas/Content/ContEng/Concepts/contengoverview.htm) | [Deploy Microcks on OKE](https://github.com/microcks/community/tree/main/install/oracle)              |
| **OVH**                    | OVH Managed Kubernetes                  | [OVH Kubernetes Documentation](https://docs.ovh.com/gb/en/kubernetes/)                           | [Deploy Microcks on OVH](https://github.com/microcks/community/tree/main/install/ovh)                 |
| **Scaleway**               | Scaleway Kubernetes Kapsule              | [Scaleway Kapsule Documentation](https://www.scaleway.com/en/docs/containers/kapsule/)           | [Deploy Microcks on Scaleway](https://github.com/microcks/community/tree/main/install/scaleway)       |
