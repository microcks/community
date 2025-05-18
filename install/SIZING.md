# Microcks Sizing Guide and Benchmarks

This document serves as both a guide for sizing your Microcks deployment and a collection of community benchmarks. By sharing your deployment configurations and performance results, you help other users make informed decisions about their own Microcks implementations.

## Running the Benchmark

Microcks includes a benchmark tool based on [k6](https://k6.io/) that simulates various user patterns:
- UI browsing users
- REST mock requests
- GraphQL mock requests
- SOAP mock requests

To run the benchmark against your Microcks deployment:

```bash
# Clone the repository (if you don't have it already)
git clone https://github.com/microcks/microcks.git

# Navigate to the benchmark directory
cd microcks/benchmark

# Run the benchmark (replace with your Microcks URL)
docker run --rm -i -e BASE_URL=https://your-microcks-url grafana/k6:latest run - < bench-microcks.js

# Optionally configure wait time between requests
docker run --rm -i -e BASE_URL=https://your-microcks-url -e WAIT_TIME=0.1 grafana/k6:latest run - < bench-microcks.js
```

## Benchmark Configuration

The k6 script simulates the following default distribution:
- 20 concurrent UI users
- 40 concurrent REST API clients (200 iterations per client)
- 20 concurrent GraphQL clients (100 iterations per client)
- 5 concurrent SOAP clients (5 iterations per client)

You can modify these parameters in the benchmark script to better match your expected usage patterns.

## Community Benchmarks

This section contains real-world deployment configurations and benchmark results from the Microcks community. Feel free to add your own experience by submitting a PR to this document.

### Benchmark Template

When adding your benchmark, please use this template:

```
### [Organization Name/Environment Description]

**Infrastructure:**
- Kubernetes version: 
- Node type/size:
- Storage class:
- Network details (if relevant):

**Microcks Configuration:**
- Version:
- Deployment method (Helm, Operator, etc.):
- Number of replicas:
- CPU allocation:
- Memory allocation:
- JVM settings:
- Special configuration (if any):

**MongoDB Configuration:**
- Version:
- Deployment type (standalone, replica set, etc.):
- CPU allocation:
- Memory allocation:
- Storage size:
- Indexes in use:

**Workload Characteristics:**
- Number of API definitions:
- Types of APIs (REST, SOAP, GraphQL, etc.):
- Average complexity of mocks:
- Usage pattern (constant, spiky, etc.):

**Benchmark Results:**
- Requests/second:
- Response time (avg, p95, p99):
- Resource utilization during test:
- Any bottlenecks identified:

**Additional Notes:**
- Any special observations or recommendations:
```

---


## Contribute Your Benchmark

To contribute your benchmark results:
1. Fork the Microcks repository
2. Add your benchmark to this file using the template provided
3. Submit a Pull Request

Your real-world experience will help the community better understand Microcks performance characteristics in various environments!
