# RapidSpike OpenTelemetry Metric Reference

This document explains how RapidSpike’s API endpoints map to OpenTelemetry metric data using the example collector
configurations in this repository.

Each collector polls a different monitoring endpoint (Uptime, Lighthouse, and User Journeys) and transforms the JSON
responses into standardized OpenTelemetry **gauge metrics**.

---

## 🔐 Authentication Reminder

All collector configurations use **HMAC-signed API authentication**.

Each request must include:

- `public_key` — your public API key
- `time` — the current Unix timestamp
- `signature` — an HMAC-SHA1 hash of your public key + timestamp, signed with your private key

You can generate these values using your own script or proxy service. For details, see [docs/auth.md](./auth.md).

---

## 🧭 HTTP Monitors

### 1. Get One Monitor's results

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
A value of `rapidspike.uptime.response_time_ms{location="Cleveland"} 64.7` represents a 64.7 ms page response time for
the Cleveland region.

---

### 2. Get One Monitor's load times

**Endpoint:**

```GET /v1/httpmonitors/{{monitor-uuid}}/responses```

**Purpose:**  
Fetches daily response time metrics for a specific HTTP monitor.

### Metrics

| Metric                                        | Type  | Description                                   | Attributes          |
| --------------------------------------------- | ----- | --------------------------------------------- | ------------------- |
| `rapidspike.httpmon.average_response_time_ms` | Gauge | Average response time (ms) for the day        | `date`, `unix_date` |
| `rapidspike.httpmon.results_total`            | Gauge | Number of results contributing to the average | `date`, `unix_date` |

---

### 3. Get One Monitor's status events

**Endpoint:**

```GET /v1/httpmonitors/{{monitor-uuid}}/statuses```

**Purpose:**  
Retrieves a list of monitor status changes (“passing”, “failing”), including timestamps and notification info.

### Metrics

| Metric                                 | Type  | Description                                   | Attributes                                                          |
| -------------------------------------- | ----- | --------------------------------------------- | ------------------------------------------------------------------- |
| `rapidspike.httpmon.status_change`     | Gauge | Monitor status (1 = passing, 0 = failing)     | `monitor_uuid`, `website_uuid`, `created_date`, `unix_created_date` |
| `rapidspike.httpmon.notification_sent` | Gauge | Notification flag (1 = sent, 0 = not sent)    | `monitor_uuid`, `website_uuid`, `created_date`, `unix_created_date` |
| `rapidspike.httpmon.region_count`      | Gauge | Number of active test regions for the monitor | `monitor_uuid`, `website_uuid`, `created_date`                      |

---

## 🌐 Lighthouse Monitors

### 1. Get One Monitor's Results

**Endpoint:**

```GET /v1/pages/{{page-uuid}}/monitors/lighthousetest/results```

**Purpose:**  
Retrieves recent Google Lighthouse test scores for a specific page, including SEO, performance, and accessibility
metrics.

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

## 🧩 User Journeys

### 1. Get One Monitor's Results

**Endpoint:**

```GET /v1/journeys/{{journey-uuid}}/samples?page=1&per_page=1&regions=````

**Purpose:**  
Retrieves the latest test sample for a scripted multi-step synthetic User Journey, including timing data and error
counts.

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

### 2. Get One Monitor's Load Times

**Endpoint:**

```GET /v1/journeys/{{journey-uuid}}/loadtimes````

**Purpose:**  
Retrieves the

| Metric                                          | Type  | Description                             | Attributes          |
| ----------------------------------------------- | ----- | --------------------------------------- | ------------------- |
| `rapidspike.journeyload.total_time_ms`          | Gauge | Total time to complete the journey (ms) | `region`, `browser` |
| `rapidspike.journeyload.on_load_time_ms`        | Gauge | On-load time for all steps (ms)         | `region`, `browser` |
| `rapidspike.journeyload.on_load_ms`             | Gauge | Browser’s reported on-load time (ms)    | `region`, `browser` |
| `rapidspike.journeyload.success`                | Gauge | Success (1 = success, 0 = fail)         | `region`, `browser` |
| `rapidspike.journeyload.error_count`            | Gauge | Total number of errors                  | `region`, `browser` |
| `rapidspike.journeyload.warning_count`          | Gauge | Total number of warnings                | `region`, `browser` |
| `rapidspike.journeyload.elements_total_count`   | Gauge | Total number of page elements           | `region`, `browser` |
| `rapidspike.journeyload.elements_total_size_kb` | Gauge | Combined size of all page elements (kB) | `region`, `browser` |

**Typical Collection Interval:** Every 60 minutes.


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
