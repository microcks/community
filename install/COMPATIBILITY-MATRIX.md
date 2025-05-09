# Microcks Compatibility Matrix

This document provides compatibility information between Microcks versions and the various dependencies it relies on. The data is based on real-world feedback from Microcks adopters.

## Microcks Versions

| Status      | Version           | Branch      | Container Images tags |
|-------------|-------------------|-------------|----------------------|
| Stable      | 1.11.2            | master      | 1.11.2, latest       |
| Dev         | 1.12.0-SNAPSHOT   | 1.12.x      | nightly              |
| Maintenance | 1.11.3-SNAPSHOT   | 1.11.x      | maintenance          |

## Purpose

This compatibility matrix serves to:
- Guide users in selecting compatible dependency versions for Microcks deployments
- Help users make informed decisions when planning upgrades
- Document tested configurations from the community
- Reduce deployment issues related to version incompatibilities

## How to Contribute

If you're using Microcks in production, please contribute your configuration details by:
1. Forking this repository
2. Adding your tested configuration to the appropriate tables
3. Creating a pull request with your changes

## Compatible Cloud Providers

The table below shows which Cloud Providers have been tested and validated with different Microcks versions:

| Certified Cloud Providers | 1.10.x | 1.11.x | 1.12.x |
|---------------------------|--------|--------|--------|
| Amazon Web Services       |        |        |        |
| Google Cloud Platform     |        |        |        |
| Microsoft Azure           |        |        |        |
| Oracle Cloud              |        |        |        |
| Scaleway                  |        |        |        |
| OVH                       |        |        |        |

### Kubernetes distrib

| Distrib     | Version | Microcks version | Reporter               | Install method | Setup/Info |
|-------------|---------|------------------|------------------------|----------------|------------|
| OpenShift   | 4.12    | 1.11.2           | [Raoul Quincempoix](https://github.com/raoul) | Helm | [Link to setup](https://github.com/microcks/microcks/blob/master/install/kubernetes/README.md) |
| GKE         | Dec 2024| 1.11.0           | [Raoul Quincempoix](https://github.com/raoul) | Operator | [Link to setup](https://microcks.io/documentation/guides/installation/kubernetes-operator/) |
| AKS         |      |            |                      |       |       |
| (add kubernetes distribution) | (version) | (microcks version) | (your name) | (install method) | (setup details) |

### Mongo distrib

| Distrib           | Version     | Microcks version | Reporter               | Install method | Setup/Info |
|-------------------|-------------|------------------|------------------------|----------------|------------|
| MongoDB Atlas     | 2025-01-12  | 1.11.2           | [Raoul Quincempoix](https://github.com/raoul) | Cloud managed | M10 cluster |
| MongoDB Community | 5.5.12      | 1.11.2           | [Raoul Quincempoix](https://github.com/raoul) | Helm chart Bitnami | Self-hosted |
| (add mongo distribution) | (version) | (microcks version) | (your name) | (install method) | (setup details) |

### Keycloak distrib

| Distrib     | Version | Microcks version | Reporter               | Install method | Setup/Info |
|-------------|---------|------------------|------------------------|----------------|------------|
| RH SSO      | 8.11    | 1.11.2           | [Raoul Quincempoix](https://github.com/raoul) | RH SSO Operator | OpenShift deployment |
| Keycloak    | 26.2.1  | 1.12.0           | [Raoul Quincempoix](https://github.com/raoul) | Helm | [Deploy guide](https://blog.devops.dev/deploy-keycloak-v24-to-k8s-cluster-with-helm-83e6714f2888) |
| (add keycloak distribution) | (version) | (microcks version) | (your name) | (install method) | (setup details) |

## Integrations

### Backstage distrib

| Distrib     | Version | Microcks version | Reporter | Setup/Info |
|-------------|---------|------------------|----------|------------|
| Backstage   |         |                  |          |            |
| (add backstage version) | (version) | (microcks version) | (your name) | (setup details) |

### Jenkins distrib

| Distrib     | Version | Microcks version | Reporter | Setup/Info |
|-------------|---------|------------------|----------|------------|
| Jenkins     |         |                  |          |            |
| (add jenkins version) | (version) | (microcks version) | (your name) | (setup details) |

## Legend

- **Tested**: Successfully tested by community members
- **Not Available**: Not supported or not tested
- **Stable**: Current stable production release
- **Dev**: Development version (not recommended for production)
- **Maintenance**: Supported maintenance version with critical fixes only

## Notes and Known Issues

This section documents known compatibility issues or special considerations:

- **Example Issue 1**: Description of compatibility issue or limitation
- **Example Issue 2**: Description of compatibility issue or limitation

## Last Updated

This compatibility matrix was last updated on: 05/05/2025

Please submit a pull request to update this information if your tested configuration differs from what is documented here.
