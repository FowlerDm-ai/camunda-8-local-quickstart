# Testing notes

These notes describe how the Camunda 8.8 local quickstart was validated and what changed in the documentation as a result.

## Test environment

| Item | Value |
|---|---|
| Date tested | 29 July 2026 |
| Camunda | 8.8.32 |
| Connectors bundle | 8.8.15 |
| Elasticsearch | 8.17.10 |
| Host | Windows |
| Container tooling | Docker Desktop and Docker Compose |
| Shell | PowerShell |
| Compose file | Lightweight `docker-compose.yaml` |

## Validation sequence

The following checks were used while developing the guide.

### 1. Confirm Docker tooling

```powershell
docker --version
docker compose version
docker run hello-world
```

Purpose: confirm that the Docker command-line tools and Docker engine were available before attempting the Camunda stack.

### 2. Validate the Compose file

```powershell
docker compose config --quiet
docker compose config
```

Purpose: catch configuration problems before startup and inspect the resolved configuration, including port mappings.

### 3. Start the stack

```powershell
docker compose up -d
```

Purpose: start the lightweight environment in detached mode.

### 4. Verify service health

```powershell
docker compose ps
```

Expected healthy services in the tested environment:

- `orchestration`
- `connectors`
- `elasticsearch`

For unhealthy or stopped services:

```powershell
docker compose ps --all
docker compose logs --tail 100 <service-name>
```

### 5. Verify the web applications

The following addresses were checked after the Orchestration Cluster became healthy:

- `http://localhost:8088/operate`
- `http://localhost:8088/tasklist`
- `http://localhost:8088/identity`

### 6. Verify the REST API

```powershell
curl.exe http://localhost:8088/v2/topology
```

The returned JSON was checked for:

- broker presence;
- partition information;
- a healthy partition;
- version information;
- `clusterSize`;
- `partitionsCount`;
- `replicationFactor`; and
- `gatewayVersion`.

A successful browser request to the same endpoint displayed raw JSON, which is expected behaviour.

## Finding: host port was 8088, not 8080

One of the useful findings from the test was the actual host-to-container port mapping.

`docker compose ps` showed a mapping in this form:

```text
0.0.0.0:8088->8080/tcp
```

That means:

- the service listens on port `8080` inside the container;
- Windows exposes it on host port `8088`; and
- local browser and API requests must therefore use port `8088`.

The tested API request was:

```powershell
curl.exe http://localhost:8088/v2/topology
```

This was carried through into the final guide so that the instructions reflected the observed environment rather than an assumed default.

## Documentation principle

Where the source documentation and the live local environment could be checked against each other, the guide was written around observed product behaviour.

The goal of the test was not simply to restate setup instructions, but to verify:

1. that the commands ran;
2. that the services became healthy;
3. that the browser applications were reachable;
4. that the REST endpoint returned valid topology data; and
5. that troubleshooting guidance matched the actual port mapping and Docker behaviour.

## Reproducing the validation

Using the same Camunda 8.8 lightweight Docker Compose distribution on Windows with Docker Desktop:

```powershell
docker compose config --quiet
docker compose up -d
docker compose ps
curl.exe http://localhost:8088/v2/topology
```

Then compare the resolved port mapping and API response with the expected behaviour documented above.

Exact patch versions and container networking details can change between releases, so those values should be re-checked rather than assumed.
