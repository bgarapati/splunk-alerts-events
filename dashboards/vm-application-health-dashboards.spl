# VM & Application Health Dashboards - SPL Queries

## 1. VM Health Overview Dashboard

### Total VM Count by Status
```spl
index=vmware sourcetype=vmware:inv:vm
| stats count by power_state, host_name
| table power_state, host_name, count
```

### VM Availability Percentage
```spl
index=vmware sourcetype=vmware:inv:vm
| stats count as total, count(eval(power_state="poweredOn")) as available by host_name
| eval availability_pct=round((available/total)*100, 2)
| table host_name, total, available, availability_pct
```

### Critical VM Alerts
```spl
index=vmware sourcetype=vmware:perf:vm
| stats count by vm_name, alert_type, severity
| where severity IN (critical, high)
| table vm_name, alert_type, severity, count
| sort - severity
```

### VM Resource Utilization Summary
```spl
index=vmware sourcetype=vmware:perf:vm
| stats avg(cpu_usage) as avg_cpu, avg(mem_usage) as avg_mem, avg(disk_usage) as avg_disk by vm_name, host_name
| table vm_name, host_name, avg_cpu, avg_mem, avg_disk
```

### VM Health Score Distribution
```spl
index=vmware sourcetype=vmware:perf:vm
| stats avg(cpu_usage) as avg_cpu, avg(mem_usage) as avg_mem by vm_name
| eval health_score=round(100 - (avg_cpu + avg_mem)/2, 2)
| stats count by health_score, health_category
| eval health_category=if(health_score >= 90, "excellent", if(health_score >= 70, "good", if(health_score >= 50, "fair", "poor")))
| table health_score, health_category, count
```

### Host Cluster Health
```spl
index=vmware sourcetype=vmware:perf:cluster
| stats avg(cpu_usage) as avg_cpu, avg(mem_usage) as avg_mem by cluster_name
| table cluster_name, avg_cpu, avg_mem
```

### VM Lifecycle Status
```spl
index=vmware sourcetype=vmware:inv:vm
| stats count by vm_name, lifecycle_status, creation_time
| table vm_name, lifecycle_status, creation_time, count
```

### VM Migration Activity
```spl
index=vmware sourcetype=vmware:inv:vm
| stats count by vm_name, source_host, dest_host, migration_status, migration_time
| table vm_name, source_host, dest_host, migration_status, migration_time, count
```

### VM Backup Status
```spl
index=backup sourcetype=vbr OR sourcetype=veeam
| stats count by vm_name, backup_status, last_backup, backup_type
| table vm_name, backup_status, last_backup, backup_type, count
```

### VM Compliance Status
```spl
index=vmware sourcetype=vmware:inv:vm
| stats count by vm_name, compliance_standard, compliance_status, last_scan
| table vm_name, compliance_standard, compliance_status, last_scan, count
```

## 2. VM Performance Dashboard

### CPU Utilization Trends (Per VM)
```spl
index=vmware sourcetype=vmware:perf:vm
| timechart span=5m avg(cpu_usage) as avg_cpu by vm_name
| fields - NULL
```

### Memory Utilization Trends (Per VM)
```spl
index=vmware sourcetype=vmware:perf:vm
| timechart span=5m avg(mem_usage) as avg_mem by vm_name
| fields - NULL
```

### Disk I/O Performance (Per VM)
```spl
index=vmware sourcetype=vmware:perf:vm
| stats avg(read_iops) as avg_read, avg(write_iops) as avg_write, avg(latency) as avg_latency by vm_name
| table vm_name, avg_read, avg_write, avg_latency
```

### Network Throughput (Per VM)
```spl
index=vmware sourcetype=vmware:perf:vm
| stats avg(network_in) as avg_in, avg(network_out) as avg_out by vm_name
| table vm_name, avg_in, avg_out
```

### VM Resource Contention
```spl
index=vmware sourcetype=vmware:perf:vm
| stats avg(cpu_ready) as avg_ready, avg(cpu_wait) as avg_wait by vm_name, host_name
| table vm_name, host_name, avg_ready, avg_wait
```

### VM Performance Score
```spl
index=vmware sourcetype=vmware:perf:vm
| stats avg(cpu_usage) as avg_cpu, avg(mem_usage) as avg_mem, avg(disk_latency) as avg_latency by vm_name
| eval performance_score=round(100 - (avg_cpu + avg_mem + avg_latency)/3, 2)
| table vm_name, avg_cpu, avg_mem, avg_latency, performance_score
| sort performance_score
```

### Top Resource-Consuming VMs
```spl
index=vmware sourcetype=vmware:perf:vm
| stats avg(cpu_usage) as avg_cpu, avg(mem_usage) as avg_mem by vm_name
| eval total_usage=avg_cpu + avg_mem
| table vm_name, avg_cpu, avg_mem, total_usage
| sort - total_usage
| head 20
```

### VM Performance Degradation Alerts
```spl
index=vmware sourcetype=vmware:perf:vm
| timechart span=1h avg(cpu_usage) as avg_cpu, avg(mem_usage) as avg_mem by vm_name
| trendline ema5(avg_cpu) as cpu_trend, ema5(avg_mem) as mem_trend
| where avg_cpu > (cpu_trend * 1.5) OR avg_mem > (mem_trend * 1.5)
| table _time, vm_name, avg_cpu, avg_mem, cpu_trend, mem_trend
```

### Capacity Forecasting
```spl
index=vmware sourcetype=vmware:perf:vm
| timechart span=1d avg(cpu_usage) as avg_cpu, avg(mem_usage) as avg_mem by vm_name
| trendline ema7(avg_cpu) as cpu_forecast, ema7(avg_mem) as mem_forecast
| fields - NULL
```

### Resource Over-provisioning Detection
```spl
index=vmware sourcetype=vmware:perf:vm
| stats avg(cpu_usage) as avg_cpu, avg(mem_usage) as avg_mem by vm_name, allocated_cpu, allocated_mem
| where avg_cpu < 20 AND avg_mem < 20
| table vm_name, allocated_cpu, allocated_mem, avg_cpu, avg_mem
| eval recommendation="downsize"
```

## 3. VM Availability Dashboard

### VM Uptime Percentage
```spl
index=vmware sourcetype=vmware:inv:vm
| stats avg(uptime_days) as avg_uptime, max(uptime_days) as max_uptime by vm_name
| table vm_name, avg_uptime, max_uptime
```

### Downtime Incidents
```spl
index=vmware sourcetype=vmware:inv:vm
| stats count by vm_name, downtime_reason, downtime_duration, downtime_start
| table vm_name, downtime_reason, downtime_duration, downtime_start, count
```

### VM Restart Events
```spl
index=vmware sourcetype=vmware:inv:vm
| stats count by vm_name, restart_reason, restart_time, user
| table vm_name, restart_reason, restart_time, user, count
```

### VM Crash Detection
```spl
index=vmware sourcetype=vmware:perf:vm
| stats count by vm_name, crash_type, crash_time, host_name
| table vm_name, crash_type, crash_time, host_name, count
```

### HA (High Availability) Status
```spl
index=vmware sourcetype=vmware:perf:cluster
| stats count by cluster_name, ha_status, ha_enabled, restart_priority
| table cluster_name, ha_status, ha_enabled, restart_priority, count
```

### VM Failover Events
```spl
index=vmware sourcetype=vmware:inv:vm
| stats count by vm_name, source_host, dest_host, failover_reason, failover_time
| table vm_name, source_host, dest_host, failover_reason, failover_time, count
```

### VM Health Check Status
```spl
index=vmware sourcetype=vmware:healthcheck
| stats count by vm_name, health_check_type, health_status, last_check
| table vm_name, health_check_type, health_status, last_check, count
```

### Recovery Time Metrics
```spl
index=vmware sourcetype=vmware:inv:vm
| stats avg(recovery_time) as avg_recovery, max(recovery_time) as max_recovery by vm_name, recovery_type
| table vm_name, recovery_type, avg_recovery, max_recovery
```

### Availability SLA Compliance
```spl
index=vmware sourcetype=vmware:inv:vm
| stats avg(availability_pct) as avg_avail by vm_name, sla_threshold
| eval compliance=if(avg_avail >= sla_threshold, "compliant", "non_compliant")
| table vm_name, avg_avail, sla_threshold, compliance
```

### Dependency Health Status
```spl
index=vmware sourcetype=vmware:inv:vm
| stats count by vm_name, dependency_type, dependency_name, dependency_status
| table vm_name, dependency_type, dependency_name, dependency_status, count
```

## 4. VM Security Dashboard

### VM Vulnerability Status
```spl
index=vmware sourcetype=vmware:inv:vm OR sourcetype=vulnerability
| stats count by vm_name, vulnerability_severity, cve_id, patch_status
| table vm_name, vulnerability_severity, cve_id, patch_status, count
| sort - vulnerability_severity
```

### Patch Compliance Level
```spl
index=vmware sourcetype=vmware:inv:vm OR sourcetype=patch
| stats count as total, count(eval(patch_status="installed")) as installed, count(eval(patch_status="missing")) as missing by vm_name
| eval compliance_pct=round((installed/total)*100, 2)
| table vm_name, total, installed, missing, compliance_pct
```

### Antivirus/EDR Status
```spl
index=vmware sourcetype=vmware:inv:vm OR sourcetype=antivirus
| stats count by vm_name, av_status, av_version, last_scan
| table vm_name, av_status, av_version, last_scan, count
```

### Firewall Rule Compliance
```spl
index=vmware sourcetype=vmware:inv:vm OR sourcetype=firewall
| stats count by vm_name, firewall_status, rule_count, compliance_status
| table vm_name, firewall_status, rule_count, compliance_status, count
```

### VM Network Security
```spl
index=vmware sourcetype=vmware:inv:vm OR sourcetype=network
| stats count by vm_name, network_policy, port_status, security_group
| table vm_name, network_policy, port_status, security_group, count
```

### Unauthorized Access Attempts
```spl
index=vmware sourcetype=vmware:inv:vm OR sourcetype=authentication
| stats count by vm_name, access_type, user, src_ip, access_status
| where access_status="denied"
| table vm_name, access_type, user, src_ip, access_status, count
```

### VM Configuration Drift
```spl
index=vmware sourcetype=vmware:inv:vm OR sourcetype=configuration
| stats count by vm_name, config_item, expected_value, actual_value, drift_status
| where drift_status="drifted"
| table vm_name, config_item, expected_value, actual_value, drift_status, count
```

### Security Policy Violations
```spl
index=vmware sourcetype=vmware:inv:vm OR sourcetype=security
| stats count by vm_name, policy_name, violation_type, severity
| table vm_name, policy_name, violation_type, severity, count
```

### VM Isolation Status
```spl
index=vmware sourcetype=vmware:inv:vm
| stats count by vm_name, isolation_status, isolation_reason, isolation_time
| table vm_name, isolation_status, isolation_reason, isolation_time, count
```

### Security Scan Results
```spl
index=vmware sourcetype=vmware:inv:vm OR sourcetype=security_scan
| stats count by vm_name, scan_type, scan_status, findings_count, last_scan
| table vm_name, scan_type, scan_status, findings_count, last_scan, count
```

## 5. Application Health Overview Dashboard

### Application Availability Status
```spl
index=application sourcetype=healthcheck OR sourcetype=ping
| stats count by application, health_status, last_check
| table application, health_status, last_check, count
```

### Active Application Instances
```spl
index=application sourcetype=application:logs
| stats count as total, count(eval(status="running")) as running, count(eval(status="stopped")) as stopped by application
| table application, total, running, stopped
```

### Critical Application Alerts
```spl
index=application severity IN (critical, high)
| stats count by application, alert_name, severity
| table application, alert_name, severity, count
| sort - severity
```

### Application Health Score
```spl
index=application sourcetype=healthcheck OR sourcetype=apm
| stats avg(health_score) as avg_health by application
| table application, avg_health
```

### Application Dependency Status
```spl
index=application sourcetype=dependency OR sourcetype=apm
| stats count by application, dependency_name, dependency_type, health_status
| table application, dependency_name, dependency_type, health_status, count
```

### Deployment Status
```spl
index=deploy sourcetype=deployment
| stats count by application, deployment_status, environment, deployment_time
| table application, deployment_status, environment, deployment_time, count
```

### SLA Compliance Overview
```spl
index=sla sourcetype=application
| stats count as total, count(eval(status="compliant")) as compliant, count(eval(status="non_compliant")) as non_compliant by application
| eval compliance_pct=round((compliant/total)*100, 2)
| table application, total, compliant, non_compliant, compliance_pct
```

### Error Rate Summary
```spl
index=application sourcetype=access_combined OR sourcetype=nginx:access
| stats count as total, count(eval(status >= 400)) as errors by application
| eval error_rate=round((errors/total)*100, 2)
| table application, total, errors, error_rate
```

### Response Time Summary
```spl
index=application sourcetype=access_combined OR sourcetype=apm
| stats avg(response_time) as avg_response by application
| table application, avg_response
```

### User Impact Assessment
```spl
index=application sourcetype=access_combined OR sourcetype=rum
| stats dc(user) as affected_users by application, status
| where status >= 400
| table application, status, affected_users
```

## 6. Application Performance Dashboard

### Response Time Percentiles (p50, p95, p99)
```spl
index=application sourcetype=access_combined OR sourcetype=apm
| stats perc50(response_time) as p50, perc95(response_time) as p95, perc99(response_time) as p99 by application
| table application, p50, p95, p99
```

### Error Rate Trends
```spl
index=application sourcetype=access_combined
| timechart span=5m count(eval(status >= 400)) as errors, count as total by application
| eval error_rate=round((errors/total)*100, 2)
| fields - NULL
```

### Throughput Metrics
```spl
index=application sourcetype=access_combined
| timechart span=5m count as requests by application
| fields - NULL
```

### Application Resource Utilization
```spl
index=application sourcetype=jmx OR sourcetype=metrics
| stats avg(cpu_usage) as avg_cpu, avg(memory_usage) as avg_mem by application
| table application, avg_cpu, avg_mem
```

### Database Performance Impact
```spl
index=database sourcetype=slowquery OR sourcetype=database:performance
| stats avg(query_duration) as avg_duration by application, database
| table application, database, avg_duration
```

### External Service Latency
```spl
index=application sourcetype=apm OR sourcetype=api:logs
| stats avg(response_time) as avg_latency by application, external_service
| table application, external_service, avg_latency
```

### Cache Performance
```spl
index=application sourcetype=redis OR sourcetype=memcached
| stats sum(cache_hits) as hits, sum(cache_misses) as misses by application
| eval hit_rate=round((hits/(hits+misses))*100, 2)
| table application, hits, misses, hit_rate
```

### Memory Leak Detection
```spl
index=application sourcetype=jmx OR sourcetype=metrics
| timechart span=1h avg(heap_memory) as avg_heap by application
| trendline sma5(avg_heap) as trend
| where avg_heap > (trend * 1.5)
| table _time, application, avg_heap, trend
```

### Thread Pool Status
```spl
index=application sourcetype=jmx OR sourcetype=metrics
| stats avg(active_threads) as avg_threads, max(threads) as max_threads by application
| eval utilization_pct=round((avg_threads/max_threads)*100, 2)
| table application, avg_threads, max_threads, utilization_pct
```

### Performance Bottleneck Identification
```spl
index=application sourcetype=apm
| stats avg(duration) as avg_duration by application, span_name, component
| sort - avg_duration
| head 20
```

## 7. Application Availability Dashboard

### Application Uptime Percentage
```spl
index=application sourcetype=uptime:monitoring
| stats avg(uptime_pct) as avg_uptime by application
| table application, avg_uptime
```

### Downtime Incidents
```spl
index=application sourcetype=incident OR sourcetype=uptime:monitoring
| stats count by application, downtime_reason, downtime_duration, downtime_start
| table application, downtime_reason, downtime_duration, downtime_start, count
```

### Service Restart Events
```spl
index=application sourcetype=systemd OR sourcetype=supervisord
| stats count by application, restart_reason, restart_time, host
| table application, restart_reason, restart_time, host, count
```

### Health Check Failures
```spl
index=application sourcetype=healthcheck OR sourcetype=ping
| stats count by application, health_check_type, health_status, last_check
| where health_status="unhealthy"
| table application, health_check_type, health_status, last_check, count
```

### Load Balancer Health
```spl
index=loadbalancer sourcetype=f5:bigip OR sourcetype=aws:elb
| stats count by application, pool_name, member_ip, health_status
| table application, pool_name, member_ip, health_status, count
```

### Auto-scaling Status
```spl
index=cloud sourcetype=aws:cloudwatch
| stats count by application, AutoScalingGroupName, desired_capacity, current_capacity, status
| table application, AutoScalingGroupName, desired_capacity, current_capacity, status, count
```

### Failover Events
```spl
index=application sourcetype=failover OR sourcetype=ha
| stats count by application, failover_type, source, destination, failover_time
| table application, failover_type, source, destination, failover_time, count
```

### Recovery Time Metrics
```spl
index=application sourcetype=incident OR sourcetype=uptime:monitoring
| stats avg(recovery_time) as avg_recovery, max(recovery_time) as max_recovery by application
| table application, avg_recovery, max_recovery
```

### Availability SLA Compliance
```spl
index=sla sourcetype=application
| stats avg(availability_pct) as avg_avail by application, sla_threshold
| eval compliance=if(avg_avail >= sla_threshold, "compliant", "non_compliant")
| table application, avg_avail, sla_threshold, compliance
```

### Dependency Health Status
```spl
index=application sourcetype=dependency OR sourcetype=apm
| stats count by application, dependency_name, dependency_type, health_status
| table application, dependency_name, dependency_type, health_status, count
```

## 8. Application Dependency Dashboard

### Service Dependency Map
```spl
index=application sourcetype=apm OR sourcetype=service_mesh
| stats count by service_name, dependency_service, connection_type
| table service_name, dependency_service, connection_type, count
```

### Dependency Health Status
```spl
index=application sourcetype=dependency OR sourcetype=apm
| stats count by application, dependency_name, dependency_type, health_status
| table application, dependency_name, dependency_type, health_status, count
```

### Cascading Failure Risk
```spl
index=application sourcetype=dependency OR sourcetype=apm
| stats count by application, dependency_name, health_status, criticality
| where health_status="unhealthy" AND criticality="high"
| table application, dependency_name, health_status, criticality, count
```

### Service Inter-dependencies
```spl
index=application sourcetype=apm OR sourcetype=service_mesh
| stats count by service_name, dependency_count, dependent_on_count
| table service_name, dependency_count, dependent_on_count
```

### Database Dependency Health
```spl
index=database sourcetype=database:performance
| stats avg(response_time) as avg_response, avg(connection_pool) as avg_pool by application, database
| table application, database, avg_response, avg_pool
```

### External API Dependency Status
```spl
index=api sourcetype=api:logs
| stats avg(response_time) as avg_latency, count as total, count(eval(status >= 400)) as errors by application, external_service
| eval error_rate=round((errors/total)*100, 2)
| table application, external_service, avg_latency, error_rate
```

### Message Queue Health
```spl
index=microservices sourcetype=kafka OR sourcetype=rabbitmq
| stats avg(queue_depth) as avg_depth, max(queue_depth) as max_depth by application, queue_name
| table application, queue_name, avg_depth, max_depth
```

### Cache Dependency Status
```spl
index=application sourcetype=redis OR sourcetype=memcached
| stats avg(hit_rate) as avg_hit, avg(response_time) as avg_response by application
| table application, avg_hit, avg_response
```

### Network Dependency Health
```spl
index=network sourcetype=network:performance
| stats avg(latency) as avg_latency, avg(packet_loss) as avg_loss by application, dependency
| table application, dependency, avg_latency, avg_loss
```

### Critical Path Analysis
```spl
index=application sourcetype=apm
| stats avg(duration) as avg_duration by application, trace_id, critical_path
| table application, trace_id, critical_path, avg_duration
| sort - avg_duration
```

## 9. VM to Application Mapping Dashboard

### VM to Application Inventory
```spl
index=vmware sourcetype=vmware:inv:vm OR index=application sourcetype=application:logs
| stats count by vm_name, application_name, host_name
| table vm_name, application_name, host_name, count
```

### Application Distribution Across VMs
```spl
index=application sourcetype=application:logs
| stats count by application_name, vm_name, host_name
| table application_name, vm_name, host_name, count
```

### Resource Allocation per Application
```spl
index=vmware sourcetype=vmware:perf:vm OR index=application sourcetype=application:logs
| stats avg(cpu_usage) as avg_cpu, avg(mem_usage) as avg_mem by application_name, vm_name
| table application_name, vm_name, avg_cpu, avg_mem
```

### VM Capacity vs Application Demand
```spl
index=vmware sourcetype=vmware:perf:vm OR index=application sourcetype=application:logs
| stats avg(vm_capacity) as capacity, avg(app_demand) as demand by vm_name, application_name
| eval utilization_pct=round((demand/capacity)*100, 2)
| table vm_name, application_name, capacity, demand, utilization_pct
```

### Consolidation Opportunities
```spl
index=vmware sourcetype=vmware:perf:vm OR index=application sourcetype=application:logs
| stats avg(cpu_usage) as avg_cpu, avg(mem_usage) as avg_mem by vm_name
| where avg_cpu < 30 AND avg_mem < 30
| table vm_name, avg_cpu, avg_mem
| eval recommendation="consolidate"
```

### Migration Planning
```spl
index=vmware sourcetype=vmware:inv:vm OR index=application sourcetype=application:logs
| stats count by vm_name, application_name, current_host, target_host, migration_priority
| table vm_name, application_name, current_host, target_host, migration_priority, count
```

### Application Placement Optimization
```spl
index=vmware sourcetype=vmware:perf:vm OR index=application sourcetype=application:logs
| stats avg(cpu_usage) as avg_cpu, avg(mem_usage) as avg_mem by application_name, host_name
| table application_name, host_name, avg_cpu, avg_mem
```

### Resource Contention Analysis
```spl
index=vmware sourcetype=vmware:perf:vm OR index=application sourcetype=application:logs
| stats avg(cpu_contention) as avg_contention by vm_name, application_name
| table vm_name, application_name, avg_contention
```

### Cost Allocation by Application
```spl
index=vmware sourcetype=vmware:cost OR index=application sourcetype=application:logs
| stats sum(cost) as total_cost by application_name, vm_name
| table application_name, vm_name, total_cost
```

### Multi-tenancy Status
```spl
index=vmware sourcetype=vmware:inv:vm OR index=application sourcetype=application:logs
| stats count by vm_name, tenant_count, application_count
| table vm_name, tenant_count, application_count
```

## 10. Capacity Planning Dashboard (VM & App)

### VM Capacity Forecasting
```spl
index=vmware sourcetype=vmware:perf:vm
| timechart span=1d avg(cpu_usage) as avg_cpu, avg(mem_usage) as avg_mem by vm_name
| trendline ema7(avg_cpu) as cpu_forecast, ema7(avg_mem) as mem_forecast
| fields - NULL
```

### Application Growth Trends
```spl
index=application sourcetype=access_combined
| timechart span=1d count as requests by application
| trendline ema7(requests) as forecast
| fields - NULL
```

### Resource Utilization Projections
```spl
index=vmware sourcetype=vmware:perf:vm OR index=application sourcetype=application:logs
| stats avg(utilization) as avg_util by resource_type, resource_name
| table resource_type, resource_name, avg_util
```

### Capacity Threshold Alerts
```spl
index=vmware sourcetype=vmware:perf:vm OR index=application sourcetype=application:logs
| stats avg(utilization) as avg_util by resource_type, resource_name
| where avg_util > 85
| table resource_type, resource_name, avg_util
```

### Right-sizing Recommendations
```spl
index=vmware sourcetype=vmware:perf:vm OR index=application sourcetype=application:logs
| stats avg(cpu_usage) as avg_cpu, avg(mem_usage) as avg_mem by vm_name, application_name
| where avg_cpu < 20 AND avg_mem < 20
| table vm_name, application_name, avg_cpu, avg_mem
| eval recommendation="downsize"
```

### Consolidation Opportunities
```spl
index=vmware sourcetype=vmware:perf:vm
| stats avg(cpu_usage) as avg_cpu, avg(mem_usage) as avg_mem by vm_name
| where avg_cpu < 30 AND avg_mem < 30
| table vm_name, avg_cpu, avg_mem
| eval recommendation="consolidate"
```

### Scaling Recommendations
```spl
index=application sourcetype=access_combined
| timechart span=1h count as requests by application
| trendline ema5(requests) as forecast
| where requests > (forecast * 1.5)
| table _time, application, requests, forecast
| eval recommendation="scale_up"
```

### Budget Forecasting
```spl
index=cost sourcetype=vmware:cost OR sourcetype=application:cost
| timechart span=1m sum(cost) as monthly_cost by cost_category
| trendline ema3(monthly_cost) as forecast
| fields - NULL
```

### Capacity Optimization Opportunities
```spl
index=vmware sourcetype=vmware:perf:vm OR index=application sourcetype=application:logs
| stats count by resource_type, optimization_type, potential_savings
| table resource_type, optimization_type, potential_savings, count
```

### Planning Horizon Overview
```spl
index=capacity sourcetype=capacity_planning
| stats count by resource_type, planning_horizon, capacity_status, action_required
| table resource_type, planning_horizon, capacity_status, action_required, count
```

### Resource Efficiency Metrics
```spl
index=vmware sourcetype=vmware:perf:vm OR index=application sourcetype=application:logs
| stats avg(allocation_pct) as avg_alloc, avg(utilization_pct) as avg_util by resource_type
| eval efficiency=round((avg_util/avg_alloc)*100, 2)
| table resource_type, avg_alloc, avg_util, efficiency
```

## 11. Alert Correlation Dashboard

### VM and Application Alert Correlation
```spl
index=vmware sourcetype=vmware:perf:vm OR index=application sourcetype=application:logs
| stats count by alert_type, severity, source_type, affected_resource
| table alert_type, severity, source_type, affected_resource, count
```

### Root Cause Analysis
```spl
index=vmware sourcetype=vmware:perf:vm OR index=application sourcetype=application:logs
| stats count by root_cause, affected_applications, affected_vms, correlation_score
| table root_cause, affected_applications, affected_vms, correlation_score, count
```

### Alert Storm Detection
```spl
index=vmware sourcetype=vmware:perf:vm OR index=application sourcetype=application:logs
| timechart span=5m count as alerts by severity
| eventstats sum(alerts) as total_alerts
| where total_alerts > 100
| table _time, severity, alerts, total_alerts
```

### Alert Priority Distribution
```spl
index=vmware sourcetype=vmware:perf:vm OR index=application sourcetype=application:logs
| stats count by priority, severity, alert_type
| chart count over priority by severity
```

### Alert Trends Over Time
```spl
index=vmware sourcetype=vmware:perf:vm OR index=application sourcetype=application:logs
| timechart span=1h count as alerts by alert_type
| fields - NULL
```

### False Positive Rate
```spl
index=vmware sourcetype=vmware:perf:vm OR index=application sourcetype=application:logs
| stats count as total, count(eval(false_positive="true")) as false_positives by alert_type
| eval fp_rate=round((false_positives/total)*100, 2)
| table alert_type, total, false_positives, fp_rate
```

### Alert Resolution Time
```spl
index=vmware sourcetype=vmware:perf:vm OR index=application sourcetype=application:logs
| stats avg(resolution_time) as avg_resolution by alert_type, severity
| table alert_type, severity, avg_resolution
```

### Team Assignment Status
```spl
index=vmware sourcetype=vmware:perf:vm OR index=application sourcetype=application:logs
| stats count as total, count(eval(status="open")) as open, count(eval(status="in_progress")) as in_progress by team, assignee
| table team, assignee, total, open, in_progress
```

### Alert Escalation Status
```spl
index=vmware sourcetype=vmware:perf:vm OR index=application sourcetype=application:logs
| stats count by alert_id, current_severity, escalation_level, escalation_time
| where escalation_level > 1
| table alert_id, current_severity, escalation_level, escalation_time, count
```

### Alert Impact Analysis
```spl
index=vmware sourcetype=vmware:perf:vm OR index=application sourcetype=application:logs
| stats count by alert_type, affected_users, affected_services, business_impact
| table alert_type, affected_users, affected_services, business_impact, count
```

## 12. Historical Health Trends Dashboard

### VM Health Trends Over Time
```spl
index=vmware sourcetype=vmware:perf:vm
| timechart span=1d avg(cpu_usage) as avg_cpu, avg(mem_usage) as avg_mem by vm_name
| fields - NULL
```

### Application Health Trends
```spl
index=application sourcetype=healthcheck OR sourcetype=apm
| timechart span=1d avg(health_score) as avg_health by application
| fields - NULL
```

### Performance Degradation Patterns
```spl
index=vmware sourcetype=vmware:perf:vm OR index=application sourcetype=application:logs
| timechart span=1w avg(performance_score) as avg_score by resource_type
| fields - NULL
```

### Availability Trends
```spl
index=vmware sourcetype=vmware:inv:vm OR index=application sourcetype=uptime:monitoring
| timechart span=1w avg(availability_pct) as avg_avail by resource_type
| fields - NULL
```

### Error Rate Trends
```spl
index=application sourcetype=access_combined
| timechart span=1w count(eval(status >= 400)) as errors, count as total by application
| eval error_rate=round((errors/total)*100, 2)
| fields - NULL
```

### Resource Utilization Trends
```spl
index=vmware sourcetype=vmware:perf:vm OR index=application sourcetype=application:logs
| timechart span=1w avg(cpu_usage) as avg_cpu, avg(mem_usage) as avg_mem by resource_type
| fields - NULL
```

### Incident Frequency Analysis
```spl
index=incident sourcetype=vmware OR sourcetype=application
| timechart span=1w count as incidents by severity
| fields - NULL
```

### MTTR Trends
```spl
index=incident sourcetype=vmware OR sourcetype=application
| timechart span=1w avg(resolution_time) as avg_mttr by resource_type
| fields - NULL
```

### SLA Compliance Trends
```spl
index=sla sourcetype=vmware OR sourcetype=application
| timechart span=1w avg(compliance_pct) as avg_compliance by resource_type
| fields - NULL
```

### Predictive Health Indicators
```spl
index=vmware sourcetype=vmware:perf:vm OR index=application sourcetype=application:logs
| timechart span=1d avg(health_score) as avg_health by resource_type
| trendline ema5(avg_health) as forecast
| fields - NULL
```
