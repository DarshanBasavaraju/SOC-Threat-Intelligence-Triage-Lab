# SOC-Threat-Intelligence-Triage-Lab
SOC threat intelligence enrichment and triage simulation using Splunk

## Overview

This project simulates a real-world SOC workflow for detecting
phishing, impersonation, and credential harvesting domains
using Splunk lookup enrichment.

Inspired by real-world experience in:

- Brand impersonation detection
- Phishing infrastructure analysis
- Threat intelligence enrichment
- Incident triage workflows

---

## Detection Logic

Threat enrichment:

index=proxy_logs
| lookup threat_lookup domain OUTPUT threat_type confidence
| search threat_type=*

Infected IP detection:

| stats count by src_ip
| sort -count

---

## Dashboard Overview
![Dashboard](screenshots/dashboard_overview.png)

## Top Infected IPs Panel
![Infected IPs](screenshots/infected_ips_panel.png)

## Threat Distribution Panel
![Threat Distribution](screenshots/threat_distribution_panel.png)

## Timeline Panel
![Timeline](screenshots/timeline_panel.png)

## Key Outcomes

- Built threat intelligence lookup integration
- Created detection logic for malicious domains
- Identified infected internal hosts
- Simulated SOC triage workflow
