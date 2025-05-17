## Docker-compose specific

### 🔧 Troubleshooting

  
#### 1. Postman Runner tests get stuck with infinite spinner

**Problem:** When running tests with the POSTMAN runner, the blue spinner runs forever and the test never completes. Browser console shows repeated `[AuthenticationHttpInterceptor] intercept for GET` messages.

**Solution:** This is typically a network connectivity issue where the Postman runtime container cannot reach the host machine.

-  **For Mac users**: Use `http://host.docker.internal:3002` instead of `http://docker.for.mac.localhost:3002` as the upstream service endpoint. While `docker.for.mac.localhost` works for the OpenAPI Schema runner, it doesn't resolve correctly from within the Postman runtime container.
-  **For Linux users**: Try using the host machine's IP address directly, or consult Docker's documentation for host networking options specific to your distribution.
-  **For Windows users**: Use `host.docker.internal` (similar to Mac).

  

**Debug steps:**

1. Enable debug logging for the Postman runtime container:

```sh
docker  compose  down
export  LOG_LEVEL=debug
docker  compose  up  -d
```

  

#### 2. Cannot access Microcks at localhost:8080

**Problem**: Browser cannot connect to Microcks interface.

**Solutions**:

- Ensure all containers are running: docker compose ps
- Check if port 8080 is already in use: `lsof -i :8080` (Mac/Linux) or `netstat -ano | findstr :8080` (Windows)
- Verify firewall settings aren't blocking the port
- Try accessing via `http://127.0.0.1:8080` instead of `localhost`

  

#### 3. Authentication issues with Keycloak

**Problem**: Cannot login with default credentials.

**Solutions**:

- Ensure you're using the exact credentials:


		Username: admin (all lowercase)

		Password: microcks123


- Clear browser cache and cookies
- Try incognito/private browsing mode
- Check Keycloak container logs: docker logs microcks-sso


#### 4. Container startup failures

**Problem**: One or more containers fail to start.

**Solutions**:

Check disk space: `df -h`

Verify Docker daemon is running: `docker ps`

Remove old containers and volumes:

```sh
docker  compose  down  -v
docker  compose  up  -d
```

Check individual container logs for specific errors

### ❓ Frequently Asked Questions

<details>
<summary><strong>Q: What's the difference between docker.for.mac.localhost and host.docker.internal?</strong></summary>

A: Both are special DNS names that resolve to the host machine from within Docker containers. However:

`docker.for.mac.localhost` is Mac-specific and deprecated
`host.docker.internal` is the recommended cross-platform solution (Mac, Windows, and newer Linux versions)

Some Microcks components (like Postman runtime) may not properly resolve the older Mac-specific hostname
</details>

<details>
<summary><strong>Q: Can I change the default ports?</strong></summary>

A: Yes, modify the port mappings in docker-compose.yml:

```yaml

services:

  microcks:

		ports:

		- "9090:8080" # Changes external port to 9090

```
</details>

<details>
<summary><strong>Q: How do I enable verbose logging for debugging?</strong></summary>

A: Set environment variables for specific containers:

```yaml
services:

	microcks-postman-runtime:

		environment:

		- LOG_LEVEL=debug
```

Or via command line:

```sh
LOG_LEVEL=debug docker compose up -d
```
</details>

<details>
<summary><strong>Q: Can I use Microcks with Podman instead of Docker?</strong></summary>

A: Yes, with some modifications:
- Use podman-compose instead of docker compose
- Network configuration may need adjustments
- Host resolution might require different approaches

  
</details>

<details>

<summary><strong>Q: Why does the OpenAPI Schema runner work but Postman runner fails with the same endpoint?</strong></summary>

A: The OpenAPI Schema runner and Postman runner use different execution environments:

  

- OpenAPI Schema runner executes from the main Microcks container

- Postman runner executes in a separate container with different network context

- This can lead to different DNS resolution behavior
</details>

<details>
<summary><strong>Q: How do I know if async features are working correctly?</strong></summary>

A: Check that all async-related containers are running:

```sh 
docker compose ps | grep -E "(kafka|zookeeper|async-minion)" 
```

Verify Kafka connectivity:

```sh 
docker exec microcks-kafka kafka-topics.sh --list --bootstrap-server localhost:9092 
```
</details>