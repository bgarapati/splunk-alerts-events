# Operational Alerts - SPL Queries

## System Maintenance

### Scheduled Maintenance Windows
```spl
index=maintenance sourcetype=maintenance:calendar
| stats count by system, maintenance_type, start_time, end_time, status
| where status="scheduled" AND start_time > now
| table _time, system, maintenance_type, start_time, end_time, status
```

### Patch Deployment Alerts
```spl
index=maintenance sourcetype=patch OR sourcetype=wsus
| stats count by system, patch_status, patch_version, deployment_time
| where patch_status="failed" OR patch_status="incomplete"
| table _time, system, patch_status, patch_version, deployment_time, count
| eval severity="warning"
```

### System Restart Notifications
```spl
index=maintenance sourcetype=system OR sourcetype=uptime
| stats count by system, restart_reason, uptime_before_restart
| where restart_reason IN (scheduled, kernel_update, maintenance)
| table _time, system, restart_reason, uptime_before_restart, count
```

### Service Restart Notifications
```spl
index=maintenance sourcetype=systemd OR sourcetype=supervisord
| stats count by service_name, restart_reason, host
| where restart_reason="manual" OR restart_reason="scheduled"
| table _time, service_name, restart_reason, host, count
```

### Configuration Changes
```spl
index=audit sourcetype=auditd OR sourcetype=configuration
| stats count by system, config_file, change_type, user
| where change_type="modified"
| table _time, system, config_file, change_type, user, count
```

## Capacity Planning

### CPU Trend Forecasting Alerts
```spl
index=os sourcetype=cpu
| timechart span=1d avg(pct_cpu) as avg_cpu by host
| trendline ema7(avg_cpu) as forecast
| where forecast > 85
| table _time, host, avg_cpu, forecast
| eval severity="warning"
```

### Memory Trend Forecasting
```spl
index=os sourcetype=mem
| timechart span=1d avg(pct_mem) as avg_mem by host
| trendline ema7(avg_mem) as forecast
| where forecast > 85
| table _time, host, avg_mem, forecast
| eval severity="warning"
```

### Disk Space Trend Forecasting
```spl
index=os sourcetype=df
| timechart span=1d avg(used_pct) as avg_used by host, mount_point
| trendline ema7(avg_used) as forecast
| where forecast > 85
| table _time, host, mount_point, avg_used, forecast
| eval severity="warning"
```

### Network Bandwidth Trend Forecasting
```spl
index=netflow sourcetype=netflow
| timechart span=1d sum(bytes) as total_bytes by interface
| eval bandwidth_pct=round((total_bytes / interface_capacity) * 100, 2)
| trendline ema7(bandwidth_pct) as forecast
| where forecast > 80
| table _time, interface, bandwidth_pct, forecast
```

### Storage Growth Alerts
```spl
index=storage sourcetype=netapp:ontap OR sourcetype=emc:vnx
| timechart span=1w avg(used_capacity) as avg_used by volume
| eval growth_rate=round((avg_used - avg_used_1w_ago) / avg_used_1w_ago * 100, 2)
| where growth_rate > 10
| table _time, volume, avg_used, growth_rate
```

### Resource Pool Exhaustion Warnings
```spl
index=vmware sourcetype=vmware:perf:cluster
| stats avg(cpu_usage) as avg_cpu, avg(mem_usage) as avg_mem by cluster_name, resource_pool
| where avg_cpu > 85 OR avg_mem > 85
| table _time, cluster_name, resource_pool, avg_cpu, avg_mem
| eval severity="warning"
```

## SLA & SLO Violations

### SLA Breach Warnings
```spl
index=sla sourcetype=sla:monitoring
| stats count by service, sla_name, sla_threshold, current_value, breach_status
| where breach_status="breached" OR current_value > sla_threshold
| table _time, service, sla_name, sla_threshold, current_value, breach_status
| eval severity="critical"
```

### SLO Threshold Violations
```spl
index=slo sourcetype=slo:monitoring
| stats count by service, slo_name, slo_target, actual_value, error_budget
| where actual_value < slo_target OR error_budget < 0
| table _time, service, slo_name, slo_target, actual_value, error_budget
| eval severity="warning"
```

### Uptime Percentage Drops
```spl
index=availability sourcetype=uptime:monitoring
| timechart span=1h avg(uptime_pct) as avg_uptime by service
| where avg_uptime < 99.9
| table _time, service, avg_uptime
| eval severity="critical"
```

### Response Time SLA Breaches
```spl
index=performance sourcetype=apm OR sourcetype=webperf
| stats avg(response_time) as avg_response by service, sla_threshold
| where avg_response > sla_threshold
| table _time, service, avg_response, sla_threshold
| eval severity="warning"
```

### Error Rate SLO Violations
```spl
index=application sourcetype=access_combined
| stats count as total, count(eval(status >= 500)) as errors by service
| eval error_rate=round((errors/total)*100, 2)
| where error_rate > 1
| table _time, service, error_rate
| eval severity="warning"
```

## Change Management

### Unauthorized Configuration Changes
```spl
index=audit sourcetype=auditd OR sourcetype=configuration
| stats count by system, config_file, user, change_approved
| where change_approved="false"
| table _time, system, config_file, user, change_approved, count
| eval severity="critical"
```

### Change Request Failures
```spl
index=changemanagement sourcetype=service_now OR sourcetype=jira
| stats count by change_request, change_status, error_message
| where change_status="failed" OR change_status="rejected"
| table _time, change_request, change_status, error_message, count
```

### Change Window Violations
```spl
index=changemanagement sourcetype=change_calendar
| stats count by change_request, scheduled_window, actual_window, violation_type
| where violation_type="outside_window"
| table _time, change_request, scheduled_window, actual_window, violation_type, count
```

### Rollback Required Events
```spl
index=deploy sourcetype=deployment OR sourcetype=rollback
| stats count by application, change_request, rollback_reason, rollback_status
| where rollback_status="required" OR rollback_status="executed"
| table _time, application, change_request, rollback_reason, rollback_status, count
| eval severity="critical"
```

### Configuration Drift Detection
```spl
index=configuration sourcetype=chef OR sourcetype=puppet OR sourcetype=ansible
| stats count by system, config_item, expected_value, actual_value
| where expected_value != actual_value
| table _time, system, config_item, expected_value, actual_value, count
| eval severity="warning"
```

## Incident Management

### Incident Creation
```spl
index=incident sourcetype=service_now OR sourcetype=jira
| stats count by incident_number, severity, category, assignee
| table _time, incident_number, severity, category, assignee, count
```

### Incident Escalation
```spl
index=incident sourcetype=service_now OR sourcetype=jira
| stats count by incident_number, current_severity, previous_severity, escalation_time
| where current_severity != previous_severity
| table _time, incident_number, current_severity, previous_severity, escalation_time, count
```

### Incident SLA Breaches
```spl
index=incident sourcetype=service_now OR sourcetype=jira
| stats count by incident_number, sla_type, sla_breach, breach_time
| where sla_breach="true"
| table _time, incident_number, sla_type, sla_breach, breach_time, count
| eval severity="critical"
```

### Major Incident Triggers
```spl
index=incident sourcetype=service_now OR sourcetype=jira
| stats count by incident_number, severity, impact, urgency
| where severity="critical" OR severity="major"
| table _time, incident_number, severity, impact, urgency, count
```

### Incident Resolution Delays
```spl
index=incident sourcetype=service_now OR sourcetype=jira
| stats count by incident_number, created_time, resolved_time, resolution_sla, delay_duration
| eval time_to_resolve=resolved_time-created_time
| where time_to_resolve > resolution_sla
| table _time, incident_number, time_to_resolve, resolution_sla, delay_duration
```

## Job & Batch Processing

### Scheduled Job Failures
```spl
index=job sourcetype=crontab OR sourcetype=scheduler
| stats count by job_name, job_type, status, execution_time
| where status="failed"
| table _time, job_name, job_type, status, execution_time, count
| eval severity="critical"
```

### Job Timeout Alerts
```spl
index=job sourcetype=crontab OR sourcetype=scheduler
| stats count by job_name, job_type, duration, timeout_threshold
| where duration > timeout_threshold
| table _time, job_name, job_type, duration, timeout_threshold, count
| eval severity="warning"
```

### Job Running Too Long
```spl
index=job sourcetype=crontab OR sourcetype=scheduler
| stats count by job_name, job_type, start_time, current_duration
| eval current_duration=now-start_time
| where current_duration > 3600
| table _time, job_name, job_type, start_time, current_duration, count
```

### Job Dependency Failures
```spl
index=job sourcetype=airflow OR sourcetype=jenkins
| stats count by job_name, dependency_job, dependency_status
| where dependency_status="failed"
| table _time, job_name, dependency_job, dependency_status, count
```

### Batch Processing Errors
```spl
index=batch sourcetype=batch:logs OR sourcetype=etl
| stats count by batch_name, batch_id, error_type, error_count
| where error_count > 0
| table _time, batch_name, batch_id, error_type, error_count
```

### ETL Job Failures
```spl
index=etl sourcetype=informatica OR sourcetype=talend
| stats count by etl_job, source, target, status, error_message
| where status="failed"
| table _time, etl_job, source, target, status, error_message, count
| eval severity="critical"
```

## Monitoring System Health

### Monitoring Agent Down
```spl
index=monitoring sourcetype=uf OR sourcetype=forwarder
| stats count by host, agent_status, last_heartbeat
| where agent_status="down" OR (now-last_heartbeat) > 300
| table _time, host, agent_status, last_heartbeat, count
| eval severity="critical"
```

### Alert Delivery Failures
```spl
index=alert sourcetype=alert:delivery OR sourcetype=email
| stats count by alert_name, delivery_status, recipient, error_message
| where delivery_status="failed"
| table _time, alert_name, delivery_status, recipient, error_message, count
```

### Metric Collection Gaps
```spl
index=metrics sourcetype=metrics
| stats count by metric_name, host, collection_status
| where collection_status="gap" OR count = 0
| table _time, metric_name, host, collection_status, count
```

### Log Shipping Failures
```spl
index=logs sourcetype=uf OR sourcetype=forwarder
| stats count by host, log_source, ship_status, error_count
| where ship_status="failed" OR error_count > 10
| table _time, host, log_source, ship_status, error_count, count
```

### Dashboard Data Gaps
```spl
index=dashboard sourcetype=dashboard:metrics
| stats count by dashboard_name, data_source, last_update, gap_duration
| eval gap_duration=now-last_update
| where gap_duration > 3600
| table _time, dashboard_name, data_source, last_update, gap_duration
```

## Service Desk & Support

### High Ticket Volume
```spl
index=servicedesk sourcetype=service_now OR sourcetype=jira
| timechart span=1h count as tickets by priority
| where tickets > 50
| table _time, priority, tickets
| eval severity="warning"
```

### Ticket SLA Breaches
```spl
index=servicedesk sourcetype=service_now OR sourcetype=jira
| stats count by ticket_number, sla_type, sla_breach, breach_time
| where sla_breach="true"
| table _time, ticket_number, sla_type, sla_breach, breach_time, count
```

### Ticket Escalation Required
```spl
index=servicedesk sourcetype=service_now OR sourcetype=jira
| stats count by ticket_number, current_level, escalation_level, escalation_reason
| where escalation_level > current_level
| table _time, ticket_number, current_level, escalation_level, escalation_reason, count
```

### Priority Ticket Backlog
```spl
index=servicedesk sourcetype=service_now OR sourcetype=jira
| stats count by priority, status, created_time
| where priority IN (critical, high) AND status="open"
| eval age_days=round((now-created_time)/86400, 2)
| where age_days > 7
| table _time, priority, status, age_days, count
```

## Compliance & Auditing

### Audit Log Failures
```spl
index=audit sourcetype=auditd OR sourcetype=windows:security
| stats count by system, audit_status, error_type
| where audit_status="failed"
| table _time, system, audit_status, error_type, count
| eval severity="critical"
```

### Compliance Policy Violations
```spl
index=compliance sourcetype=compliance:scanner
| stats count by policy_name, system, violation_type, severity
| where severity="critical" OR severity="high"
| table _time, policy_name, system, violation_type, severity, count
```

### Regulatory Reporting Due
```spl
index=compliance sourcetype=regulatory:reporting
| stats count by report_type, due_date, status, days_remaining
| eval days_remaining=round((due_date-now)/86400, 0)
| where days_remaining < 7 AND status="not_submitted"
| table _time, report_type, due_date, status, days_remaining, count
```

### Audit Trail Gaps
```spl
index=audit sourcetype=auditd
| stats count by system, audit_source, last_audit_time
| eval gap_duration=now-last_audit_time
| where gap_duration > 3600
| table _time, system, audit_source, last_audit_time, gap_duration
```

## Cost & Resource Optimization

### Cloud Cost Anomalies
```spl
index=cloud sourcetype=aws:cost OR sourcetype=azure:cost
| timechart span=1d sum(cost) as daily_cost by service
| trendline ema7(daily_cost) as forecast
| where daily_cost > (forecast * 1.5)
| table _time, service, daily_cost, forecast
| eval severity="warning"
```

### Resource Over-provisioning Alerts
```spl
index=cloud sourcetype=aws:cloudwatch OR sourcetype=azure:monitor
| stats avg(cpu_usage) as avg_cpu, avg(memory_usage) as avg_mem by instance_id, instance_type
| where avg_cpu < 20 AND avg_mem < 20
| table _time, instance_id, instance_type, avg_cpu, avg_mem
| eval recommendation="downsize"
```

### Idle Resource Detection
```spl
index=cloud sourcetype=aws:cloudwatch OR sourcetype=azure:monitor
| stats avg(cpu_usage) as avg_cpu, avg(network_in) as avg_net_in by resource_id, resource_type
| where avg_cpu < 5 AND avg_net_in < 1024
| table _time, resource_id, resource_type, avg_cpu, avg_net_in
| eval status="idle"
```

### Cost Budget Breaches
```spl
index=cost sourcetype=cost:monitoring
| stats sum(actual_cost) as total_cost, sum(budget) as total_budget by department, month
| where total_cost > total_budget
| table _time, department, month, total_cost, total_budget
| eval severity="warning"
```

### Unattached Storage Detection
```spl
index=cloud sourcetype=aws:ebs OR sourcetype=azure:disk
| stats count by volume_id, status, attached_instance
| where status="available" AND attached_instance="none"
| table _time, volume_id, status, attached_instance, count
| eval recommendation="delete_or_attach"
```
