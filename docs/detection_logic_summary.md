# Detection Logic Summary

## 1. Objective

This project simulates a Security Operations Center (SOC) use case where internal proxy traffic is monitored and enriched with external threat intelligence to detect potentially compromised hosts communicating with malicious domains.

The objectives are to:

- Detect access to phishing and malicious domains
- Identify potentially infected internal hosts
- Prioritize events based on threat intelligence confidence
- Visualize malicious activity trends over time
- Demonstrate structured SOC triage and detection engineering logic

---

## 2. Data Sources

### 2.1 Proxy Logs (index=proxy_logs)

The proxy logs simulate outbound web traffic from internal users.

Primary fields used:

- `src_ip` → Internal host IP address
- `domain` → Destination domain accessed
- `_time` → Event timestamp

These logs represent user browsing activity exiting the network perimeter.

---

### 2.2 Threat Intelligence Feed (threat_feed.csv)

The threat intelligence feed is implemented as a Splunk lookup table.

Fields:

- `domain` → Malicious domain indicator
- `threat_type` → Threat category (phishing, impersonation, credential harvesting)
- `confidence` → Reliability score of the indicator

This lookup simulates an external threat intelligence provider.

---

## 3. Threat Enrichment Logic

Threat enrichment is performed using a Splunk lookup that correlates proxy traffic with known malicious indicators.

### Base Enrichment Query

```spl
index=proxy_logs
| lookup threat_lookup domain OUTPUT threat_type confidence
```

### How Enrichment Works

- The `domain` field from proxy logs is matched against the `domain` field in the lookup table.
- If a match exists:
  - `threat_type` is appended to the event.
  - `confidence` is appended to the event.
- If no match exists:
  - The fields remain null.

This process transforms raw web traffic into enriched security events ready for detection logic.

---

## 4. Detection Logic

### 4.1 Malicious Domain Access Detection

```spl
index=proxy_logs
| lookup threat_lookup domain OUTPUT threat_type confidence
| search threat_type=*
```

Purpose:

- Identify users accessing known malicious domains.
- Detect phishing attempts.
- Detect credential harvesting activity.
- Identify brand impersonation exposure.

---

### 4.2 Infected Host Identification

```spl
index=proxy_logs
| lookup threat_lookup domain OUTPUT threat_type confidence
| search threat_type=*
| stats count by src_ip
| sort -count
```

Purpose:

- Identify internal hosts communicating with malicious infrastructure.
- Detect repeated suspicious behavior.
- Prioritize hosts based on activity frequency.

Higher event count indicates higher probability of compromise.

---

### 4.3 Threat Distribution Analysis

```spl
index=proxy_logs
| lookup threat_lookup domain OUTPUT threat_type confidence
| search threat_type=*
| stats count by threat_type
```

Purpose:

- Identify dominant attack vectors.
- Measure phishing vs impersonation activity.
- Support SOC reporting and threat trend analysis.

---

### 4.4 Timeline Analysis

```spl
index=proxy_logs
| lookup threat_lookup domain OUTPUT threat_type confidence
| search threat_type=*
| timechart count by threat_type
```

Purpose:

- Detect spikes in malicious domain access.
- Identify coordinated phishing campaigns.
- Monitor attack patterns over time.

---

## 5. Confidence-Based Prioritization

The `confidence` field determines investigation urgency.

Confidence Levels:

- **High** → Verified malicious domain (Immediate SOC investigation required)
- **Medium** → Suspicious domain with moderate threat reporting
- **Low** → Newly observed or weakly correlated indicator

High-confidence matches should trigger prioritized response.

---

## 6. Alert Trigger Conditions

An alert should trigger if:

1. A high-confidence malicious domain is accessed.
2. A single host accesses multiple malicious domains.
3. A spike in malicious activity occurs within a short timeframe.
4. Repeated access attempts to credential harvesting domains are observed.

### Example High-Confidence Alert Query

```spl
index=proxy_logs
| lookup threat_lookup domain OUTPUT threat_type confidence
| search confidence="high"
| stats count by src_ip, domain
| where count >= 1
```

---

## 7. SOC Investigation Workflow

When a detection is triggered:

1. Identify affected `src_ip`.
2. Map IP to user identity.
3. Review browsing history.
4. Validate endpoint telemetry.
5. Reset credentials if phishing-related.
6. Block malicious domain at proxy/firewall.
7. Add IOC to internal block list.
8. Document findings in case management system.

---

## 8. Security Concepts Demonstrated

This project demonstrates:

- Log ingestion and normalization
- Threat intelligence enrichment
- Indicator of Compromise (IOC) correlation
- Detection engineering logic
- Confidence-based prioritization
- Host-level risk analysis
- SOC triage workflow modeling
- Security dashboard development

---

## 9. Limitations

Current limitations:

- Static threat feed (manual CSV upload)
- Simulated proxy logs
- No automated alerting integration
- No endpoint detection correlation
- No user identity mapping
- No automated response actions

---

## 10. Future Improvements

Potential enhancements:

- Automated threat feed ingestion via API
- Host-based risk scoring model
- Integration with endpoint detection logs
- Automated alert generation
- Cloud log ingestion (AWS / Azure)
- MITRE ATT&CK technique mapping
- SOAR-based response automation

---

## Conclusion

This project demonstrates how raw proxy logs can be enriched with threat intelligence to generate actionable security detections. It models a realistic SOC workflow involving correlation, prioritization, and structured investigation logic.

The implementation reflects practical detection engineering principles rather than simple dashboard visualization, showcasing applied SOC and threat intelligence skills.
