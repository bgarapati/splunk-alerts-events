# Security Dashboards - SPL Queries

## 1. Security Operations Center (SOC) Dashboard

### Real-time Threat Overview
```spl
index=security
| stats count by threat_type, severity
| eval threat_level=if(severity="critical", "high", if(severity="high", "medium", "low"))
| table threat_type, severity, threat_level, count
| sort - severity, - count
```

### Active Incidents Count
```spl
index=incident sourcetype=siem
| stats count as total_incidents, count(eval(severity="critical")) as critical, count(eval(severity="high")) as high
| table total_incidents, critical, high
```

### Critical Security Alerts
```spl
index=security severity IN (critical, high)
| stats count by alert_name, severity, source
| table _time, alert_name, severity, source, count
| sort - severity, - _time
| head 20
```

### Threat Intelligence Feed
```spl
index=threat_intel
| stats count by threat_feed, threat_type, confidence
| table threat_feed, threat_type, confidence, count
| sort - confidence
```

### Blocked/Allowed Traffic Ratio
```spl
index=firewall
| stats count as total, count(eval(action="block")) as blocked, count(eval(action="allow")) as allowed
| eval block_ratio=round((blocked/total)*100, 2), allow_ratio=round((allowed/total)*100, 2)
| table total, blocked, allowed, block_ratio, allow_ratio
```

### Malware Detection Timeline
```spl
index=endpoint sourcetype=antivirus
| timechart span=1h count by threat_name
| fields - NULL
```

### Intrusion Detection Status
```spl
index=ids sourcetype=snort OR sourcetype=suricata
| stats count by alert_type, severity, sensor
| table alert_type, severity, sensor, count
| sort - severity
```

### Vulnerability Summary
```spl
index=vulnerability sourcetype=nessus OR sourcetype=qualys
| stats count as total, count(eval(severity="critical")) as critical, count(eval(severity="high")) as high by host
| table host, total, critical, high
| sort - critical
```

### Security Posture Score
```spl
index=security
| stats count as total_events, count(eval(severity="critical")) as critical, count(eval(severity="high")) as high
| eval risk_score=round(100 - ((critical*10 + high*5) / total_events * 100), 2)
| table total_events, critical, high, risk_score
```

## 2. Authentication & Access Dashboard

### Failed Login Attempts Over Time
```spl
index=authentication action="failure"
| timechart span=1h count by src_ip
| fields - NULL
```

### Successful vs Failed Logins
```spl
index=authentication
| stats count as total, count(eval(action="success")) as success, count(eval(action="failure")) as failure
| eval success_pct=round((success/total)*100, 2), failure_pct=round((failure/total)*100, 2)
| table total, success, failure, success_pct, failure_pct
```

### Geographic Login Distribution
```spl
index=authentication action="success"
| stats count by src_country
| geostats count
```

### Unusual Login Locations
```spl
index=authentication action="success"
| stats count by user, src_country, src_city
| lookup geo_location_table user as user OUTPUT known_locations
| where NOT src_country IN (known_locations)
| table _time, user, src_country, src_city, count
```

### Privileged Account Activity
```spl
index=authentication user IN (admin, administrator, root, sudo)
| stats count by user, action, src_ip, dest
| table _time, user, action, src_ip, dest, count
| sort - _time
```

### MFA Success/Failure Rates
```spl
index=authentication action IN (mfa_success, mfa_failure)
| stats count as total, count(eval(action="mfa_success")) as success, count(eval(action="mfa_failure")) as failure
| eval success_rate=round((success/total)*100, 2), failure_rate=round((failure/total)*100, 2)
| table total, success, failure, success_rate, failure_rate
```

### Account Lockout Trends
```spl
index=authentication action="lockout"
| timechart span=1d count by user
| fields - NULL
```

### Authentication Method Breakdown
```spl
index=authentication
| stats count by auth_method, status
| table auth_method, status, count
| chart count over auth_method by status
```

### Active Sessions Overview
```spl
index=authentication action="session_start"
| stats count as active_sessions by user, src_ip, application
| table user, src_ip, application, active_sessions
| sort - active_sessions
```

### Suspicious User Behavior Indicators
```spl
index=authentication action="success"
| stats count by user, src_ip, hour
| eval baseline_avg=avg(count), current_count=count
| where current_count > (baseline_avg * 3)
| table _time, user, src_ip, baseline_avg, current_count, anomaly_score
```

## 3. Network Security Dashboard

### Traffic Volume by Protocol
```spl
index=network sourcetype=netflow
| stats sum(bytes) as total_bytes by protocol
| eval total_mb=round(total_bytes/1048576, 2)
| table protocol, total_bytes, total_mb
| sort - total_bytes
```

### Blocked Connection Attempts
```spl
index=firewall action="block"
| stats count by src_ip, dest_ip, dest_port, rule_name
| table _time, src_ip, dest_ip, dest_port, rule_name, count
| sort - count
| head 20
```

### Port Scanning Activity
```spl
index=network
| stats dc(dest_port) as unique_ports, values(dest_port) as ports by src_ip, _time
| bin _time span=5m
| where unique_ports > 20
| table _time, src_ip, unique_ports, ports
```

### DDoS Attack Indicators
```spl
index=network
| timechart span=1m count by src_ip
| eventstats sum(count) as total_requests
| where total_requests > 10000
| table _time, src_ip, total_requests, attack_severity
```

### Firewall Rule Violations
```spl
index=firewall action="deny"
| stats count by src_ip, dest_ip, dest_port, rule_name
| table _time, src_ip, dest_ip, dest_port, rule_name, count
| sort - count
```

### Intrusion Detection Alerts
```spl
index=ids sourcetype=snort OR sourcetype=suricata
| stats count by alert_type, severity, src_ip, dest_ip
| table _time, alert_type, severity, src_ip, dest_ip, count
| sort - severity
```

### Top Source/Destination IPs
```spl
index=network sourcetype=netflow
| stats sum(bytes) as total_bytes by src_ip, dest_ip
| eval total_mb=round(total_bytes/1048576, 2)
| table src_ip, dest_ip, total_mb
| sort - total_mb
| head 20
```

### Network Anomaly Timeline
```spl
index=network
| timechart span=1h avg(bytes) as avg_bytes, max(bytes) as max_bytes by protocol
| fields - NULL
```

### VPN Connection Status
```spl
index=vpn sourcetype=cisco_anyconnect OR sourcetype=openvpn
| stats count by user, vpn_server, connection_status
| table user, vpn_server, connection_status, count
```

### DNS Query Analysis
```spl
index=dns
| stats count by query_type, response_code, domain
| table query_type, response_code, domain, count
| sort - count
| head 20
```

## 4. Endpoint Security Dashboard

### Endpoint Health Status
```spl
index=endpoint sourcetype=antivirus OR sourcetype=edr
| stats count by host, health_status, last_scan
| table host, health_status, last_scan, count
```

### Antivirus Detection Trends
```spl
index=endpoint sourcetype=antivirus
| timechart span=1d count by threat_type
| fields - NULL
```

### EDR Alert Summary
```spl
index=endpoint sourcetype=edr
| stats count by host, alert_type, severity
| table host, alert_type, severity, count
| sort - severity
```

### Software Inventory
```spl
index=endpoint sourcetype=inventory
| stats count by host, software_name, version, vendor
| table host, software_name, version, vendor, count
```

### Patch Compliance Status
```spl
index=endpoint sourcetype=patch
| stats count as total, count(eval(patch_status="installed")) as installed, count(eval(patch_status="missing")) as missing by host
| eval compliance_pct=round((installed/total)*100, 2)
| table host, total, installed, missing, compliance_pct
| sort compliance_pct
```

### Vulnerable Endpoints
```spl
index=endpoint sourcetype=vulnerability
| stats count by host, vulnerability_severity, cve_id
| where vulnerability_severity IN (critical, high)
| table host, vulnerability_severity, cve_id, count
| sort - vulnerability_severity
```

### Process Execution Anomalies
```spl
index=endpoint sourcetype=Sysmon:*
| stats count by host, process_name, CommandLine, user
| where process_name IN (*powershell.exe*, *cmd.exe*, *wscript.exe*)
| table host, process_name, CommandLine, user, count
```

### File Integrity Monitoring
```spl
index=endpoint sourcetype=fim
| stats count by host, file_path, change_type, user
| table _time, host, file_path, change_type, user, count
| sort - _time
```

### USB Device Activity
```spl
index=endpoint sourcetype=usb
| stats count by host, device_type, action, user
| table _time, host, device_type, action, user, count
```

### Endpoint Risk Score
```spl
index=endpoint
| stats count as total_events, count(eval(severity="critical")) as critical, count(eval(severity="high")) as high by host
| eval risk_score=round(100 - ((critical*10 + high*5) / total_events * 100), 2)
| table host, total_events, critical, high, risk_score
| sort risk_score
```

## 5. Cloud Security Dashboard

### Cloud Account Security Posture
```spl
index=aws sourcetype=aws:cloudtrail OR sourcetype=aws:config
| stats count by account_id, service, security_status
| table account_id, service, security_status, count
```

### IAM Privilege Escalation Attempts
```spl
index=aws sourcetype=aws:cloudtrail
| stats count by userIdentity.userName, eventName, sourceIPAddress
| where eventName IN (AttachUserPolicy, AttachRolePolicy, PutUserPolicy)
| table _time, userIdentity.userName, eventName, sourceIPAddress, count
```

### CloudTrail/Azure Monitor Events
```spl
index=aws sourcetype=aws:cloudtrail
| timechart span=1h count by eventName
| fields - NULL
```

### S3/Azure Storage Access
```spl
index=aws sourcetype=aws:s3:accesslogs
| stats count by bucket, operation, requester, http_status
| table bucket, operation, requester, http_status, count
```

### Unusual API Usage
```spl
index=aws sourcetype=aws:cloudtrail
| stats count by userIdentity.userName, eventName, sourceIPAddress
| eval api_usage=count
| where api_usage > 1000
| table _time, userIdentity.userName, eventName, sourceIPAddress, api_usage
```

### Container Security Status
```spl
index=kubernetes sourcetype=kube:objects
| stats count by pod_name, namespace, security_context, image
| table pod_name, namespace, security_context, image, count
```

### Kubernetes Pod Security
```spl
index=kubernetes sourcetype=kube:objects
| stats count by namespace, pod_name, security_policy, violation_type
| where violation_type="security_violation"
| table namespace, pod_name, security_policy, violation_type, count
```

### Cloud Misconfigurations
```spl
index=aws sourcetype=aws:config OR sourcetype=aws:security_hub
| stats count by resource_type, config_rule, compliance_status
| where compliance_status="NON_COMPLIANT"
| table resource_type, config_rule, compliance_status, count
```

### Cost Anomaly Detection
```spl
index=aws sourcetype=aws:cost
| timechart span=1d sum(cost) as daily_cost by service
| trendline ema7(daily_cost) as forecast
| table _time, service, daily_cost, forecast
```

### Compliance Status
```spl
index=aws sourcetype=aws:security_hub OR sourcetype=aws:config
| stats count as total, count(eval(compliance_status="COMPLIANT")) as compliant, count(eval(compliance_status="NON_COMPLIANT")) as non_compliant by standard
| eval compliance_pct=round((compliant/total)*100, 2)
| table standard, total, compliant, non_compliant, compliance_pct
```

## 6. Data Protection Dashboard

### Data Access Patterns
```spl
index=access
| stats count by user, data_type, access_type, file_path
| table user, data_type, access_type, file_path, count
| sort - count
```

### Sensitive Data Access Logs
```spl
index=access data_class="sensitive"
| stats count by user, data_type, file_path, access_time
| table _time, user, data_type, file_path, access_time, count
```

### Data Exfiltration Attempts
```spl
index=network (dest_port IN (21, 22, 873) OR protocol="ftp" OR protocol="ssh")
| stats sum(bytes) as total_bytes by src_ip, dest_ip, protocol
| where total_bytes > 104857600
| eval data_size_mb=round(total_bytes/1048576, 2)
| table _time, src_ip, dest_ip, protocol, data_size_mb
```

### DLP (Data Loss Prevention) Alerts
```spl
index=dlp sourcetype=dlp:alerts
| stats count by policy_name, severity, user, file_type
| table _time, policy_name, severity, user, file_type, count
| sort - severity
```

### File Access Violations
```spl
index=access
| stats count by user, file_path, access_type, permission
| where access_type="denied"
| table _time, user, file_path, access_type, permission, count
```

### Database Query Monitoring
```spl
index=database
| stats count by user, database, table, query_type
| table user, database, table, query_type, count
| sort - count
```

### Encryption Status
```spl
index=encryption sourcetype=encryption:status
| stats count by resource_type, encryption_status, algorithm
| table resource_type, encryption_status, algorithm, count
```

### Data Classification Summary
```spl
index=access
| stats count by data_class, resource_type, sensitivity_level
| table data_class, resource_type, sensitivity_level, count
```

### PII/PHI Access Tracking
```spl
index=access data_class IN (pii, phi)
| stats count by user, data_class, access_type, file_path
| table _time, user, data_class, access_type, file_path, count
```

### Data Retention Compliance
```spl
index=audit sourcetype=data_retention
| stats count by data_type, retention_policy, compliance_status, days_retained
| table data_type, retention_policy, compliance_status, days_retained
```

## 7. Threat Intelligence Dashboard

### Known Malicious IPs
```spl
index=threat_intel
| stats count by ip_address, threat_type, confidence, source_feed
| table ip_address, threat_type, confidence, source_feed, count
| sort - confidence
```

### Threat Feed Indicators
```spl
index=threat_intel
| stats count by feed_name, indicator_type, last_seen
| table feed_name, indicator_type, last_seen, count
```

### IOC (Indicators of Compromise) Matches
```spl
index=security
| lookup threat_intel_iocs ip as src_ip OUTPUT is_ioc, ioc_type
| where is_ioc="true"
| stats count by src_ip, ioc_type, match_time
| table _time, src_ip, ioc_type, match_time, count
```

### Attack Surface Analysis
```spl
index=network
| stats dc(dest_port) as open_ports, values(dest_port) as ports by dest_ip
| table dest_ip, open_ports, ports
| sort - open_ports
```

### Phishing Campaign Tracking
```spl
index=email sourcetype=email:security
| stats count by campaign_id, sender_domain, recipient_domain, threat_score
| table campaign_id, sender_domain, recipient_domain, threat_score, count
| sort - threat_score
```

### Malware Hash Detections
```spl
index=endpoint sourcetype=antivirus
| stats count by file_hash, threat_name, detection_time, host
| table file_hash, threat_name, detection_time, host, count
```

### C2 (Command & Control) Communications
```spl
index=network
| lookup c2_domains domain as dest OUTPUT is_c2, c2_family
| where is_c2="true"
| stats count by src_ip, dest, c2_family, _time
| table _time, src_ip, dest, c2_family, count
```

### Dark Web Monitoring
```spl
index=threat_intel source="dark_web"
| stats count by keyword, forum, post_date, threat_level
| table keyword, forum, post_date, threat_level, count
```

### Vulnerability Exploit Tracking
```spl
index=vulnerability sourcetype=exploitdb
| stats count by cve_id, exploit_available, exploit_type, cvss_score
| table cve_id, exploit_available, exploit_type, cvss_score, count
| sort - cvss_score
```

### Threat Actor Attribution
```spl
index=threat_intel
| stats count by threat_actor, tactics, techniques, country
| table threat_actor, tactics, techniques, country, count
```

## 8. Compliance Dashboard

### PCI DSS Compliance Status
```spl
index=compliance standard="PCI_DSS"
| stats count as total, count(eval(status="compliant")) as compliant, count(eval(status="non_compliant")) as non_compliant by control_id
| eval compliance_pct=round((compliant/total)*100, 2)
| table control_id, total, compliant, non_compliant, compliance_pct
```

### HIPAA Compliance Metrics
```spl
index=compliance standard="HIPAA"
| stats count by requirement, status, last_audit
| table requirement, status, last_audit, count
```

### GDPR Compliance Tracking
```spl
index=compliance standard="GDPR"
| stats count by article, data_subject, processing_activity, compliance_status
| table article, data_subject, processing_activity, compliance_status, count
```

### SOX Control Status
```spl
index=compliance standard="SOX"
| stats count by control_id, control_type, test_status, last_test_date
| table control_id, control_type, test_status, last_test_date, count
```

### Industry-specific Compliance
```spl
index=compliance
| stats count by standard, industry, compliance_status, last_assessment
| table standard, industry, compliance_status, last_assessment, count
```

### Policy Violation Summary
```spl
index=audit
| stats count by policy_name, violation_type, severity, user
| table policy_name, violation_type, severity, user, count
| sort - severity
```

### Audit Trail Completeness
```spl
index=audit
| stats count by system, audit_source, log_retention_days, coverage_pct
| table system, audit_source, log_retention_days, coverage_pct
```

### Regulatory Reporting Status
```spl
index=compliance
| stats count by report_type, due_date, submission_status, submitted_date
| table report_type, due_date, submission_status, submitted_date, count
```

### Audit Finding Status
```spl
index=compliance
| stats count by finding_id, severity, status, remediation_due_date
| table finding_id, severity, status, remediation_due_date, count
```

### Risk Assessment Overview
```spl
index=risk
| stats count by risk_category, risk_level, likelihood, impact
| table risk_category, risk_level, likelihood, impact, count
| sort - risk_level
```

## 9. Incident Response Dashboard

### Active Incidents Overview
```spl
index=incident sourcetype=siem
| stats count by incident_id, severity, status, assignee
| where status="open" OR status="in_progress"
| table incident_id, severity, status, assignee, count
| sort - severity
```

### Incident Severity Distribution
```spl
index=incident sourcetype=siem
| stats count by severity, status
| chart count over severity by status
```

### MTTR (Mean Time to Respond)
```spl
index=incident sourcetype=siem
| stats avg(response_time) as avg_response_time, max(response_time) as max_response_time by severity
| table severity, avg_response_time, max_response_time
```

### MTTF (Mean Time to Fix)
```spl
index=incident sourcetype=siem
| stats avg(resolution_time) as avg_resolution_time, max(resolution_time) as max_resolution_time by severity
| table severity, avg_resolution_time, max_resolution_time
```

### Incident Trends Over Time
```spl
index=incident sourcetype=siem
| timechart span=1d count by severity
| fields - NULL
```

### Response Team Workload
```spl
index=incident sourcetype=siem
| stats count as total_incidents, count(eval(status="open")) as open, count(eval(status="in_progress")) as in_progress by assignee
| table assignee, total_incidents, open, in_progress
```

### Escalation Status
```spl
index=incident sourcetype=siem
| stats count by incident_id, current_severity, escalation_level, escalation_time
| where escalation_level > 1
| table incident_id, current_severity, escalation_level, escalation_time, count
```

### Root Cause Analysis Progress
```spl
index=incident sourcetype=siem
| stats count by incident_id, rca_status, rca_completion_pct
| where rca_status IN (in_progress, not_started)
| table incident_id, rca_status, rca_completion_pct, count
```

### Post-incident Review Status
```spl
index=incident sourcetype=siem
| stats count by incident_id, pir_status, pir_due_date
| where pir_status="not_completed"
| table incident_id, pir_status, pir_due_date, count
```

### Incident Hotspots
```spl
index=incident sourcetype=siem
| stats count by source_system, incident_type, severity
| table source_system, incident_type, severity, count
| sort - count
```

## 10. Vulnerability Management Dashboard

### Critical Vulnerabilities Count
```spl
index=vulnerability sourcetype=nessus OR sourcetype=qualys
| stats count by host, severity, cve_id
| where severity="critical"
| table host, severity, cve_id, count
| sort - count
```

### Vulnerability Trends Over Time
```spl
index=vulnerability sourcetype=nessus OR sourcetype=qualys
| timechart span=1w count by severity
| fields - NULL
```

### Patch Compliance Status
```spl
index=vulnerability sourcetype=patch
| stats count as total, count(eval(patch_status="installed")) as installed, count(eval(patch_status="missing")) as missing by host
| eval compliance_pct=round((installed/total)*100, 2)
| table host, total, installed, missing, compliance_pct
| sort compliance_pct
```

### Vulnerability Risk Score
```spl
index=vulnerability sourcetype=nessus OR sourcetype=qualys
| stats count by host, severity, cvss_score
| eval risk_score=round(cvss_score * severity_weight, 2)
| table host, severity, cvss_score, risk_score, count
| sort - risk_score
```

### CVE (Common Vulnerabilities) Tracking
```spl
index=vulnerability sourcetype=nessus OR sourcetype=qualys
| stats count by cve_id, severity, cvss_score, publish_date
| table cve_id, severity, cvss_score, publish_date, count
| sort - cvss_score
```

### Exploitability Assessment
```spl
index=vulnerability sourcetype=exploitdb
| stats count by cve_id, exploit_available, exploit_type, platform
| table cve_id, exploit_available, exploit_type, platform, count
```

### Asset Vulnerability Mapping
```spl
index=vulnerability sourcetype=nessus OR sourcetype=qualys
| stats count by asset_type, host, severity, vulnerability_count
| table asset_type, host, severity, vulnerability_count, count
```

### Remediation Progress
```spl
index=vulnerability sourcetype=remediation
| stats count as total, count(eval(status="remediated")) as remediated, count(eval(status="pending")) as pending by host
| eval remediation_pct=round((remediated/total)*100, 2)
| table host, total, remediated, pending, remediation_pct
```

### Vulnerability Aging
```spl
index=vulnerability sourcetype=nessus OR sourcetype=qualys
| stats count by cve_id, severity, days_since_detection
| eval age_category=if(days_since_detection < 30, "new", if(days_since_detection < 90, "aging", "old"))
| table cve_id, severity, days_since_detection, age_category, count
```

### Third-party Component Risks
```spl
index=vulnerability sourcetype=sbom OR sourcetype=dependency_check
| stats count by component_name, component_version, severity, cve_id
| table component_name, component_version, severity, cve_id, count
| sort - severity
```
