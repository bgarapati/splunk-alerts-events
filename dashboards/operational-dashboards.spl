# Operational Dashboards - SPL Queries

## 1. Operations Overview Dashboard

### Overall Operational Health
```spl
index=operations
| stats count as total, count(eval(status="healthy")) as healthy, count(eval(status="unhealthy")) as unhealthy by category
| eval health_pct=round((healthy/total)*100, 2)
| table category, total, healthy, unhealthy, health_pct
```

### Active Incidents Count
```spl
index=incident sourcetype=operations
| stats count as total, count(eval(status="open")) as open, count(eval(status="in_progress")) as in_progress
| table total, open, in_progress
```

### Critical Alerts Summary
```spl
index=operations severity IN (critical, high)
| stats count by category, severity, alert_name
| table category, severity, alert_name, count
| sort - severity
```

### SLA Compliance Status
```spl
index=sla sourcetype=operations
| stats count as total, count(eval(status="compliant")) as compliant, count(eval(status="non_compliant")) as non_compliant by service
| eval compliance_pct=round((compliant/total)*100, 2)
| table service, total, compliant, non_compliant, compliance_pct
```

### Team Workload Distribution
```spl
index=incident sourcetype=operations
| stats count as total, count(eval(status="open")) as open, count(eval(status="in_progress")) as in_progress by team, assignee
| table team, assignee, total, open, in_progress
```

### Change Activity Volume
```spl
index=changemanagement sourcetype=operations
| timechart span=1d count as changes by change_type
| fields - NULL
```

### Maintenance Schedule
```spl
index=maintenance sourcetype=maintenance:calendar
| stats count by system, maintenance_type, start_time, end_time, status
| where start_time > now AND start_time < now+604800
| table system, maintenance_type, start_time, end_time, status, count
```

### Capacity Utilization
```spl
index=capacity sourcetype=capacity_planning
| stats avg(utilization) as avg_util by resource_type, resource_name
| table resource_type, resource_name, avg_util
| sort - avg_util
```

### Risk Summary
```spl
index=risk sourcetype=operations
| stats count by risk_category, risk_level, likelihood, impact
| table risk_category, risk_level, likelihood, impact, count
| sort - risk_level
```

### Upcoming Deadlines
```spl
index=operations sourcetype=tasks
| stats count by task_name, due_date, assignee, status
| where due_date > now AND due_date < now+604800 AND status!="completed"
| table task_name, due_date, assignee, status, count
```

## 2. Incident Management Dashboard

### Active Incidents Overview
```spl
index=incident sourcetype=service_now OR sourcetype=jira
| stats count by incident_id, severity, status, assignee
| where status="open" OR status="in_progress"
| table incident_id, severity, status, assignee, count
| sort - severity
```

### Incident Severity Distribution
```spl
index=incident sourcetype=service_now OR sourcetype=jira
| stats count by severity, status
| chart count over severity by status
```

### Incident Trends Over Time
```spl
index=incident sourcetype=service_now OR sourcetype=jira
| timechart span=1d count by severity
| fields - NULL
```

### MTTR (Mean Time to Respond)
```spl
index=incident sourcetype=service_now OR sourcetype=jira
| stats avg(response_time) as avg_response, max(response_time) as max_response by severity
| table severity, avg_response, max_response
```

### MTTF (Mean Time to Fix)
```spl
index=incident sourcetype=service_now OR sourcetype=jira
| stats avg(resolution_time) as avg_resolution, max(resolution_time) as max_resolution by severity
| table severity, avg_resolution, max_resolution
```

### Team Assignment Status
```spl
index=incident sourcetype=service_now OR sourcetype=jira
| stats count as total, count(eval(status="open")) as open, count(eval(status="in_progress")) as in_progress by team, assignee
| table team, assignee, total, open, in_progress
```

### Escalation Triggers
```spl
index=incident sourcetype=service_now OR sourcetype=jira
| stats count by incident_id, current_severity, escalation_level, escalation_time
| where escalation_level > 1
| table incident_id, current_severity, escalation_level, escalation_time, count
```

### Resolution Time Trends
```spl
index=incident sourcetype=service_now OR sourcetype=jira
| timechart span=1w avg(resolution_time) as avg_resolution by severity
| fields - NULL
```

### Incident Hotspots
```spl
index=incident sourcetype=service_now OR sourcetype=jira
| stats count by source_system, incident_type, severity
| table source_system, incident_type, severity, count
| sort - count
```

### Post-incident Review Status
```spl
index=incident sourcetype=service_now OR sourcetype=jira
| stats count by incident_id, pir_status, pir_due_date
| where pir_status="not_completed"
| table incident_id, pir_status, pir_due_date, count
```

## 3. Change Management Dashboard

### Pending Change Requests
```spl
index=changemanagement sourcetype=service_now OR sourcetype=jira
| stats count by change_request, change_type, priority, requested_by
| where status="pending" OR status="approved"
| table change_request, change_type, priority, requested_by, count
```

### Change Approval Status
```spl
index=changemanagement sourcetype=service_now OR sourcetype=jira
| stats count by change_request, change_type, approval_status, reviewer
| table change_request, change_type, approval_status, reviewer, count
```

### Change Implementation Calendar
```spl
index=changemanagement sourcetype=service_now OR sourcetype=jira
| stats count by change_request, scheduled_start, scheduled_end, status
| where scheduled_start > now AND scheduled_start < now+604800
| table change_request, scheduled_start, scheduled_end, status, count
```

### Change Success Rate
```spl
index=changemanagement sourcetype=service_now OR sourcetype=jira
| stats count as total, count(eval(status="completed")) as completed, count(eval(status="failed")) as failed by change_type
| eval success_rate=round((completed/total)*100, 2)
| table change_type, total, completed, failed, success_rate
```

### Rollback Events
```spl
index=changemanagement sourcetype=service_now OR sourcetype=jira
| stats count by change_request, rollback_reason, rollback_status, rollback_time
| table change_request, rollback_reason, rollback_status, rollback_time, count
```

### Change Window Violations
```spl
index=changemanagement sourcetype=service_now OR sourcetype=jira
| stats count by change_request, scheduled_window, actual_window, violation_type
| where violation_type="outside_window"
| table change_request, scheduled_window, actual_window, violation_type, count
```

### Configuration Drift Status
```spl
index=configuration sourcetype=chef OR sourcetype=puppet OR sourcetype=ansible
| stats count by system, config_item, expected_value, actual_value, drift_status
| where drift_status="drifted"
| table system, config_item, expected_value, actual_value, drift_status, count
```

### Impact Analysis
```spl
index=changemanagement sourcetype=service_now OR sourcetype=jira
| stats count by change_request, affected_systems, impact_level, risk_assessment
| table change_request, affected_systems, impact_level, risk_assessment, count
```

### Change Risk Assessment
```spl
index=changemanagement sourcetype=service_now OR sourcetype=jira
| stats count by change_request, risk_level, risk_factors, mitigation_plan
| table change_request, risk_level, risk_factors, mitigation_plan, count
```

### CAB (Change Advisory Board) Schedule
```spl
index=changemanagement sourcetype=service_now OR sourcetype=jira
| stats count by cab_meeting, change_request, meeting_date, cab_status
| table cab_meeting, change_request, meeting_date, cab_status, count
```

## 4. Capacity Planning Dashboard

### Resource Utilization Trends
```spl
index=capacity sourcetype=capacity_planning
| timechart span=1d avg(utilization) as avg_util by resource_type
| fields - NULL
```

### Capacity Forecasting
```spl
index=capacity sourcetype=capacity_planning
| timechart span=1w avg(utilization) as avg_util by resource_type
| trendline ema4(avg_util) as forecast
| fields - NULL
```

### Growth Projections
```spl
index=capacity sourcetype=capacity_planning
| stats count by resource_type, current_utilization, projected_utilization, growth_rate, months_until_capacity
| table resource_type, current_utilization, projected_utilization, growth_rate, months_until_capacity
```

### Resource Pool Status
```spl
index=capacity sourcetype=capacity_planning
| stats avg(utilization) as avg_util by resource_pool, resource_type
| table resource_pool, resource_type, avg_util
```

### Right-sizing Recommendations
```spl
index=capacity sourcetype=capacity_planning
| stats count by resource_id, resource_type, current_size, recommended_size, savings_potential
| table resource_id, resource_type, current_size, recommended_size, savings_potential, count
```

### Cost Optimization Opportunities
```spl
index=capacity sourcetype=capacity_planning
| stats sum(current_cost) as current_cost, sum(optimized_cost) as optimized_cost by resource_type
| eval savings=current_cost - optimized_cost, savings_pct=round((savings/current_cost)*100, 2)
| table resource_type, current_cost, optimized_cost, savings, savings_pct
```

### Budget vs Actual Usage
```spl
index=cost sourcetype=capacity_planning
| stats sum(actual_cost) as total_cost, sum(budget) as total_budget by department, month
| table department, month, total_cost, total_budget, variance
| eval variance=total_cost - total_budget
```

### Capacity Threshold Alerts
```spl
index=capacity sourcetype=capacity_planning
| stats avg(utilization) as avg_util by resource_type, resource_name
| where avg_util > 85
| table resource_type, resource_name, avg_util
```

### Resource Allocation Efficiency
```spl
index=capacity sourcetype=capacity_planning
| stats avg(allocation_pct) as avg_alloc, avg(utilization_pct) as avg_util by resource_type
| eval efficiency=round((avg_util/avg_alloc)*100, 2)
| table resource_type, avg_alloc, avg_util, efficiency
```

### Planning Horizon Overview
```spl
index=capacity sourcetype=capacity_planning
| stats count by resource_type, planning_horizon, capacity_status, action_required
| table resource_type, planning_horizon, capacity_status, action_required, count
```

## 5. SLA & SLO Dashboard

### SLA Compliance Status
```spl
index=sla sourcetype=sla:monitoring
| stats count as total, count(eval(status="compliant")) as compliant, count(eval(status="non_compliant")) as non_compliant by service, sla_name
| eval compliance_pct=round((compliant/total)*100, 2)
| table service, sla_name, total, compliant, non_compliant, compliance_pct
```

### SLO Achievement Rates
```spl
index=slo sourcetype=slo:monitoring
| stats avg(achievement_rate) as avg_achievement by service, slo_name
| table service, slo_name, avg_achievement
```

### Uptime Percentage
```spl
index=availability sourcetype=uptime:monitoring
| timechart span=1h avg(uptime_pct) as avg_uptime by service
| fields - NULL
```

### Response Time SLAs
```spl
index=performance sourcetype=apm OR sourcetype=webperf
| stats avg(response_time) as avg_response by service, sla_threshold
| eval compliance=if(avg_response <= sla_threshold, "compliant", "non_compliant")
| table service, avg_response, sla_threshold, compliance
```

### Error Rate SLOs
```spl
index=application sourcetype=access_combined
| stats count as total, count(eval(status >= 500)) as errors by service
| eval error_rate=round((errors/total)*100, 2)
| table service, error_rate
```

### Availability Metrics
```spl
index=availability sourcetype=uptime:monitoring
| stats avg(availability_pct) as avg_avail, max(downtime_minutes) as max_downtime by service
| table service, avg_avail, max_downtime
```

### Performance SLAs
```spl
index=performance sourcetype=apm
| stats avg(response_time) as avg_response, avg(throughput) as avg_throughput by service
| table service, avg_response, avg_throughput
```

### Recovery Objectives
```spl
index=sla sourcetype=sla:monitoring
| stats count by service, rto_threshold, rpo_threshold, actual_rto, actual_rpo
| table service, rto_threshold, rpo_threshold, actual_rto, actual_rpo
```

### Customer-facing SLAs
```spl
index=sla sourcetype=sla:monitoring
| stats count as total, count(eval(status="compliant")) as compliant by customer, sla_name
| eval compliance_pct=round((compliant/total)*100, 2)
| table customer, sla_name, total, compliant, compliance_pct
```

### SLA Breach Trends
```spl
index=sla sourcetype=sla:monitoring
| timechart span=1d count(eval(status="non_compliant")) as breaches by service
| fields - NULL
```

## 6. Job & Batch Processing Dashboard

### Scheduled Job Status
```spl
index=job sourcetype=crontab OR sourcetype=scheduler
| stats count by job_name, job_type, status, last_execution
| table job_name, job_type, status, last_execution, count
```

### Job Execution Timeline
```spl
index=job sourcetype=crontab OR sourcetype=scheduler
| timechart span=1h count as executions by job_name
| fields - NULL
```

### Job Success/Failure Rate
```spl
index=job sourcetype=crontab OR sourcetype=scheduler
| stats count as total, count(eval(status="success")) as success, count(eval(status="failed")) as failed by job_name
| eval success_rate=round((success/total)*100, 2)
| table job_name, total, success, failed, success_rate
```

### Job Duration Trends
```spl
index=job sourcetype=crontab OR sourcetype=scheduler
| timechart span=1d avg(duration) as avg_duration by job_name
| fields - NULL
```

### Job Queue Backlog
```spl
index=job sourcetype=airflow OR sourcetype=jenkins
| stats count as queue_depth by job_type, queue_name
| table job_type, queue_name, queue_depth
```

### Dependency Health
```spl
index=job sourcetype=airflow OR sourcetype=jenkins
| stats count by job_name, dependency_job, dependency_status
| table job_name, dependency_job, dependency_status, count
```

### Resource Utilization by Job
```spl
index=job sourcetype=crontab OR sourcetype=scheduler
| stats avg(cpu_usage) as avg_cpu, avg(memory_usage) as avg_mem by job_name
| table job_name, avg_cpu, avg_mem
```

### Job Retry Statistics
```spl
index=job sourcetype=airflow OR sourcetype=jenkins
| stats count as total, count(eval(retry_count > 0)) as retried, avg(retry_count) as avg_retries by job_name
| table job_name, total, retried, avg_retries
```

### Critical Job Monitoring
```spl
index=job sourcetype=crontab OR sourcetype=scheduler
| stats count by job_name, job_type, status, duration, priority
| where priority="critical" AND status="failed"
| table job_name, job_type, status, duration, priority, count
```

### Job Performance Analysis
```spl
index=job sourcetype=crontab OR sourcetype=scheduler
| stats avg(duration) as avg_duration, max(duration) as max_duration, min(duration) as min_duration by job_name
| table job_name, avg_duration, max_duration, min_duration
```

## 7. Monitoring System Health Dashboard

### Monitoring Agent Status
```spl
index=monitoring sourcetype=uf OR sourcetype=forwarder
| stats count by host, agent_status, agent_version, last_heartbeat
| table host, agent_status, agent_version, last_heartbeat, count
```

### Data Collection Health
```spl
index=monitoring sourcetype=metrics
| stats count as total, count(eval(collection_status="success")) as success, count(eval(collection_status="failed")) as failed by data_source
| eval success_rate=round((success/total)*100, 2)
| table data_source, total, success, failed, success_rate
```

### Alert Delivery Status
```spl
index=alert sourcetype=alert:delivery OR sourcetype=email
| stats count as total, count(eval(delivery_status="success")) as success, count(eval(delivery_status="failed")) as failed by alert_name, recipient
| eval success_rate=round((success/total)*100, 2)
| table alert_name, recipient, total, success, failed, success_rate
```

### Metric Shipping Status
```spl
index=metrics sourcetype=metrics
| stats count by metric_name, host, ship_status, error_count
| table metric_name, host, ship_status, error_count, count
```

### Log Aggregation Health
```spl
index=logs sourcetype=uf OR sourcetype=forwarder
| stats count as total, count(eval(ship_status="success")) as success, count(eval(ship_status="failed")) as failed by host, log_source
| eval success_rate=round((success/total)*100, 2)
| table host, log_source, total, success, failed, success_rate
```

### Dashboard Data Gaps
```spl
index=dashboard sourcetype=dashboard:metrics
| stats count by dashboard_name, data_source, last_update, gap_duration
| eval gap_duration=now-last_update
| where gap_duration > 3600
| table dashboard_name, data_source, last_update, gap_duration, count
```

### Notification System Status
```spl
index=notification sourcetype=email OR sourcetype=slack OR sourcetype=pagerduty
| stats count by notification_type, delivery_status, recipient
| table notification_type, delivery_status, recipient, count
```

### Monitoring Storage Utilization
```spl
index=monitoring sourcetype=storage
| stats avg(used_capacity) as avg_used, max(used_capacity) as max_used by storage_type
| eval capacity_pct=round((max_used/total_capacity)*100, 2)
| table storage_type, avg_used, max_used, capacity_pct
```

### Alert Rule Evaluation Status
```spl
index=alert sourcetype=alert:rules
| stats count by alert_rule, evaluation_status, last_evaluation, error_count
| table alert_rule, evaluation_status, last_evaluation, error_count, count
```

### Monitoring API Availability
```spl
index=monitoring sourcetype=api
| stats avg(response_time) as avg_response, count as total, count(eval(status >= 200 AND status < 300)) as success by api_endpoint
| eval success_rate=round((success/total)*100, 2)
| table api_endpoint, avg_response, total, success, success_rate
```

## 8. Service Desk & Support Dashboard

### Ticket Volume Trends
```spl
index=servicedesk sourcetype=service_now OR sourcetype=jira
| timechart span=1d count as tickets by priority
| fields - NULL
```

### Ticket Priority Distribution
```spl
index=servicedesk sourcetype=service_now OR sourcetype=jira
| stats count by priority, status
| chart count over priority by status
```

### SLA Breach Count
```spl
index=servicedesk sourcetype=service_now OR sourcetype=jira
| stats count by ticket_number, sla_type, sla_breach, breach_time
| where sla_breach="true"
| table ticket_number, sla_type, sla_breach, breach_time, count
```

### First Response Time
```spl
index=servicedesk sourcetype=service_now OR sourcetype=jira
| stats avg(first_response_time) as avg_response, max(first_response_time) as max_response by priority
| table priority, avg_response, max_response
```

### Resolution Time Trends
```spl
index=servicedesk sourcetype=service_now OR sourcetype=jira
| timechart span=1w avg(resolution_time) as avg_resolution by priority
| fields - NULL
```

### Team Workload Metrics
```spl
index=servicedesk sourcetype=service_now OR sourcetype=jira
| stats count as total, count(eval(status="open")) as open, count(eval(status="in_progress")) as in_progress by team, assignee
| table team, assignee, total, open, in_progress
```

### Customer Satisfaction Scores
```spl
index=servicedesk sourcetype=service_now OR sourcetype=jira
| stats avg(satisfaction_score) as avg_score by team, assignee
| table team, assignee, avg_score
```

### Knowledge Base Gaps
```spl
index=servicedesk sourcetype=kb OR sourcetype=service_now
| stats count by category, kb_article_count, usage_count, gap_status
| where gap_status="gap_identified"
| table category, kb_article_count, usage_count, gap_status, count
```

### Auto-resolution Rate
```spl
index=servicedesk sourcetype=service_now OR sourcetype=jira
| stats count as total, count(eval(resolution_type="auto")) as auto_resolved by ticket_type
| eval auto_rate=round((auto_resolved/total)*100, 2)
| table ticket_type, total, auto_resolved, auto_rate
```

### Escalation Triggers
```spl
index=servicedesk sourcetype=service_now OR sourcetype=jira
| stats count by ticket_number, current_level, escalation_level, escalation_reason
| where escalation_level > current_level
| table ticket_number, current_level, escalation_level, escalation_reason, count
```

## 9. Compliance & Auditing Dashboard

### Compliance Status Overview
```spl
index=compliance sourcetype=compliance:scanner
| stats count as total, count(eval(status="compliant")) as compliant, count(eval(status="non_compliant")) as non_compliant by standard, control_id
| eval compliance_pct=round((compliant/total)*100, 2)
| table standard, control_id, total, compliant, non_compliant, compliance_pct
```

### Audit Trail Completeness
```spl
index=audit sourcetype=auditd OR sourcetype=windows:security
| stats count by system, audit_source, log_retention_days, coverage_pct
| table system, audit_source, log_retention_days, coverage_pct
```

### Policy Violation Summary
```spl
index=audit sourcetype=auditd OR sourcetype=configuration
| stats count by policy_name, violation_type, severity, user
| table policy_name, violation_type, severity, user, count
| sort - severity
```

### Regulatory Reporting Status
```spl
index=compliance sourcetype=regulatory:reporting
| stats count by report_type, due_date, submission_status, submitted_date
| eval days_remaining=round((due_date-now)/86400, 0)
| where days_remaining < 30
| table report_type, due_date, submission_status, submitted_date, days_remaining, count
```

### Audit Finding Status
```spl
index=compliance sourcetype=compliance:scanner
| stats count by finding_id, severity, status, remediation_due_date
| table finding_id, severity, status, remediation_due_date, count
```

### Risk Assessment Status
```spl
index=risk sourcetype=risk:assessment
| stats count by risk_category, risk_level, likelihood, impact, mitigation_status
| table risk_category, risk_level, likelihood, impact, mitigation_status, count
```

### Access Review Progress
```spl
index=compliance sourcetype=access_review
| stats count as total, count(eval(status="completed")) as completed, count(eval(status="pending")) as pending by review_type, reviewer
| table review_type, reviewer, total, completed, pending
```

### Compliance Certification Tracking
```spl
index=compliance sourcetype=certification
| stats count by certification_name, expiry_date, status, last_audit
| eval days_remaining=round((expiry_date-now)/86400, 0)
| table certification_name, expiry_date, status, last_audit, days_remaining
```

### Control Effectiveness
```spl
index=compliance sourcetype=compliance:scanner
| stats avg(effectiveness_score) as avg_effectiveness by control_id, control_type
| table control_id, control_type, avg_effectiveness
```

### Audit Schedule
```spl
index=audit sourcetype=audit:calendar
| stats count by audit_type, scheduled_date, auditor, status
| where scheduled_date > now AND scheduled_date < now+90
| table audit_type, scheduled_date, auditor, status, count
```

## 10. Cost & Resource Optimization Dashboard

### Cloud Cost Breakdown
```spl
index=cloud sourcetype=aws:cost OR sourcetype=azure:cost
| stats sum(cost) as total_cost by service, region
| eval cost_usd=round(total_cost, 2)
| table service, region, cost_usd
| sort - cost_usd
```

### Cost Trend Analysis
```spl
index=cloud sourcetype=aws:cost OR sourcetype=azure:cost
| timechart span=1d sum(cost) as daily_cost by service
| fields - NULL
```

### Resource Utilization vs Cost
```spl
index=cloud sourcetype=aws:cloudwatch OR sourcetype=azure:monitor
| stats avg(cpu_usage) as avg_cpu, avg(memory_usage) as avg_mem, sum(cost) as total_cost by instance_id, instance_type
| table instance_id, instance_type, avg_cpu, avg_mem, total_cost
```

### Idle Resource Detection
```spl
index=cloud sourcetype=aws:cloudwatch OR sourcetype=azure:monitor
| stats avg(cpu_usage) as avg_cpu, avg(network_in) as avg_net_in by resource_id, resource_type
| where avg_cpu < 5 AND avg_net_in < 1024
| table resource_id, resource_type, avg_cpu, avg_net_in
| eval status="idle"
```

### Right-sizing Opportunities
```spl
index=cloud sourcetype=aws:cloudwatch OR sourcetype=azure:monitor
| stats avg(cpu_usage) as avg_cpu, avg(memory_usage) as avg_mem by instance_id, instance_type
| where avg_cpu < 20 AND avg_mem < 20
| table instance_id, instance_type, avg_cpu, avg_mem
| eval recommendation="downsize"
```

### Reserved Instance Coverage
```spl
index=cloud sourcetype=aws:cost
| stats sum(reserved_hours) as reserved, sum(on_demand_hours) as on_demand by instance_type
| eval coverage_pct=round((reserved/(reserved+on_demand))*100, 2)
| table instance_type, reserved, on_demand, coverage_pct
```

### Budget vs Actual Spending
```spl
index=cost sourcetype=cost:monitoring
| stats sum(actual_cost) as total_cost, sum(budget) as total_budget by department, month
| table department, month, total_cost, total_budget, variance
| eval variance=total_cost - total_budget
```

### Cost Anomaly Detection
```spl
index=cloud sourcetype=aws:cost OR sourcetype=azure:cost
| timechart span=1d sum(cost) as daily_cost by service
| trendline ema7(daily_cost) as forecast
| where daily_cost > (forecast * 1.5)
| table _time, service, daily_cost, forecast
```

### Cost Allocation by Department
```spl
index=cost sourcetype=cost:monitoring
| stats sum(cost) as total_cost by department, cost_center, service
| table department, cost_center, service, total_cost
```

### Optimization Recommendations
```spl
index=cloud sourcetype=aws:cost OR sourcetype=azure:cost
| stats count by resource_id, resource_type, optimization_type, potential_savings
| table resource_id, resource_type, optimization_type, potential_savings, count
```

## 11. Maintenance & Patching Dashboard

### Patch Status Overview
```spl
index=maintenance sourcetype=patch OR sourcetype=wsus
| stats count as total, count(eval(patch_status="installed")) as installed, count(eval(patch_status="missing")) as missing by system, patch_type
| table system, patch_type, total, installed, missing
```

### Vulnerability Patching Progress
```spl
index=maintenance sourcetype=patch OR sourcetype=vulnerability
| stats count as total, count(eval(status="patched")) as patched, count(eval(status="vulnerable")) as vulnerable by system, severity
| table system, severity, total, patched, vulnerable
```

### Maintenance Window Schedule
```spl
index=maintenance sourcetype=maintenance:calendar
| stats count by system, maintenance_type, start_time, end_time, status
| where start_time > now AND start_time < now+604800
| table system, maintenance_type, start_time, end_time, status, count
```

### Patch Success Rate
```spl
index=maintenance sourcetype=patch OR sourcetype=wsus
| stats count as total, count(eval(status="success")) as success, count(eval(status="failed")) as failed by system
| eval success_rate=round((success/total)*100, 2)
| table system, total, success, failed, success_rate
```

### System Restart Schedule
```spl
index=maintenance sourcetype=system OR sourcetype=uptime
| stats count by system, restart_reason, scheduled_time, status
| where scheduled_time > now AND scheduled_time < now+604800
| table system, restart_reason, scheduled_time, status, count
```

### Backup Status
```spl
index=backup sourcetype=vbr OR sourcetype=veeam
| stats count by system, backup_status, last_backup, backup_type
| table system, backup_status, last_backup, backup_type, count
```

### Compliance Patching
```spl
index=maintenance sourcetype=patch OR sourcetype=compliance
| stats count as total, count(eval(compliance_status="compliant")) as compliant, count(eval(compliance_status="non_compliant")) as non_compliant by system, compliance_standard
| table system, compliance_standard, total, compliant, non_compliant
```

### Outage Planning
```spl
index=maintenance sourcetype=maintenance:calendar
| stats count by system, outage_type, scheduled_duration, impact_assessment
| table system, outage_type, scheduled_duration, impact_assessment, count
```

### Maintenance Impact Analysis
```spl
index=maintenance sourcetype=maintenance:calendar
| stats count by system, affected_services, impact_level, notification_status
| table system, affected_services, impact_level, notification_status, count
```

### Post-maintenance Validation
```spl
index=maintenance sourcetype=maintenance:calendar
| stats count by system, maintenance_type, validation_status, validation_time
| where validation_status="failed"
| table system, maintenance_type, validation_status, validation_time, count
```

## 12. Team Performance Dashboard

### Team Workload Distribution
```spl
index=incident sourcetype=service_now OR sourcetype=jira
| stats count as total, count(eval(status="open")) as open, count(eval(status="in_progress")) as in_progress by team, assignee
| table team, assignee, total, open, in_progress
```

### Task Completion Rates
```spl
index=tasks sourcetype=service_now OR sourcetype=jira
| stats count as total, count(eval(status="completed")) as completed by team, assignee
| eval completion_rate=round((completed/total)*100, 2)
| table team, assignee, total, completed, completion_rate
```

### Response Time Metrics
```spl
index=incident sourcetype=service_now OR sourcetype=jira
| stats avg(response_time) as avg_response, max(response_time) as max_response by team, assignee
| table team, assignee, avg_response, max_response
```

### Efficiency Trends
```spl
index=incident sourcetype=service_now OR sourcetype=jira
| timechart span=1w avg(resolution_time) as avg_resolution by team
| fields - NULL
```

### Skill Gap Analysis
```spl
index=training sourcetype=training:records
| stats count by skill, team, required_level, current_level, gap_status
| where gap_status="gap"
| table skill, team, required_level, current_level, gap_status, count
```

### Training Requirements
```spl
index=training sourcetype=training:records
| stats count by team, training_type, required_training, completion_status
| where completion_status="not_completed"
| table team, training_type, required_training, completion_status, count
```

### Onboarding Progress
```spl
index=hr sourcetype=onboarding
| stats count by employee, team, onboarding_stage, completion_pct
| where completion_pct < 100
| table employee, team, onboarding_stage, completion_pct
```

### Team Availability
```spl
index=hr sourcetype=availability
| stats count by team, member, availability_status, available_hours
| table team, member, availability_status, available_hours, count
```

### Performance KPIs
```spl
index=performance sourcetype=performance:metrics
| stats avg(kpi_value) as avg_kpi, max(kpi_value) as max_kpi by team, kpi_name
| table team, kpi_name, avg_kpi, max_kpi
```

### Capacity Planning for Team
```spl
index=hr sourcetype=capacity_planning
| stats count by team, current_capacity, projected_demand, hiring_needed
| table team, current_capacity, projected_demand, hiring_needed, count
```
