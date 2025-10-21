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

## HTTP Monitors

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

### 4. Get All Monitors Live Status

**Endpoint:**

```GET /v1/wallboards/http/websites/availability```

**Purpose:**  
Retrieves a list of all monitors configured (by website) and their live status.

### Metrics

| Metric                                      | Type  | Description                                       | Attributes                                                 |
| ------------------------------------------- | ----- | ------------------------------------------------- | ---------------------------------------------------------- |
| `rapidspike.websites.total`                 | Sum   | Total number of websites in the account           | —                                                          |
| `rapidspike.websites.with_monitors`         | Sum   | Websites that have at least one HTTP monitor      | —                                                          |
| `rapidspike.websites.http_monitors_total`   | Sum   | Total number of HTTP monitors across all websites | —                                                          |
| `rapidspike.websites.http_monitors_passing` | Sum   | Total number of passing HTTP monitors             | —                                                          |
| `rapidspike.websites.http_monitors_failing` | Sum   | Total number of failing HTTP monitors             | —                                                          |
| `rapidspike.websites.http_monitors_paused`  | Sum   | Total number of paused HTTP monitors              | —                                                          |
| `rapidspike.websites.uptime_status`         | Gauge | Website health (1=passing, 0=failing, -1=unknown) | `website_uuid`, `website_label`, `domain_name`             |
| `rapidspike.websites.pages`                 | Gauge | Number of HTTP pages monitored per website        | `website_uuid`, `website_label`, `domain_name`             |
| `rapidspike.websites.page_status`           | Gauge | Page-level uptime (1=passing, 0=failing)          | `website_uuid`, `website_label`, `page_label`, `page_path` |

**Typical Collection Interval:** Every 5 minutes.

---

## Lighthouse Monitors

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

## User Journeys

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
Retrieves the load times for a single journey.

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

### 3. Get All Monitor Events

**Endpoint:**

```GET /v1/wallboards/journeys/events```

**Purpose:**  
Retrieves recent User Journey monitor events for an account, showing status changes and latest results.

| Metric                                 | Type  | Description                                        | Attributes                         |
| -------------------------------------- | ----- | -------------------------------------------------- | ---------------------------------- |
| `rapidspike.journeys.total_journeys`   | Sum   | Total number of monitored journeys                 | —                                  |
| `rapidspike.journeys.total_events`     | Sum   | Number of journey event entries in the time window | —                                  |
| `rapidspike.journeys.passing_count`    | Sum   | Number of journeys currently passing               | —                                  |
| `rapidspike.journeys.failing_count`    | Sum   | Number of journeys currently failing               | —                                  |
| `rapidspike.journeys.error_count`      | Sum   | Number of journeys with active errors              | —                                  |
| `rapidspike.journeys.warning_count`    | Sum   | Number of journeys with warnings                   | —                                  |
| `rapidspike.journeys.duration_seconds` | Gauge | Duration of the journey’s event period (seconds)   | `journey_uuid`, `label`, `website` |
| `rapidspike.journeys.latest_status`    | Gauge | `1=passing`, `0=failing`, `-1=unknown`             | `journey_uuid`, `label`, `website` |
| `rapidspike.journeys.avg_total_time`   | Gauge | Journey test runtime (ms)                          | `journey_uuid`, `label`, `region`  |

**Typical Collection Interval:** Every 5 minutes.

--

## Websites

### 1. Get One Monitor's Stats

**Endpoint:**

```GET /v1/websites/{{website-uuid}}/stats```

**Purpose:**  
Provides high-level website uptime and monitor statistics.

### Metrics

| Metric                                            | Type  | Description                                                          | Attributes |
| ------------------------------------------------- | ----- | -------------------------------------------------------------------- | ---------- |
| `rapidspike.website.total_monitors`               | Gauge | Total number of monitors under this website                          | —          |
| `rapidspike.website.passing_monitors`             | Gauge | Number of monitors passing                                           | —          |
| `rapidspike.website.failing_monitors`             | Gauge | Number of monitors failing                                           | —          |
| `rapidspike.website.average_response_ms`          | Gauge | Average response time across monitors                                | —          |
| `rapidspike.website.average_uptime_percentage`    | Gauge | Average uptime %                                                     | —          |
| `rapidspike.website.total_status_changes`         | Sum   | Total number of monitor status changes                               | —          |
| `rapidspike.website.total_events`                 | Sum   | Total number of monitor events                                       | —          |
| `rapidspike.website.time_since_last_change_hours` | Gauge | Time since last status change (hours)                                | `date`     |
| `rapidspike.website.status_state`                 | Gauge | Website state (`1 = all_passing`, `0 = has_failing`, `-1 = unknown`) | —          |

**Typical Collection Interval:** Every 15 minutes.

## Sitemaps

### 1. Get One Sitemap's Tests

**Endpoint:**

```GET /v1/bulkurlmonitor/{{bulk-url-uuid}}/tests```

**Purpose:**  
Retrieves recent Bulk URL (sitemap) test results, including total and tested URLs per region.

### Metrics

| Metric                                    | Type  | Description                                        | Attributes                              |
| ----------------------------------------- | ----- | -------------------------------------------------- | --------------------------------------- |
| `rapidspike.bulkurl.total_potential_urls` | Gauge | Number of URLs discovered in sitemap               | `region`, `region_code`, `created_date` |
| `rapidspike.bulkurl.total_tested_urls`    | Gauge | Number of URLs successfully tested                 | `region`, `region_code`, `created_date` |
| `rapidspike.bulkurl.test_coverage_ratio`  | Gauge | Ratio of tested URLs to total potential URLs (0–1) | `region`, `region_code`, `created_date` |
| `rapidspike.bulkurl.test_runs`            | Sum   | Total number of test runs recorded                 | `region`, `region_code`                 |

**Typical Collection Interval:** Every 24 hours.

### 2. Get One Sitemap's Test Result

**Endpoint:**

```GET /v1/bulkurlmonitor/{{bulk-url-uuid}}/tests/{{test-uuid}}```

**Purpose:**  
Fetches detailed URL-level test results, including response codes and timings for each page in the sitemap.

### Metrics

| Metric                                | Type  | Description                                  | Attributes                                                   |
| ------------------------------------- | ----- | -------------------------------------------- | ------------------------------------------------------------ |
| `rapidspike.bulkurl.response_time_ms` | Gauge | Response time in milliseconds per tested URL | `region`, `region_code`, `url`, `response_code`, `unix_time` |
| `rapidspike.bulkurl.response_code`    | Gauge | HTTP response code per tested URL            | `region`, `region_code`, `url`, `unix_time`                  |
| `rapidspike.bulkurl.url_count`        | Sum   | Number of URLs tested                        | `region`, `region_code`                                      |
| `rapidspike.bulkurl.failed_responses` | Sum   | Count of failed responses (≥400)             | `region`, `region_code`                                      |

**Typical Collection Interval:** Every 24 hours.






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
