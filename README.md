# Camunda 8 Local Quickstart

A tested local-development guide for running and verifying **Camunda 8.8 Self-Managed** on Windows with Docker Desktop and Docker Compose.

This repository is an independent technical writing sample. It documents the setup I actually ran, including REST API validation and a host/container port-mapping issue identified during testing.

## What this demonstrates

- Running Camunda 8.8 locally with Docker Compose
- Checking service health
- Opening Operate, Tasklist and Identity
- Calling the `/v2/topology` REST endpoint
- Interpreting the returned JSON
- Diagnosing host-to-container port mapping
- Updating documentation to match observed product behaviour rather than assumed defaults

## Tested environment

| Component | Version / configuration |
|---|---|
| Camunda | 8.8.32 |
| Connectors bundle | 8.8.15 |
| Elasticsearch | 8.17.10 |
| Compose configuration | Lightweight `docker-compose.yaml` |
| Host environment | Windows, Docker Desktop and PowerShell |

Tested on **29 July 2026**.

## Files

- [`camunda-8-local-quickstart.md`](camunda-8-local-quickstart.md) — the complete tested guide
- [`testing-notes.md`](testing-notes.md) — what was tested, what was observed, and what changed as a result

## Scope

This guide covers local development and evaluation only. It is not production deployment guidance.

## Sources

The guide was created from:

- Camunda 8.8 Developer quickstart with Docker Compose
- Camunda 8.8 Orchestration Cluster REST API overview
- Docker Compose CLI documentation

Each instruction was checked against the local environment described above.

---

**Independent technical writing sample by Daniel Fowler. Not official Camunda documentation.**
