# RapidSpike OpenTelemetry Metric Reference

This document explains how RapidSpike’s API endpoints map to OpenTelemetry metric data using the example collector configurations in this repository.

Each collector polls a different monitoring endpoint (Uptime, Lighthouse, and User Journeys) and transforms the JSON responses into standardized OpenTelemetry **gauge metrics**.

---

## 🔐 Authentication Reminder

All collector configurations use **HMAC-signed API authentication**.

Each request must include:

- `public_key` — your public API key  
- `time` — the current Unix timestamp  
- `signature` — an HMAC-SHA1 hash of your public key + timestamp, signed with your private key  

You can generate these values using your own script or proxy service.
For details, see [docs/auth.md](./auth.md).

---

## 🧭 1. HTTP Monitors

**Endpoint:**

```GET /v1/httpmonitors/{{monitor-uuid}}/latest```


**Purpose:**  
Fetches recent results from an Uptime (HTTP) monitor, including HTTP status, response time, and content check outcomes.

### Metrics

| Metric | Type | Description | Attributes |
|---------|------|-------------|-------------|
| `rapidspike.uptime.response_time_ms` | Gauge | Response time in milliseconds for each location check. | `location`, `country` |
| `rapidspike.uptime.success` | Gauge | Success state (1 = success, 0 = failure). | `location`, `country` |

**Example:**  
A value of `rapidspike.uptime.response_time_ms{location="Cleveland"} 64.7` represents a 64.7 ms page response time for the Cleveland region.

---

## 🌐 2. Lighthouse Monitors

**Endpoint:**

```GET /v1/pages/{{page-uuid}}/monitors/lighthousetest/results```


**Purpose:**  
Retrieves recent Google Lighthouse test scores for a specific page, including SEO, performance, and accessibility metrics.

### Metrics

| Metric | Type | Description | Attributes |
|---------|------|-------------|-------------|
| `rapidspike.lighthouse.performance` | Gauge | Performance score (0–100). | `region`, `device`, `location` |
| `rapidspike.lighthouse.accessibility` | Gauge | Accessibility score (0–100). | `region`, `device`, `location` |
| `rapidspike.lighthouse.seo` | Gauge | SEO score (0–100). | `region`, `device`, `location` |
| `rapidspike.lighthouse.pwa` | Gauge | Progressive Web App compliance score (0–100). | `region`, `device`, `location` |
| `rapidspike.lighthouse.best_practices` | Gauge | Best practices score (0–100). | `region`, `device`, `location` |

**Typical Collection Interval:** Every 6 hours.

---

## 🧩 3. User Journeys

**Endpoint:**

```GET /v1/journeys/{{journey-uuid}}/samples?page=1&per_page=1&regions=````


**Purpose:**  
Retrieves the latest test sample for a scripted multi-step synthetic User Journey, including timing data and error counts.

### Metrics

| Metric | Type | Description | Attributes |
|---------|------|-------------|-------------|
| `rapidspike.journey.total_time_ms` | Gauge | Total time for the entire journey. | `region`, `browser`, `comment` |
| `rapidspike.journey.step_time_ms` | Gauge | Duration of the last executed step. | `region`, `browser` |
| `rapidspike.journey.dom_time_ms` | Gauge | DOM content load time. | `region`, `browser` |
| `rapidspike.journey.load_time_ms` | Gauge | Page load time. | `region`, `browser` |
| `rapidspike.journey.render_time_ms` | Gauge | Time to first render completion. | `region`, `browser` |
| `rapidspike.journey.errors_total` | Gauge | Number of detected errors in the journey. | `region`, `browser` |
| `rapidspike.journey.error_breakdown_total` | Gauge | Total number of error-like events (errors + warnings). | `region`, `browser` |
| `rapidspike.journey.error_breakdown_errors` | Gauge | Critical errors. | `region`, `browser` |
| `rapidspike.journey.error_breakdown_warnings` | Gauge | Warning-level issues. | `region`, `browser` |
| `rapidspike.journey.error_breakdown_failures` | Gauge | Step or journey-level failures. | `region`, `browser` |

**Typical Collection Interval:** Every 5 minutes.

---

## 🧠 Metric Naming Convention

RapidSpike metrics use the format:

`rapidspike.<category>.<metric_name>`


Where:
- **category** corresponds to the monitor type (`uptime`, `lighthouse`, `journey`, etc.).
- **metric_name** is in lower case with underscores.
- All values are numeric (gauges by default).

Examples:
- `rapidspike.uptime.response_time_ms`
- `rapidspike.lighthouse.performance`
- `rapidspike.journey.total_time_ms`

---


For setup and authentication details, see:
- [docs/setup.md](./setup.md)
- [docs/auth.md](./auth.md)
