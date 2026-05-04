# Brute-Force Login Detection

A security investigation using **Splunk** to detect brute-force SSH login attacks from real-world authentication logs.This work covers the full SOC analyst workflow data ingestion, field extraction, SPL-based detection queries, threat intelligence enrichment, and dashboard construction.

---

## Project Overview

Brute-force attacks (MITRE ATT&CK: T1110) are among the most common threats against internet-facing systems. This project analyses a real SSH authentication log to identify attacking IPs, targeted accounts, and successful compromises using 7 custom SPL detection queries.

**Tool:** Splunk  
**Dataset:** OpenSSH authentication log from the [LogHub repository](https://zenodo.org/records/8196385) (`SSH.tar.gz`), weeks of continuous SSH exposure on a public-facing Linux host  
**Log pre-processing:** A Python script using the `geoip2` library was used to tag each log entry with a country code (e.g. `[CN]`, `[US]`) before ingestion, enabling geographic attribution without real-time API calls

---

## Repository Contents

| File | Description |
|---|---|
| `Brute Force Detection Technical Report.pdf` | Full investigation report with methodology, SPL queries, and findings |
| `brute_force_detection_dashboard-2026-04-08.pdf` | Screenshot of the completed Splunk SOC dashboard |
| `ip_blocklist.csv` | Threat intelligence lookup table used to enrich attacking IP data |


---

## Detection Logic

The core detection principle: legitimate users fail authentication at most a handful of times. Automated attack tools generate hundreds to thousands of failures per minute.

Seven SPL queries were developed, each targeting a different dimension of the threat:

| Query | Purpose |
|---|---|
| 1 - Total Failed Logins | Baseline failure count with severity rating (MEDIUM / HIGH / CRITICAL) |
| 2 - Top Attacking IPs | Top 10 source IPs by failure count, enriched with blocklist lookup |
| 3 - Threshold Detection | Any IP with ≥5 failures in a 10-minute window flagged as confirmed attacker |
| 4 - Targeted Account Analysis | Most attacked usernames ranked by composite risk score (failures × unique attackers) |
| 5 - Attack Timeline | Time-series of failed logins in 30-minute buckets for the SOC dashboard |
| 6 - Compromise Detection | IPs with >3 failures **and** at least one success, the brute-force success signature |
| 7 - Geographic Attribution | Failed login volume aggregated by country of origin |

Full query code and line-by-line explanations are in the technical report.

---

## SOC Dashboard

A four-panel Splunk Classic Dashboard was built to surface the investigation findings operationally:

- **Panel 1 - Attack Timeline:** Full-width line chart of attack volume over time
- **Panel 2 - Top 10 Attacking IPs:** Table with IP, failure count, country, threat level, and analyst notes
- **Panel 3 - Most Targeted Accounts:** Risk-scored account table to drive password reset and MFA prioritisation
- **Panel 4 - Attack Source by Country:** Bar chart for geo-blocking decisions

See `brute_force_detection_dashboard-2026-04-08.pdf` for the completed dashboard.

---

## Author
- Muhammad Ashar Latif

