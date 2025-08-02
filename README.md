# Example Observability Server

This project provides a simple, all-in-one observability stack that can be run locally with Docker Compose. It is designed for developers who need a ready-to-use backend for instrumented applications during local development and testing.

It provides a complete environment for collecting and visualizing traces, logs, and metrics (the "three pillars of observability").

## Components Included

This stack is built on the Grafana LGTM stack and includes:

-   **Grafana**: For visualization, dashboards, and exploring telemetry.
-   **Loki**: For log aggregation.
-   **Grafana Tempo**: For distributed tracing.
-   **Mimir**: For metrics.
-   **OpenTelemetry Collector**: A single, vendor-agnostic endpoint to receive all your telemetry data (`traces`, `metrics`, `logs`) and forward it to the appropriate backend (Tempo, Loki, Mimir).

## Prerequisites

-   **Docker Engine**: [Install Docker](https://docs.docker.com/engine/install/)
-   **Docker Compose V2**: The `docker compose` command is now included with Docker Desktop. For Linux, it can be installed as a plugin. See the [official installation instructions](https://docs.docker.com/compose/install/).
-   **Grafana Loki Docker Plugin**: This is required to collect logs from Docker containers.

### Installing the Loki Plugin

Run the following command to install the plugin. You can accept the default prompts for the plugin configuration.

```sh
docker plugin install grafana/loki-docker-driver:latest --alias loki --grant-all-permissions
```

## How to Run

1.  Navigate to this directory.
2.  Start all services in the background:
    ```sh
    docker compose up -d
    ```

> **A Note on `docker-compose`**
>
> This guide uses the modern `docker compose` (with a space), which is part of the Docker CLI. The older, standalone `docker-compose` (with a hyphen) may still work, but it is considered legacy and is not officially supported by this documentation.

## How to Use

Once the stack is running, you can configure your instrumented applications to send telemetry to the OpenTelemetry Collector.

### Connecting from Other Projects

To send telemetry from an application running in a different Docker Compose project, the recommended method is to add a special DNS entry to your application's container that points to the host machine.

In your application's `docker-compose.yaml` file, add an `extra_hosts` entry to your service definition. This works on Docker Engine 20.10+ across all platforms (Linux, Mac, Windows).

```yaml
# your-app/docker-compose.yaml

services:
  my-app-service:
    # ... your other service configuration
    extra_hosts:
      - "host.docker.internal:host-gateway"
```

Then, set the OTLP endpoint in your application's configuration (e.g., in its `.env` file or `docker-compose.yaml` environment variables) to use this DNS name:

-   **OTLP Endpoint URL**: `http://host.docker.internal:4318`

This is the most reliable method for cross-compose communication during local development.

### Service Endpoints

-   **Grafana UI**: [http://localhost:3000](http://localhost:3000)
    -   **Username**: `admin`
    -   **Password**: `admin`
    -   *(Note: Default credentials are set in the `.env` file)*

The datasources for Loki, Tempo, and Mimir are pre-provisioned in Grafana, so you can immediately start exploring your data.

## How to Stop

To stop and remove all the running containers, run:

```sh
docker compose down
```

## Configuration

The configuration for each component (`grafana`, `loki`, `tempo`, `mimir`, `otel-collector`) can be found in its respective subdirectory. The `compose.yaml` file defines how the services are orchestrated.
