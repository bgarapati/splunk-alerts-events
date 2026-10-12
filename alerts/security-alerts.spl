# Security Alerts - SPL Queries

## Authentication & Access Control

### Failed Login Attempts - Multiple Failed Logins from Same IP
```spl
index=authentication sourcetype=linux_secure OR sourcetype=WinEventLog:Security OR sourcetype=aws_cloudtrail
(action="failure" OR EventCode=4625 OR errorCode="AccessDenied")
| stats count by src_ip, user
| where count > 5
| sort - count
```

### Brute Force Attack Detection
```spl
index=authentication (action="failure" OR EventCode=4625)
| bin _time span=5m
| stats count as failed_attempts by _time, src_ip, user
| where failed_attempts > 10
| eval risk_score = min(100, failed_attempts * 10)
| sort - failed_attempts
```

### Account Lockout Events
```spl
index=authentication (EventCode=4740 OR message="account locked" OR action="lockout")
| stats count by user, src_ip, _time
| table _time, user, src_ip, count
| sort - _time
```

### Privileged Account Usage
```spl
index=authentication (user IN (admin, administrator, root, sudo) OR EventCode=4672 OR action="privilege_use")
| stats count by user, action, src_ip, dest
| table _time, user, action, src_ip, dest
| sort - _time
```

### Suspicious Login Activity - Unusual Time/Location
```spl
index=authentication action="success"
| eval hour=strftime(_time, "%H")
| where hour < 6 OR hour > 22
| stats count by user, src_ip, hour, src_country
| where count > 0
| eval severity="high"
```

### MFA Failures
```spl
index=authentication (action="mfa_failure" OR EventCode=6274 OR message="MFA failed")
| stats count by user, src_ip, _time
| where count > 3
| table _time, user, src_ip, count
| sort - _time
```

## Malware & Threat Detection

### Antivirus/Malware Detection
```spl
index=endpoint sourcetype=Symantec:* OR sourcetype=McAfee:* OR sourcetype=Windows:*
(virus_name=* OR threat=* OR malware=* OR action="quarantine" OR action="blocked")
| stats count by host, threat_name, action, _time
| table _time, host, threat_name, action, count
| sort - _time
```

### Known Malicious IP Connections
```spl
index=network
| lookup threat_intel_malicious_ips ip as src_ip OUTPUT is_malicious, threat_type
| where is_malicious="true"
| stats count by src_ip, dest_ip, threat_type, _time
| table _time, src_ip, dest_ip, threat_type, count
| sort - _time
```

### Command and Control (C2) Communications
```spl
index=network (dest_port IN (443, 8080, 9999) OR outbound_connection)
| lookup c2_domains domain as dest OUTPUT is_c2, c2_family
| where is_c2="true"
| stats count by src_ip, dest, c2_family, _time
| table _time, src_ip, dest, c2_family, count
```

### DNS Tunneling Detection
```spl
index=dns
| eval query_length=len(query)
| stats avg(query_length) as avg_len, max(query_length) as max_len, count by src_ip, domain
| where avg_len > 50 OR max_len > 100
| eval risk_level=if(avg_len > 80, "critical", "high")
```

### Suspicious Process Execution
```spl
index=endpoint sourcetype=Sysmon:* (process_name IN (*powershell.exe*, *cmd.exe*, *wscript.exe*) OR CommandLine=*EncodedCommand*)
| stats count by host, process_name, CommandLine, user
| where count > 0
| eval severity="high"
```

## Network Security

### Port Scanning Detection
```spl
index=network
| stats dc(dest_port) as unique_ports, values(dest_port) as ports by src_ip, _time
| bin _time span=1m
| where unique_ports > 20
| eval scan_type=if(unique_ports > 100, "aggressive", "normal")
| table _time, src_ip, unique_ports, scan_type
```

### SQL Injection Attempts
```spl
index=web uri="* OR ' OR 1=1 --*"
| stats count by src_ip, uri, method, user_agent
| where count > 0
| eval attack_type="SQL Injection"
| table _time, src_ip, uri, method, user_agent, attack_type
```

### XSS (Cross-site Scripting) Attacks
```spl
index=web uri="*<script* OR *javascript:* OR *onerror=*"
| stats count by src_ip, uri, method
| where count > 0
| eval attack_type="XSS"
| table _time, src_ip, uri, method, attack_type
```

### DDoS Attack Indicators
```spl
index=network
| timechart span=1m count by src_ip
| eventstats sum(count) as total_requests
| where total_requests > 10000
| eval attack_severity=if(total_requests > 50000, "critical", "high")
```

### Firewall Rule Violations
```spl
index=firewall action="deny"
| stats count by src_ip, dest_ip, dest_port, rule_name
| where count > 10
| table _time, src_ip, dest_ip, dest_port, rule_name, count
| sort - count
```

## Data Protection

### Unauthorized Access to Sensitive Data
```spl
index=access (file_path IN (*confidential*, *secret*, *password*, *credit_card*) OR data_class="sensitive")
| stats count by user, file_path, action, src_ip
| where action="read" OR action="download"
| table _time, user, file_path, action, src_ip, count
```

### Data Exfiltration Attempts
```spl
index=network (dest_port IN (21, 22, 873) OR protocol="ftp" OR protocol="ssh")
| stats sum(bytes) as total_bytes by src_ip, dest_ip, protocol
| where total_bytes > 104857600
| eval data_size_mb=round(total_bytes/1048576, 2)
| table _time, src_ip, dest_ip, protocol, data_size_mb
```

### PII/PHI Access Without Authorization
```spl
index=database (query IN (*SELECT*ssn*, *SELECT*credit_card*, *SELECT*phi*))
| stats count by user, database, table, query
| where user NOT IN (authorized_user_list)
| table _time, user, database, table, query, count
```

## Insider Threats

### Privilege Escalation Attempts
```spl
index=authentication (EventCode=4672 OR action="admin_grant" OR action="sudo")
| stats count by user, src_ip, privilege_granted
| where count > 0
| eval risk_level="high"
| table _time, user, src_ip, privilege_granted, count
```

### Unusual User Behavior Patterns
```spl
index=authentication action="success"
| stats count by user, src_ip, dest, hour
| eval baseline_avg=avg(count), current_count=count
| where current_count > (baseline_avg * 3)
| eval anomaly="high_activity"
| table _time, user, src_ip, dest, baseline_avg, current_count, anomaly
```

### Bulk Data Downloads
```spl
index=network (bytes > 10485760)
| stats sum(bytes) as total_bytes, count by src_ip, user
| where total_bytes > 1073741824
| eval data_size_gb=round(total_bytes/1073741824, 2)
| table _time, src_ip, user, data_size_gb, count
```

## Cloud Security

### AWS CloudTrail Anomalies
```spl
index=aws sourcetype=aws:cloudtrail
| stats count by userIdentity.userName, eventName, sourceIPAddress
| where eventName IN (ConsoleLogin, GetInstanceProfile, CreateAccessKey, DeleteUser)
| table _time, userIdentity.userName, eventName, sourceIPAddress, count
```

### Cloud Storage Public Access
```spl
index=aws sourcetype=aws:s3:accesslogs
| stats count by bucket, operation, requester
| where operation IN (REST.GET.OBJECT, REST.PUT.OBJECT) AND requester="anonymous"
| eval security_risk="public_access"
| table _time, bucket, operation, requester, security_risk
```

### Container Security Events
```spl
index=kubernetes
| stats count by pod_name, namespace, event_type, reason
| where event_type="Warning" OR event_type="Failed"
| table _time, pod_name, namespace, event_type, reason, count
```

## Endpoint Security

### Suspicious Registry Changes
```spl
index=endpoint sourcetype=Sysmon:* EventCode=13 OR EventCode=14
| stats count by host, TargetObject, Details
| where TargetObject IN (*Run*, *RunOnce*, *Startup*)
| eval risk_level="high"
| table _time, host, TargetObject, Details, count
```

### Unauthorized Software Installation
```spl
index=endpoint sourcetype=WinEventLog:Microsoft-Windows-AppLocker/*
| stats count by host, FilePath, Publisher
| where action="Blocked"
| table _time, host, FilePath, Publisher, count
```

## Application Security

### Web Application Firewall (WAF) Alerts
```spl
index=web sourcetype=aws:loadbalancer OR sourcetype=f5:bigip:asm
| stats count by src_ip, dest_ip, rule_name, violation_type
| where action="blocked"
| table _time, src_ip, dest_ip, rule_name, violation_type, count
```

### API Abuse Detection
```spl
index=api
| stats count by api_key, endpoint, method
| where count > 1000
| eval abuse_type="rate_limit_exceeded"
| table _time, api_key, endpoint, method, count, abuse_type
```

### Input Validation Failures
```spl
index=web (status=400 OR status=422)
| stats count by endpoint, error_message, src_ip
| where count > 50
| table _time, endpoint, error_message, src_ip, count
```
