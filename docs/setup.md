# RapidSpike OpenTelemetry Setup Guide

This guide explains how to set up the OpenTelemetry Collector using RapidSpike’s example configurations.  
It covers installation, authentication, environment variables, and running the collector with Docker.

---

## 🧱 1. Overview

The OpenTelemetry Collector is a lightweight service that can:

- Poll RapidSpike’s public API for monitoring data (Uptime, Lighthouse, and User Journeys)
- Transform JSON responses into OpenTelemetry metrics
- Export those metrics to your preferred observability platform (Prometheus, Grafana, Datadog, etc.)

Each configuration in this repository is a self-contained example of how to collect data from a specific RapidSpike API endpoint.

---

## ⚙️ 2. Prerequisites

Before starting, you’ll need:

- A **RapidSpike API key pair** (Public + Private keys)  
- [Docker](https://docs.docker.com/get-docker/) installed locally  
- (Optional) Access to a metrics backend such as **Prometheus**, **Grafana**, or **Datadog**

---

## 🔐 3. Authentication

RapidSpike’s API uses **HMAC-signed authentication**.  
Each request must include:

| Parameter | Description |
|------------|-------------|
| `public_key` | Your public RapidSpike API key |
| `time` | Current Unix timestamp |
| `signature` | Base64-encoded HMAC-SHA1 signature generated with your private key |

See [docs/auth.md](./auth.md) for full details and sample signing code.

You can generate these values manually with a signing script, or run a small proxy that signs API requests automatically.

---

## 🌍 4. Environment Variables

All collector examples rely on the following environment variables:

| Variable | Description | Example |
|-----------|-------------|----------|
| `RAPIDSPIKE_BASE_URL` | Base URL for the RapidSpike API | `https://api.rapidspike.com` |
| `RAPIDSPIKE_PUBLIC_KEY` | Your public API key | `rs_public_abc123` |
| `RAPIDSPIKE_PRIVATE_KEY` | *(Optional)* private key used for signing (if using a proxy) | `rs_private_xyz789` |
| `RAPIDSPIKE_TIME` | Current Unix timestamp for signature validity | `1760364499` |
| `RAPIDSPIKE_SIGNATURE` | Base64-encoded HMAC signature | `z5ejsdf8a9L…` |

You can export these variables before running the Collector, or inject them at runtime.

Example:
```bash
export RAPIDSPIKE_BASE_URL=https://api.rapidspike.com
export RAPIDSPIKE_PUBLIC_KEY=your-public-key
export RAPIDSPIKE_TIME=$(date +%s)
export RAPIDSPIKE_SIGNATURE=$(php sign.php)
```

## 🧩 5. Choosing a Collector Configuration

Choose one of the pre-built collector configs from the `/collectors` folder:

| Config File | Description |
|--------------|-------------|
| `rapidspike-httpmon.yaml` | Collects Uptime (HTTP) monitor results |
| `rapidspike-lighthouse.yaml` | Collects Lighthouse test scores |
| `rapidspike-journey.yaml` | Collects User Journey metrics |
| `rapidspike-multi.yaml` | Collects all the above in a single pipeline |

You can run any of them individually or together depending on which RapidSpike features you use.

---

## 🚀 6. Running the Collector (Docker)

Run the OpenTelemetry Collector using Docker and one of the example configurations:

```bash
docker run --rm \
  -v $(pwd)/collectors:/etc/otel/config \
  -e RAPIDSPIKE_BASE_URL \
  -e RAPIDSPIKE_PUBLIC_KEY \
  -e RAPIDSPIKE_TIME \
  -e RAPIDSPIKE_SIGNATURE \
  otel/opentelemetry-collector:latest \
  --config /etc/otel/config/rapidspike-httpmon.yaml
```

If you use rapidspike-multi.yaml, the collector will gather data from all endpoints (HTTP, Lighthouse, and Journeys) simultaneously.


## 🪄 7. Exporting Data

All example configurations include an exporter block that defines where the OpenTelemetry Collector sends the metrics after polling the RapidSpike API.

RapidSpike itself does not currently provide an OTLP ingestion endpoint.
Instead, you should configure the collector to export metrics to your chosen observability platform.

Example Destinations

You can replace the exporter section in any collector YAML with one of the following options:

**Export to a local Prometheus instance**

```exporters:
  prometheus:
    endpoint: "0.0.0.0:9464"
```
**Export to Grafana Cloud (OTLP)**

```
exporters:
  otlphttp:
    endpoint: "https://otlp-gateway.grafana.net/otlp"
    headers:
      Authorization: "Bearer ${GRAFANA_CLOUD_API_KEY}"
```
**Export to a local file for testing**

```
exporters:
  file:
    path: "/tmp/rapidspike-metrics.json"
```

---

## 🧰 8. Testing & Troubleshooting

### Validate Configuration Syntax
Before running, test that your configuration syntax is valid:
```bash
otelcontribcol --config /etc/otel/config/rapidspike-httpmon.yaml --dry-run
```

**Enable Debug Logging**

To view detailed collector output, add a logging exporter:

```
exporters:
  logging:
    loglevel: debug
```

