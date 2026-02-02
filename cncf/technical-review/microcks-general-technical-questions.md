# General Technical Review - Microcks / Incubation

- **Project:** Microcks
- **Project Version:** v1.13.2
- **Website:** https://microcks.io
- **Date Updated:** 2026-01-30
- **Template Version:** v1.0
- **Description:**  
  Microcks is an open source cloud native tool for mocking and testing APIs and event-driven. It supports REST, gRPC, GraphQL, SOAP, and AsyncAPI-based services, enabling teams streamline cloud native application development and maintenance (See, [What is Microcks?](https://microcks.io/documentation/overview/what-is-microcks/)).

## Day 0 - Planning Phase

### Scope

- **Roadmap process:**  
  The Microcks roadmap is driven by community input, maintainer and code-owners discussions, GitHub issues, and adopter feedback. Scope for mid- and long-term features is discussed openly in the project repository and community channels, with prioritization based on user impact, maintainability, and alignment with cloud native API lifecycle needs. Contributions map directly back to roadmap items through issues and pull requests.

- **Target personas:**  
  - API developers  
  - Platform and DevOps engineers  
  - QA and test engineers  
  - Platform engineering teams supporting API governance

  See, [Who can use Microcks?](https://microcks.io/documentation/overview/what-is-microcks/#who-can-use-microcks).

- **Primary use case:**  
  Mocking, simulating, and testing APIs and asynchronous services based on contract definitions (OpenAPI, AsyncAPI, gRPC, GraphQL, SOAP).

  See, [Main Concepts](https://microcks.io/documentation/overview/main-concepts/).

- **Additional use cases:**  
  - Shift-left mocking and testing on developpers desktop and IDE. See, [How Microcks fit to unify inner and outer loops in cloud native](https://www.linkedin.com/pulse/how-microcks-fit-unify-inner-outer-loops-cloud-native-kheddache/)
  - Testing and conformance automation in CI/CD pipelines. See, [Using GitHub Actions](https://microcks.io/documentation/guides/automation/github-actions/), [Using Jenkins Pipeline](https://microcks.io/documentation/guides/automation/jenkins/), [Using GitLab CI](https://microcks.io/documentation/guides/automation/gitlab/), [Using Tekton Pipeline](https://microcks.io/documentation/guides/automation/tekton/)  
  - Collaborative and Consumer-driven contract testing. See, [Mocks and contract testing are two sides of the same coin](https://medium.com/@lbroudoux/mocks-and-contract-testing-are-two-sides-of-the-same-coin-c6c06e91e1c8)
  - API sandbox as a service. See, [Set up On-Premises API Mocking with Traefik Hub and Microcks](https://doc.traefik.io/traefik-hub/api-mocking/on-premises-setup), [GSMA](https://www.linkedin.com/feed/update/urn:li:activity:7420133539402162176/), [Lombard Odier](https://microcks.io/blog/lombard-odier-revolutionizing-api-strategy/)
  - Mocked APIs exposed as MCP tools for LLM and agentic interactions, enabling validation and testing. See: [MCP endpoints support](https://microcks.io/documentation/explanations/mcp-endpoints/)

- **Unsupported use cases:**  
  - Acting as a production API gateway. See, [Microcks security self-assessment](https://tag-security.cncf.io/community/assessments/projects/microcks/self-assessment/#non-goals).

- **Intended adopters:**  
  Individual developers and organizational teams building or operating cloud native applications, microservices, and API- or event-driven systems across all industry verticals, as reflected by the diversity of our [public adopter list](https://github.com/microcks/.github/blob/main/ADOPTERS.md).

- **End user research:**  
  Feedback is primarily gathered through GitHub issues, community discussions, conference talks, and workshops. No formal published research reports are currently available. 
  
  But, we provide adopter use case blog posts that explain the motivations and criteria behind selecting Microcks. See:
  - [BNP Paribas' IT Journey with Microcks](https://microcks.io/blog/bnp-journey-with-microcks/)
  - [Revolutionizing API Strategy: Lombard Odier's Success Story with Microcks](https://microcks.io/blog/lombard-odier-revolutionizing-api-strategy/)
  - [CNAM Partners with Microcks for Automated SOAP Service Mocking](https://microcks.io/blog/cnam-soap-service-mocking/)
  - [J.B. Hunt: Mock It till You Make It with Microcks](https://microcks.io/blog/jb-hunt-mock-it-till-you-make-it/)

Documentation reference:  
<https://microcks.io/documentation/>

---

### Usability

- **User interaction:**  
  Users interact with Microcks through:
  - A web-based UI, CLI, REST APIs
  - CI/CD integrations (e.g., Jenkins, GitHub Actions)
  - Integrate Microcks into your unit tests using [Testcontainers](https://microcks.io/documentation/guides/usage/developing-testcontainers/) libraries.

- **UX/UI:**  
  Microcks provides a web UI for managing API definitions, mocks, tests, and results. The UI is designed for ease of use by developers and testers, with visual feedback for test execution and service simulation.

- **Integration with other projects:**  
  See, TAG App Delivery - [Microcks update deck](https://docs.google.com/presentation/d/1dQpmqwOKFroAHPzhAzSXip1AnM96WUH-9UrVNiBgtP0/edit#slide=id.g32b6d8da232_0_279) for the big picture.

  More details:

  - Helm: <https://microcks.io/documentation/references/configuration/helm-chart-config/>
  - Kubernetes Operator: <https://microcks.io/documentation/guides/installation/kubernetes-operator/>
  - OIDC: <https://microcks.io/blog/mocking-oidc-redirect/>
  - Prometheus: <https://microcks.io/documentation/explanations/monitoring/#technical-metrics>
  - Grafana: <https://microcks.io/documentation/explanations/monitoring/#grafana-dashboard>
  - OpenTelemetry: <https://microcks.io/blog/observability-for-microcks-at-scale/> & <https://microcks.io/documentation/explanations/monitoring/#opentelemetry-support>
  - Keycloak: <https://microcks.io/documentation/guides/administration/users/> & <https://microcks.io/documentation/references/configuration/security-config/#identity-management>
  - gRPC: <https://microcks.io/documentation/tutorials/first-grpc-mock/>
  - CloudEvents: <https://microcks.io/blog/simulating-cloudevents-with-asyncapi/>
  - Cosign: <https://microcks.io/documentation/references/container-images/#signatures>
  - SLSA: <https://microcks.io/documentation/references/container-images/#provenance>
  - Podman: <https://microcks.io/documentation/guides/installation/podman-compose/>
  - Backstage: <https://microcks.io/blog/backstage-integration-launch/>
  - Traefik: <https://doc.traefik.io/traefik-hub/api-mocking/on-premises-setup>
  - GitLab: <https://about.gitlab.com/blog/2023/09/27/microcks-and-gitlab-part-one/>
  - Tekton: <https://microcks.io/documentation/guides/automation/tekton/>
  - Jenkins: <https://microcks.io/documentation/guides/automation/jenkins/>
  - GitHub Actions: <https://microcks.io/documentation/guides/automation/github-actions/>
  - Dagger: <https://daggerverse.dev/mod/github.com/fluent-ci-templates/microcks-pipeline@645fe89a0d2a46afbfb778a938cddc06d26b4c4c>
  - Testcontainers: <https://microcks.io/documentation/guides/usage/developing-testcontainers/> & <https://testcontainers.com/modules/microcks/>
  - Docker Extension: <https://www.docker.com/blog/get-started-with-the-microcks-docker-extension-for-api-mocking-and-testing/>
  - Dapr: <https://www.cncf.io/blog/2025/06/25/cloud-native-app-local-development-made-easy-with-microcks-and-dapr/> & 🎥 <https://youtu.be/Pdeqic_pXGE?si=7mPOZfRWMWSEChoK>

Documentation reference:  
<https://microcks.io/documentation/tutorials/>  
<https://microcks.io/documentation/guides/usage/>

---

### Design

- **Design principles:**  
  Microcks is a modular, cloud-native application that can be deployed using various installation methods. All components are distributed as container images with different flavours to adapt to use cases (a Kubernetes cluster or a developer's laptop). Many deployment options are available: all infrastructure-dependent components are configurable, and adopters can run their own or use managed services.

  Microcks' internal design principles follow a clean, hexagonal architecture, with all features available through REST APIs.

  The schema below represents a full-featured architecture deployment with relations and actions between actors and connection to outer brokers. We represented Kafka ones (X broker) as well as brokers (Y and Z) from other protocols. Microcks users access the main webapp either from their browser to see the console or from the CLI or any other application using the API endpoints.

  ![Microcks architecture](https://microcks.io/images/documentation/architecture-full.png)

- **Architecture requirements:**  
  See, [Architecture & deployment](https://microcks.io/documentation/explanations/deployment-options/) options documentation for comprehensive details.

- **Service dependencies:**  
  - MongoDB (persistent storage)  
  - Keycloak and PostgreSQL (authentication and authorization) is optional
  - External brokers for Async messages (Kafka, MQTT...) are optional

- **Identity and Access Management:**  
  Microcks integrates with Keycloak and supports OAuth2/OpenID Connect for user authentication and role-based access control.

- **Sovereignty:**  
  Data residency and sovereignty are managed by deploying Microcks within the adopter’s own infrastructure and cluster.

- **Compliance requirements:**  
  None are provided directly by the project itself, as we (the maintainers) consider this to be the responsibility of adopters. That said, we are aware that some organizations have integrated Microcks into their internal PCI DSS compliance processes.

  Additionally, some adopters leverage Microcks to support application compliance indirectly, by enabling contract testing, improving traceability, and enforcing controlled access through IAM integrations.

- **High Availability:**  
  When deployed in a centralized organizational setup, high availability is achieved by running multiple replicas of Microcks services on Kubernetes, backed by highly available MongoDB and PostgreSQL instances for Keycloak deployments.

  See, [Deployment topologies](https://microcks.io/documentation/explanations/deployment-topologies/)

- **Resource requirements:**  
  Resource usage depends on scale and usage patterns. Typical deployments require moderate CPU and memory, with increased needs when running large numbers of tests or simulations.

  To help adopters with sizing Microcks, we’ve shared [k6 benchmark results](https://github.com/microcks/microcks/tree/master/benchmark) and launched a [community initiative](https://github.com/microcks/community/blob/main/install/SIZING.md) where adopters can share best practices and feedback. This initiative was also part of the [LFX Mentorship Program](https://www.cncf.io/blog/2025/08/14/beyond-code-open-source-mentorship-and-microcks/).

- **Storage requirements:**  
  - Persistent or ephemeral storage for MongoDB  
  - Ephemeral storage for pods and runtime data

- **API Design:**  
  - [REST-based](https://microcks.io/documentation/references/apis/open-api/) and [Async Events APIs](https://microcks.io/documentation/references/apis/async-api/) with well-defined contracts  
  - Versioned APIs with documented breaking changes

- **Release process:**  
  Microcks follows semantic versioning with documented major, minor, and patch releases, accompanied by release notes (blog post for each release, See: [Microcks 1.13.0 release](https://microcks.io/blog/microcks-1.13.0-release/)).

Documentation reference:  
<https://microcks.io/documentation/explanations/>

---

### Installation

- **Installation approach:**  
  Microcks can be installed via:
  - Helm charts
  - Kubernetes Operator (full GitOps approach)
  - Docker Compose (development/testing)

* **Testing and validation:**  
  Installation is validated by accessing the Microcks UI, deploying and running built-in sample APIs.

Documentation reference:  
<https://microcks.io/documentation/guides/installation/>

---

### Security

- **Security self assessment:**  
  A formal CNCF security self-assessment has been done and published.

  See, [Microcks Self-assessment](https://tag-security.cncf.io/community/assessments/projects/microcks/self-assessment/)

- **Cloud native security tenets:**  
  - Secure by default configurations  
  - Explicit opt-in for advanced or experimental features  
  - Clear documentation for loosening security controls when needed

- **Security hygiene:**  
  - Regular dependency updates  
  - Community review of contributions  
  - CI pipelines for validation and testing
  - Tools we use to secure our supply chain: Sonar Cloud, FOSSA, Cosign / Sigstore, Clair / Docker Scout, Syft

- **Threat modeling:**  
  - Least-privilege access via Kubernetes RBAC  
  - TLS and certificate management handled by the platform  
  - Secure software supply chain practices via [container image](https://microcks.io/documentation/references/container-images/) builds and dependency management

Documentation reference:  
<https://microcks.io/documentation/references/configuration/security-config/>

---

## Day 1 - Installation and Deployment Phase

### Project Installation and Configuration

Microcks is installed primarily on Kubernetes using Helm charts or Operators. Configuration is provided through [Helm values](https://microcks.io/documentation/references/configuration/helm-chart-config/), [Custom Resources](https://microcks.io/documentation/guides/installation/kubernetes-operator/) (for Operators), and environment variables. Optional components such as Kafka integration are enabled explicitly.

Documentation reference:  
<https://microcks.io/documentation/guides/installation/>

---

### Project Enablement and Rollback

- Microcks is enabled by deploying its Kubernetes resources into a namespace.
- No control plane or node downtime is required.
- Disabling is achieved by uninstalling the Helm release or deleting the Operator-managed resource.
- Microcks does not alter cluster defaults or mutate workloads.
- Cleanup is handled automatically by Helm or the Operator.

Documentation reference:  
<https://microcks.io/documentation/guides/installation/>

---

### Rollout, Upgrade and Rollback Planning

- Compatibility with Kubernetes is maintained through continuous testing and dependency updates.
- Upgrades are performed via Helm upgrade or Operator reconciliation.
- Rollbacks are supported using Helm rollback or redeploying a previous Operator configuration.
- Failures impact only Microcks services, not existing workloads.
- Metrics such as pod health, API error rates, and connectivity issues inform rollback decisions.
- Deprecations and breaking changes are communicated via release notes and documentation.
- Alpha and beta features are explicitly gated and opt-in using our [images nighly](https://quay.io/repository/microcks/microcks?tab=history) builds.

Documentation reference:  
<https://github.com/microcks/microcks/blob/master/TESTED_CONFIGURATIONS.md>

WIP as a community effort (adopters contribution):  
<https://github.com/microcks/community/blob/main/install/COMPATIBILITY-MATRIX.md>
