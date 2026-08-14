# Run and verify Camunda 8.8 locally with Docker Compose

Use this guide to start the lightweight Camunda 8.8 Self-Managed environment on Windows with Docker Desktop. You will confirm that its services are healthy, open the integrated web applications, and test the Orchestration Cluster REST API.

By the end, you will be able to:

- validate the Docker Compose configuration;
- start Camunda in the background;
- interpret container health and port mappings;
- open Operate, Tasklist and Identity; and
- verify the cluster through a live API request.

## Local development only

Camunda's Docker Compose files are intended for local development and evaluation. Use a supported production deployment approach for production environments.

## Validated configuration

This guide was tested on 29 July 2026 with:

| Component | Version or configuration |
|---|---|
| Camunda | 8.8.32 |
| Connectors bundle | 8.8.15 |
| Elasticsearch | 8.17.10 |
| Compose configuration | Lightweight `docker-compose.yaml` |
| Host environment | Windows, Docker Desktop and PowerShell |

Exact patch versions may change when the Camunda 8.8 Docker Compose distribution is updated.

## Before you begin

You need:

- Docker Desktop installed and running;
- Docker Compose available through the `docker compose` command;
- the Camunda 8.8 Docker Compose distribution downloaded and extracted; and
- PowerShell or another terminal opened in the extracted directory.

The lightweight configuration starts three services: the Orchestration Cluster, Connectors and Elasticsearch. In Camunda 8.8, Operate, Tasklist and Identity are integrated into the Orchestration Cluster service.

## 1. Confirm Docker is available

Run:

```powershell
docker --version
docker compose version
```

Both commands should return version information.

To verify the Docker engine as well as the command-line tools, run the optional test container:

```powershell
docker run hello-world
```

The container prints a success message and then stops. A stopped `hello-world` container in Docker Desktop is expected.

## 2. Open PowerShell in the Compose directory

In File Explorer, open the folder that contains `docker-compose.yaml`. Click the address bar, type `powershell`, and press Enter.

Confirm that the Compose file is present:

```powershell
dir
```

If the file is not listed, move to the correct extracted directory before continuing.

## 3. Validate the configuration

Run:

```powershell
docker compose config --quiet
```

A successful validation returns control to the prompt without displaying an error.

To inspect the fully resolved configuration, including ports and environment variables, run:

```powershell
docker compose config
```

Resolve any missing variables, invalid attributes or file errors before starting the stack.

## 4. Start Camunda

Run:

```powershell
docker compose up -d
```

The `-d` option starts the containers in detached mode, so they remain active after the PowerShell prompt returns.

The first run takes longer because Docker downloads the required images and creates a network and persistent volumes.

When the command completes, it should report that the containers have started. Initialising the Orchestration Cluster can take several minutes.

## 5. Check service health

Run:

```powershell
docker compose ps
```

In the validated environment, the output included these services:

| Service | Expected state | Purpose |
|---|---|---|
| `orchestration` | `Up ... (healthy)` | Runs the process engine, web applications and APIs |
| `connectors` | `Up ... (healthy)` | Runs inbound and outbound connectors |
| `elasticsearch` | `Up ... (healthy)` | Provides secondary storage and search |

Do not continue while a required service is starting, unhealthy, restarting or exited.

Wait briefly, then inspect its logs:

```powershell
docker compose logs --tail 100 <service-name>
```

Replace `<service-name>` with the value shown in the `SERVICE` column.

## 6. Open the web applications

Open these addresses in a browser:

| Application | Address | Purpose |
|---|---|---|
| Operate | `http://localhost:8088/operate` | Monitor and troubleshoot process instances |
| Tasklist | `http://localhost:8088/tasklist` | Complete user tasks |
| Identity | `http://localhost:8088/identity` | Manage users and permissions |

Use the default web credentials when prompted:

- Username: `demo`
- Password: `demo`

If the pages do not load immediately, wait until `docker compose ps` shows the Orchestration Cluster as healthy.

## 7. Verify the REST API

In PowerShell, request the cluster topology:

```powershell
curl.exe http://localhost:8088/v2/topology
```

Use `curl.exe` rather than `curl` in Windows PowerShell to avoid calling a PowerShell alias instead of the curl executable.

A successful request returns JSON similar to this shortened example:

```json
{
  "brokers": [
    {
      "nodeId": 0,
      "partitions": [
        {
          "partitionId": 1,
          "role": "leader",
          "health": "healthy"
        }
      ],
      "version": "8.8.32"
    }
  ],
  "clusterSize": 1,
  "partitionsCount": 1,
  "replicationFactor": 1,
  "gatewayVersion": "8.8.32"
}
```

The container IP address may differ between runs. Focus on the health, version, cluster size and partition information.

### Raw JSON in the browser is expected

You can also open:

```text
http://localhost:8088/v2/topology
```

in a browser. A plain white page displaying raw JSON is a successful API response, not a broken webpage.

# Troubleshooting

## The API does not respond on port 8080

Check the `PORTS` column:

```powershell
docker compose ps
```

A mapping such as:

```text
0.0.0.0:8088->8080/tcp
```

means that Windows exposes the service on host port `8088`, while the application listens on port `8080` inside the container.

Use the host-side port on the left:

```powershell
curl.exe http://localhost:8088/v2/topology
```

## A container is not healthy

List current and stopped containers:

```powershell
docker compose ps --all
```

Then inspect the affected service:

```powershell
docker compose logs --tail 100 <service-name>
```

Read from the first error rather than only the final line. Later failures are often consequences of an earlier configuration or dependency problem.

To follow new log entries while a service starts:

```powershell
docker compose logs --follow <service-name>
```

Press `Ctrl+C` to stop following the logs. This does not stop the containers.

## Docker reports that a port is already allocated

The error identifies the host port in use.

Stop the conflicting application or change the host-side port in `docker-compose.yaml`. Validate the file again before restarting:

```powershell
docker compose config --quiet
docker compose up -d
```

## Stop or reset the environment

Stop and remove the containers while retaining their volumes:

```powershell
docker compose down
```

To remove the containers and all persisted local data:

```powershell
docker compose down -v
```

### Use `-v` with care

The `-v` option deletes volumes, including local users and process data. Use it only when you intend to reset the environment.

# Result

You have started a local Camunda 8.8 environment, confirmed that all three services are healthy, opened the integrated web applications, and verified the Orchestration Cluster through the live `/v2/topology` endpoint.

## Sources

- Camunda 8.8 Developer quickstart with Docker Compose
- Camunda 8.8 Orchestration Cluster REST API overview
- Docker Compose CLI documentation

**Independent technical writing sample by Daniel Fowler. Not official Camunda documentation.**
