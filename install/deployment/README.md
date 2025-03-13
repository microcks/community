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

| Vendor                     | Platform Name                          | Compatible | Documentation                                                                                                                                      | Microcks Deployment Guide                                                                               |
|----------------------------|----------------------------------------|------------|----------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------|
| **Amazon Web Services**    | Amazon Elastic Kubernetes Service (EKS) | Yes     | [EKS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html)                                                         | [Deploy Microcks on EKS](https://github.com/alikhere/project-microcks/tree/main/deployment/cloud-providers/aws)       |
| **Google Cloud Platform**  | Google Kubernetes Engine (GKE)          | Yes     | [GKE Documentation](https://cloud.google.com/kubernetes-engine/docs/quickstart)                                                                    | [Deploy Microcks on GKE](https://github.com/alikhere/project-microcks/tree/main/deployment/cloud-providers/gcp)       |
| **Microsoft Azure**        | Azure Kubernetes Service (AKS)          | Yes     | [AKS Documentation](https://learn.microsoft.com/en-us/azure/aks/)                                                                                  | [Deploy Microcks on AKS](https://github.com/alikhere/project-microcks/tree/main/deployment/cloud-providers/azure)     |
| **Oracle Cloud**           | Oracle Container Engine for Kubernetes (OKE) | Yes | [OKE Documentation](https://docs.oracle.com/en-us/iaas/Content/ContEng/Concepts/contengoverview.htm)                                               | [Deploy Microcks on OKE](https://github.com/alikhere/project-microcks/tree/main/deployment/cloud-providers/oracle)    |
| **OVH**                    | OVH Managed Kubernetes                  | Yes     | [OVH Kubernetes Documentation](https://docs.ovh.com/gb/en/kubernetes/)                                                                             | [Deploy Microcks on OVH](https://github.com/alikhere/project-microcks/tree/main/deployment/cloud-providers/ovh)       |
| **Scaleway**               | Scaleway Kubernetes Kapsule              | Yes     | [Scaleway Kapsule Documentation](https://www.scaleway.com/en/docs/containers/kapsule/)                                                             | [Deploy Microcks on Scaleway](https://github.com/alikhere/project-microcks/tree/main/deployment/cloud-providers/scaleway) |
