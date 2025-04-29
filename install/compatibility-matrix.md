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

For each compatibility entry, please:
- Add the specific versions you've tested (e.g., "MongoDB 5.0.6, 6.0.1")
- If a configuration doesn't work, please note it as "Not Available"
- Leave blank if you haven't tested this combination

## Compatible Cloud Providers

The table below shows which Cloud Providers have been tested and validated with different Microcks versions:

| Certified Cloud Providers | 1.11.0 |       | 1.11.1 |       | 1.11.2 |       |
|---------------------------|--------|--------|--------|--------|--------|--------|
|                           | UPI    | IPI    | UPI    | IPI    | UPI    | IPI    |
| Amazon Web Services       |        |        |        |        |        |        |
| Google Cloud Platform     |        |        |        |        |        |        |
| Microsoft Azure           |        |        |        |        |        |        |
| Oracle Cloud              |        |        |        |        |        |        |
| Scaleway                  |        |        |        |        |        |        |
| OVH                       |        |        |        |        |        |        |

## Dependency Version Compatibility

Please list the dependency versions you've tested with each Microcks version below. You can add compatible versions as comma-separated values in each cell.

| Dependency | Microcks 1.11.0 | Microcks 1.11.1 | Microcks 1.11.2 |
|------------|-----------------|-----------------|-----------------|
| MongoDB    |                 |                 |                 |
| PostgreSQL |                 |                 |                 |
| Kubernetes |                 |                 |                 |
| Java       |                 |                 |                 |
| Keycloak   |                 |                 |                 |
| (add other dependency) |    |                 |                 |

## Additional Integrations

Please list any additional integrations you've tested with each Microcks version below. You can add compatible versions as comma-separated values in each cell.

| Integration | Microcks 1.11.0 | Microcks 1.11.1 | Microcks 1.11.2 |
|-------------|-----------------|-----------------|-----------------|
| (add integration name) |     |                 |                 |
|             |                 |                 |                 |
|             |                 |                 |                 |

## Legend

- **UPI**: User Provisioned Infrastructure
- **IPI**: Installer Provisioned Infrastructure
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

This compatibility matrix was last updated on: 29/04/2025

Please submit a pull request to update this information if your tested configuration differs from what is documented here.
