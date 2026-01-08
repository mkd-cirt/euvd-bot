# euvd-bot

This repository contains a **GitHub Actions–based automation created by Sevdali Selmani** for monitoring **Critical** and **Known Exploited** vulnerabilities published in the **ENISA EU Vulnerability Database (EUVD)** and delivering a concise feed to a **Slack channel via Incoming Webhook**.

The solution is intended for **CSIRTs, SOCs, regulators, and critical infrastructure operators** that require an authoritative, low-noise vulnerability awareness capability based on EUVD data.

---

## Overview

On a scheduled basis, the workflow:

1. Queries the official **ENISA EUVD API**.
2. Retrieves vulnerabilities that are:
   - **Critical** (CVSS base score ≥ 9.0), **or**
   - **Known Exploited** (`exploited=true`).
3. Limits results to the **last N hours** (default: 24 hours).
4. Deduplicates previously posted vulnerabilities to prevent repeated alerts.
5. Posts a structured **Slack Block Kit** message with a Markdown fallback.
6. Limits output to **a maximum of 10 vulnerabilities per run** to reduce alert fatigue.

---

## Data Source

- **EU Vulnerability Database (EUVD)** – ENISA  
- **API documentation:** https://euvd.enisa.europa.eu/apidoc  
- **API base URL:**
https://euvdservices.enisa.europa.eu/api

yaml

Only documented EUVD endpoints and parameters are used.

---

## Filtering Logic

### Critical vulnerabilities
- Retrieved using:
/search?fromScore=9&toScore=10

csharp
Copy code
- CVSS base score ≥ 9.0 is treated as **Critical**, in line with CVSS v3/v4 standards.

### Exploited vulnerabilities
- Retrieved using:
/search?exploited=true

yaml

### Logical OR implementation
As EUVD query parameters are applied using logical **AND**, the workflow performs **two separate searches** (Critical and Exploited) and merges the results client-side. Duplicates are removed using the **EUVD vulnerability ID** as the primary key.

---

## Slack Output

Each Slack message includes:

- Severity indicators: `[CRITICAL]`, `[EXPLOITED]`
- EUVD vulnerability ID with a direct link to the EUVD portal
- CVE / alias information (when available)
- CVSS base score and version
- Last updated timestamp

If more than 10 vulnerabilities match in a single execution, only the first 10 are posted, followed by a summary note indicating additional matches.

---

## Repository Structure

.
├── euvd_to_slack.py
├── requirements.txt
├── README.md
└── .github/
└── workflows/
└── euvd-slack.yml

yaml

---

## GitHub Actions

- Executed via **GitHub Actions** (no dedicated infrastructure required)
- Default schedule: **every 30 minutes**
- Manual execution supported (`workflow_dispatch`)
- Python version: **3.12**

---

## Required GitHub Secrets

Configure under:

**Repository → Settings → Secrets and variables → Actions**

| Secret | Description |
|------|-------------|
| `SLACK_WEBHOOK_URL` | Slack Incoming Webhook URL |
| `REDIS_URL` | Redis connection string for persistent deduplication |

Redis is strongly recommended to ensure consistent deduplication across GitHub Actions runs.

---

## Runtime Configuration

The workflow supports the following configuration variables:

| Variable | Default | Description |
|--------|---------|-------------|
| `LOOKBACK_HOURS` | `24` | Lookback window for vulnerability retrieval |
| `MAX_ITEMS` | `10` | Maximum vulnerabilities posted per run |
| `PAGE_SIZE` | `100` | EUVD API page size (maximum allowed) |
| `REQUEST_TIMEOUT` | `20` | HTTP request timeout (seconds) |
| `DRY_RUN` | `false` | Disable Slack posting if set to true |
| `LOG_LEVEL` | `INFO` | Logging verbosity |

---

## Deduplication Strategy

- Uses **Redis** to persist EUVD vulnerability IDs already posted to Slack
- Deduplication entries are retained for **180 days**
- Prevents duplicate notifications across overlapping execution windows

---

## Deployment Steps

1. Create a GitHub repository.
2. Add the repository files.
3. Create a Slack Incoming Webhook for the target channel.
4. Add the required GitHub Secrets:
   - `SLACK_WEBHOOK_URL`
   - `REDIS_URL`
5. Push changes to the default branch.
6. Trigger the workflow manually or wait for the scheduled execution.

---

## Operational Notes

- Recommended execution interval: **30–60 minutes**
- `LOOKBACK_HOURS` should be equal to or greater than the execution interval
- Use a dedicated Slack channel for vulnerability awareness
- Designed for **signal over noise** in operational environments

---

## Limitations

- Slack Incoming Webhooks do not support threaded replies or message updates
- EUVD date filtering is date-based; precise hour-based filtering is enforced client-side

These limitations are expected and handled by design.

---

## License

MIT License

---

**Created and maintained by Sevdali Selmani**
