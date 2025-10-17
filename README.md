# RapidSpike OpenTelemetry Configurations

This repository provides **canonical OpenTelemetry Collector configurations** for sending data from [RapidSpike](https://rapidspike.com) monitoring endpoints into any OpenTelemetry-compatible observability backend.

These configurations demonstrate how to:
- Poll the RapidSpike API for metrics such as uptime, page performance, and Lighthouse scores.
- Parse JSON responses into OpenTelemetry metrics.
- Export those metrics to an OTLP endpoint (e.g. RapidSpike, Grafana, Datadog, or any compatible backend).

---

## 📦 What’s Included

| File | Description |
|------|--------------|
| `collectors/base-collector.yaml` | A minimal starting point for an OTel Collector instance. |
| `collectors/rapidspike-httpmon.yaml` | Fetches response time and status metrics from HTTP monitors. |
| `collectors/rapidspike-lighthouse.yaml` | Fetches SEO, accessibility, and performance scores from Lighthouse monitors. |
| `collectors/rapidspike-multi.yaml` | Combines both the above into a single multi-receiver collector. |

---

## 🚀 Quick Start

1. **Install or run the OTel Collector**

   ```bash
   docker run --rm -v $(pwd)/collectors:/etc/otel/config \
     -e RAPIDSPIKE_API_KEY=your-api-key \
     -e RAPIDSPIKE_BASE_URL=https://api.rapidspike.com \
     otel/opentelemetry-collector:latest \
     --config /etc/otel/config/rapidspike-httpmon.yaml

2. **Verify metrics output**

Check the collector logs — you should see data exported to your chosen OTLP endpoint.

3. **Modify or extend**

Copy one of the collector configs and replace {{monitor-uuid}} or {{page-uuid}} with your own monitor/page IDs.