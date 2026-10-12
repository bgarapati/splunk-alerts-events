# Infrastructure Dashboards - SPL Queries

## 1. Infrastructure Overview Dashboard

### Overall Infrastructure Health
```spl
index=infrastructure
| stats count as total, count(eval(status="healthy")) as healthy, count(eval(status="unhealthy")) as unhealthy by category
| eval health_pct=round((healthy/total)*100, 2)
| table category, total, healthy, unhealthy, health_pct
```

### Critical Alerts Summary
```spl
index=infrastructure severity IN (critical, high)
| stats count by category, severity, alert_name
| table category, severity, alert_name, count
| sort - severity
```

### Active Incidents Count
```spl
index=incident sourcetype=infrastructure
| stats count as total, count(eval(status="open")) as open, count(eval(status="in_progress")) as in_progress
| table total, open, in_progress
```

### System Availability Percentage
```spl
index=infrastructure sourcetype=uptime
| stats avg(uptime_pct) as avg_uptime by system_type
| table system_type, avg_uptime
```

### Resource Utilization Trends
```spl
index=os sourcetype=cpu OR sourcetype=mem
| timechart span=1h avg(pct_cpu) as avg_cpu, avg(pct_mem) as avg_mem
| fields - NULL
```

### Upcoming Maintenance Windows
```spl
index=maintenance sourcetype=maintenance:calendar
| stats count by system, maintenance_type, start_time, end_time
| where start_time > now AND start_time < now+86400
| table system, maintenance_type, start_time, end_time, count
```

### SLA Compliance Status
```spl
index=sla sourcetype=infrastructure
| stats count as total, count(eval(status="compliant")) as compliant, count(eval(status="non_compliant")) as non_compliant by service
| eval compliance_pct=round((compliant/total)*100, 2)
| table service, total, compliant, non_compliant, compliance_pct
```

### Capacity Utilization Overview
```spl
index=capacity sourcetype=capacity_planning
| stats avg(utilization) as avg_util by resource_type, resource_name
| table resource_type, resource_name, avg_util
| sort - avg_util
```

### Infrastructure Risk Score
```spl
index=infrastructure
| stats count as total_events, count(eval(severity="critical")) as critical, count(eval(severity="high")) as high
| eval risk_score=round(100 - ((critical*10 + high*5) / total_events * 100), 2)
| table total_events, critical, high, risk_score
```

### Recent Changes Summary
```spl
index=audit sourcetype=configuration
| stats count by system, change_type, user, change_time
| where change_time > now-86400
| table system, change_type, user, change_time, count
| sort - change_time
```

## 2. Server/VM Health Dashboard

### Server Availability Status
```spl
index=os sourcetype=ping OR sourcetype=updown
| stats count by host, status
| table host, status, count
```

### CPU Utilization Trends
```spl
index=os sourcetype=cpu
| timechart span=5m avg(pct_cpu) as avg_cpu by host
| fields - NULL
```

### Memory Utilization Trends
```spl
index=os sourcetype=mem
| timechart span=5m avg(pct_mem) as avg_mem by host
| fields - NULL
```

### Disk Space Usage
```spl
index=os sourcetype=df
| stats avg(used_pct) as avg_used, max(used_pct) as max_used by host, mount_point
| table host, mount_point, avg_used, max_used
| sort - max_used
```

### Disk I/O Performance
```spl
index=os sourcetype=iostat
| stats avg(read_bytes) as avg_read, avg(write_bytes) as avg_write, avg(wait_time) as avg_wait by host
| table host, avg_read, avg_write, avg_wait
```

### Server Temperature Monitoring
```spl
index=ipmi sourcetype=temperature
| stats avg(temperature) as avg_temp, max(temperature) as max_temp by host, sensor_location
| table host, sensor_location, avg_temp, max_temp
```

### Hardware Failure Alerts
```spl
index=hardware sourcetype=ipmi OR sourcetype=hp_ilo OR sourcetype=dell_om
| stats count by host, component, severity, message
| where severity IN (critical, major)
| table host, component, severity, message, count
```

### Server Performance Score
```spl
index=os sourcetype=cpu OR sourcetype=mem OR sourcetype=iostat
| stats avg(pct_cpu) as avg_cpu, avg(pct_mem) as avg_mem, avg(wait_time) as avg_wait by host
| eval performance_score=round(100 - (avg_cpu + avg_mem + avg_wait)/3, 2)
| table host, avg_cpu, avg_mem, avg_wait, performance_score
| sort performance_score
```

### VM Resource Allocation
```spl
index=vmware sourcetype=vmware:perf:vm
| stats avg(cpu_usage) as avg_cpu, avg(mem_usage) as avg_mem, avg(disk_usage) as avg_disk by vm_name, host_name
| table vm_name, host_name, avg_cpu, avg_mem, avg_disk
```

### Host vs Guest Metrics
```spl
index=vmware sourcetype=vmware:perf:esxhost OR sourcetype=vmware:perf:vm
| stats avg(cpu_usage) as avg_cpu, avg(mem_usage) as avg_mem by host_name, resource_type
| table host_name, resource_type, avg_cpu, avg_mem
```

## 3. Network Infrastructure Dashboard

### Network Device Status
```spl
index=network sourcetype=snmp OR sourcetype=cisco:ios
| stats count by device_name, device_type, status
| table device_name, device_type, status, count
```

### Bandwidth Utilization
```spl
index=netflow sourcetype=netflow
| timechart span=5m sum(bytes) as total_bytes by interface
| eval bandwidth_pct=round((total_bytes / interface_capacity) * 100, 2)
| table _time, interface, bandwidth_pct, total_bytes
```

### Network Latency Metrics
```spl
index=network sourcetype=ping OR sourcetype=fping
| stats avg(response_time) as avg_latency, max(response_time) as max_latency by dest
| table dest, avg_latency, max_latency
```

### Packet Loss Statistics
```spl
index=network sourcetype=ping OR sourcetype=fping
| stats avg(packet_loss) as avg_loss, max(packet_loss) as max_loss by dest
| table dest, avg_loss, max_loss
```

### Interface Error Rates
```spl
index=network sourcetype=snmp
| stats count as total, count(eval(error_type="crc")) as crc, count(eval(error_type="collision")) as collision by interface
| eval error_rate=round(((crc + collision) / total) * 100, 2)
| table interface, total, crc, collision, error_rate
```

### Routing Protocol Status
```spl
index=network sourcetype=cisco:ios OR sourcetype=juniper:junos
| stats count by router, protocol, status, neighbor
| table router, protocol, status, neighbor, count
```

### Top Talkers (by IP/Protocol)
```spl
index=netflow sourcetype=netflow
| stats sum(bytes) as total_bytes by src_ip, dest_ip, protocol
| eval total_mb=round(total_bytes/1048576, 2)
| table src_ip, dest_ip, protocol, total_mb
| sort - total_mb
| head 20
```

### VPN Connection Status
```spl
index=vpn sourcetype=cisco_anyconnect OR sourcetype=openvpn
| stats count by user, vpn_server, connection_status, session_duration
| table user, vpn_server, connection_status, session_duration, count
```

### Load Balancer Health
```spl
index=loadbalancer sourcetype=f5:bigip OR sourcetype=aws:elb
| stats count by pool_name, member_ip, health_status
| table pool_name, member_ip, health_status, count
```

### Network Topology View
```spl
index=network sourcetype=snmp
| stats count by device_name, device_type, connected_to
| table device_name, device_type, connected_to, count
```

## 4. Storage Infrastructure Dashboard

### Storage Array Health
```spl
index=storage sourcetype=emc:vxrail OR sourcetype=netapp:ontap
| stats count by array_name, health_status, component_type
| table array_name, health_status, component_type, count
```

### Storage Capacity Utilization
```spl
index=storage sourcetype=netapp:ontap OR sourcetype=emc:vnx
| stats avg(used_pct) as avg_used, max(used_pct) as max_used by array_name, volume
| table array_name, volume, avg_used, max_used
| sort - max_used
```

### I/O Performance Metrics
```spl
index=storage sourcetype=netapp:ontap OR sourcetype=emc:vnx
| stats avg(read_iops) as avg_read_iops, avg(write_iops) as avg_write_iops, avg(response_time) as avg_response by array_name
| table array_name, avg_read_iops, avg_write_iops, avg_response
```

### Disk Drive Status (RAID)
```spl
index=storage sourcetype=hp_msa OR sourcetype=dell_emc
| stats count by array_name, disk_id, raid_status, failure_type
| table array_name, disk_id, raid_status, failure_type, count
```

### Storage Controller Status
```spl
index=storage sourcetype=netapp:ontap OR sourcetype=emc:vnx
| stats count by array_name, controller_id, controller_status
| table array_name, controller_id, controller_status, count
```

### SAN Fabric Health
```spl
index=storage sourcetype=brocade OR sourcetype=cisco_mds
| stats count by switch_name, port_status, port_id, error_type
| table switch_name, port_status, port_id, error_type, count
```

### NAS Device Performance
```pl
index=storage sourcetype=netapp:nas OR sourcetype=emc:nas
| stats avg(read_throughput) as avg_read, avg(write_throughput) as avg_write by nas_device
| table nas_device, avg_read, avg_write
```

### Backup Storage Status
```spl
index=backup sourcetype=vbr OR sourcetype=veeam
| stats count by backup_repository, repository_status, used_capacity, total_capacity
| eval used_pct=round((used_capacity/total_capacity)*100, 2)
| table backup_repository, repository_status, used_capacity, total_capacity, used_pct
```

### Replication Lag Metrics
```spl
index=storage sourcetype=netapp:snapmirror OR sourcetype=emc:replication
| stats avg(lag_time) as avg_lag, max(lag_time) as max_lag by source_volume, dest_volume
| table source_volume, dest_volume, avg_lag, max_lag
```

### Storage Tiering Overview
```spl
index=storage sourcetype=netapp:ontap OR sourcetype=emc:vnx
| stats count by tier, volume, data_size
| table tier, volume, data_size, count
```

## 5. Database Infrastructure Dashboard

### Database Server Status
```spl
index=database sourcetype=mssql OR sourcetype=mysql OR sourcetype=postgresql
| stats count by database_host, database_name, status
| table database_host, database_name, status, count
```

### Connection Pool Utilization
```spl
index=database sourcetype=connection_pool
| stats avg(active_connections) as avg_active, max(max_connections) as max_conn by database_host
| eval utilization_pct=round((avg_active/max_conn)*100, 2)
| table database_host, avg_active, max_conn, utilization_pct
```

### Query Performance Metrics
```spl
index=database sourcetype=slowquery OR sourcetype=database:performance
| stats avg(query_duration) as avg_duration, max(query_duration) as max_duration by database_host, query_type
| table database_host, query_type, avg_duration, max_duration
```

### Slow Query Analysis
```spl
index=database sourcetype=slowquery
| stats count by database_host, database_name, query_duration, query_hash
| where query_duration > 5
| table database_host, database_name, query_duration, query_hash, count
| sort - query_duration
```

### Database Size Trends
```spl
index=database sourcetype=database:metrics
| timechart span=1d avg(database_size) as avg_size by database_name
| fields - NULL
```

### Transaction Log Status
```spl
index=database sourcetype=mssql OR sourcetype=oracle
| stats count by database_host, database_name, log_status, log_size
| where log_status="full" OR log_size > 10737418240
| table database_host, database_name, log_status, log_size, count
```

### Replication Health
```spl
index=database sourcetype=replication
| stats count by primary_host, replica_host, replication_status, lag_time
| table primary_host, replica_host, replication_status, lag_time, count
```

### Backup Status
```spl
index=database sourcetype=backup
| stats count by database_host, database_name, backup_status, backup_time
| table database_host, database_name, backup_status, backup_time, count
```

### Resource Contention
```spl
index=database sourcetype=database:performance
| stats avg(cpu_usage) as avg_cpu, avg(mem_usage) as avg_mem, avg(io_wait) as avg_io by database_host
| table database_host, avg_cpu, avg_mem, avg_io
```

### Database Performance Score
```spl
index=database sourcetype=database:performance
| stats avg(query_duration) as avg_query, avg(cpu_usage) as avg_cpu, avg(io_wait) as avg_io by database_host
| eval performance_score=round(100 - (avg_query*10 + avg_cpu + avg_io)/3, 2)
| table database_host, avg_query, avg_cpu, avg_io, performance_score
```

## 6. Virtualization Dashboard

### Host Server Health
```spl
index=vmware sourcetype=vmware:perf:esxhost
| stats count by host_name, host_status, cluster_name
| table host_name, host_status, cluster_name, count
```

### VM Resource Utilization
```spl
index=vmware sourcetype=vmware:perf:vm
| stats avg(cpu_usage) as avg_cpu, avg(mem_usage) as avg_mem by vm_name, host_name
| table vm_name, host_name, avg_cpu, avg_mem
```

### vMotion/Migration Activity
```spl
index=vmware sourcetype=vmware:inv:vm
| stats count by vm_name, source_host, dest_host, migration_status, migration_time
| table vm_name, source_host, dest_host, migration_status, migration_time, count
```

### Cluster Resource Usage
```spl
index=vmware sourcetype=vmware:perf:cluster
| stats avg(cpu_usage) as avg_cpu, avg(mem_usage) as avg_mem by cluster_name
| table cluster_name, avg_cpu, avg_mem
```

### HA (High Availability) Status
```spl
index=vmware sourcetype=vmware:perf:cluster
| stats count by cluster_name, ha_status, ha_enabled
| table cluster_name, ha_status, ha_enabled, count
```

### DRS (Distributed Resource Scheduler) Status
```spl
index=vmware sourcetype=vmware:perf:cluster
| stats count by cluster_name, drs_status, drs_mode
| table cluster_name, drs_status, drs_mode, count
```

### VM Snapshot Status
```spl
index=vmware sourcetype=vmware:inv:vm
| stats count by vm_name, snapshot_count, snapshot_age, snapshot_size
| table vm_name, snapshot_count, snapshot_age, snapshot_size, count
```

### Virtual Network Performance
```spl
index=vmware sourcetype=vmware:perf:network
| stats avg(network_in) as avg_in, avg(network_out) as avg_out by vm_name, portgroup
| table vm_name, portgroup, avg_in, avg_out
```

### Storage vMotion Activity
```spl
index=vmware sourcetype=vmware:inv:vm
| stats count by vm_name, source_datastore, dest_datastore, svmotion_status
| table vm_name, source_datastore, dest_datastore, svmotion_status, count
```

### Capacity Planning Metrics
```spl
index=vmware sourcetype=vmware:perf:cluster
| stats avg(cpu_usage) as avg_cpu, avg(mem_usage) as avg_mem, avg(storage_usage) as avg_storage by cluster_name
| table cluster_name, avg_cpu, avg_mem, avg_storage
```

## 7. Cloud Infrastructure Dashboard

### Cloud Resource Inventory
```spl
index=aws sourcetype=aws:ec2 OR sourcetype=aws:cloudtrail
| stats count by resource_type, resource_id, region, state
| table resource_type, resource_id, region, state, count
```

### Instance Health Status
```spl
index=aws sourcetype=aws:cloudwatch
| stats count by instance_id, instance_state, health_status
| table instance_id, instance_state, health_status, count
```

### Auto-scaling Group Status
```spl
index=aws sourcetype=aws:cloudwatch
| stats count by AutoScalingGroupName, desired_capacity, current_capacity, status
| table AutoScalingGroupName, desired_capacity, current_capacity, status, count
```

### Cost Overview by Service
```spl
index=aws sourcetype=aws:cost
| stats sum(cost) as total_cost by service, region
| eval cost_usd=round(total_cost, 2)
| table service, region, cost_usd
| sort - cost_usd
```

### Resource Utilization Trends
```spl
index=aws sourcetype=aws:cloudwatch
| timechart span=1h avg(CPUUtilization) as avg_cpu by instance_id
| fields - NULL
```

### Cloud Service Dependencies
```spl
index=aws sourcetype=aws:cloudtrail
| stats count by service, dependent_service, dependency_type
| table service, dependent_service, dependency_type, count
```

### Security Group Status
```spl
index=aws sourcetype=aws:ec2
| stats count by security_group_id, rule_count, inbound_rules, outbound_rules
| table security_group_id, rule_count, inbound_rules, outbound_rules, count
```

### IAM Role Usage
```spl
index=aws sourcetype=aws:iam
| stats count by role_name, attached_policies, last_used_date
| table role_name, attached_policies, last_used_date, count
```

### Region-wise Distribution
```spl
index=aws sourcetype=aws:cloudtrail
| stats count by region, service, resource_type
| table region, service, resource_type, count
```

### Reserved Instance Utilization
```spl
index=aws sourcetype=aws:cost
| stats sum(reserved_hours) as reserved, sum(used_hours) as used by instance_type
| eval utilization_pct=round((used/reserved)*100, 2)
| table instance_type, reserved, used, utilization_pct
```

## 8. Container & Orchestration Dashboard

### Kubernetes Cluster Health
```spl
index=kubernetes sourcetype=kube:objects
| stats count by cluster_name, component, health_status
| table cluster_name, component, health_status, count
```

### Node Status Overview
```spl
index=kubernetes sourcetype=kube:objects
| stats count by node_name, status, condition, reason
| table node_name, status, condition, reason, count
```

### Pod Health Distribution
```spl
index=kubernetes sourcetype=kube:objects
| stats count by namespace, pod_name, phase, status
| table namespace, pod_name, phase, status, count
```

### Container Resource Usage
```spl
index=kubernetes sourcetype=kube:metrics
| stats avg(cpu_usage) as avg_cpu, avg(memory_usage) as avg_mem by pod_name, container_name
| table pod_name, container_name, avg_cpu, avg_mem
```

### Deployment Status
```spl
index=kubernetes sourcetype=kube:objects
| stats count by deployment_name, namespace, replicas, available_replicas, status
| table deployment_name, namespace, replicas, available_replicas, status, count
```

### Service Endpoint Health
```spl
index=kubernetes sourcetype=kube:objects
| stats count by service_name, namespace, endpoint_type, endpoint_status
| table service_name, namespace, endpoint_type, endpoint_status, count
```

### Ingress Controller Status
```spl
index=kubernetes sourcetype=kube:objects
| stats count by ingress_name, namespace, backend_status, health_status
| table ingress_name, namespace, backend_status, health_status, count
```

### Helm Release Status
```spl
index=kubernetes sourcetype=helm
| stats count by release_name, namespace, chart, status
| table release_name, namespace, chart, status, count
```

### Image Registry Status
```spl
index=kubernetes sourcetype=container_registry
| stats count by image_name, tag, registry, pull_count
| table image_name, tag, registry, pull_count
```

### Namespace Utilization
```spl
index=kubernetes sourcetype=kube:metrics
| stats avg(cpu_usage) as avg_cpu, avg(memory_usage) as avg_mem by namespace
| table namespace, avg_cpu, avg_mem
```

## 9. Backup & Recovery Dashboard

### Backup Job Status
```spl
index=backup sourcetype=vbr OR sourcetype=veeam OR sourcetype=comvault
| stats count by job_name, job_status, backup_server, job_type
| table job_name, job_status, backup_server, job_type, count
```

### Backup Success Rate
```spl
index=backup sourcetype=vbr OR sourcetype=veeam
| stats count as total, count(eval(job_status="Success")) as success, count(eval(job_status="Failed")) as failed by job_name
| eval success_rate=round((success/total)*100, 2)
| table job_name, total, success, failed, success_rate
```

### RPO (Recovery Point Objective) Compliance
```spl
index=backup sourcetype=vbr OR sourcetype=veeam
| stats count by job_name, last_backup_time, rpo_threshold, rpo_status
| eval time_since_backup=now-last_backup_time
| where time_since_backup > rpo_threshold
| table job_name, last_backup_time, time_since_backup, rpo_threshold, rpo_status, count
```

### RTO (Recovery Time Objective) Compliance
```spl
index=backup sourcetype=vbr OR sourcetype=veeam
| stats count by job_name, restore_time, rto_threshold, rto_status
| where restore_time > rto_threshold
| table job_name, restore_time, rto_threshold, rto_status, count
```

### Backup Storage Utilization
```spl
index=backup sourcetype=vbr OR sourcetype=veeam
| stats avg(used_capacity) as avg_used, max(used_capacity) as max_used by backup_repository
| eval capacity_pct=round((max_used/total_capacity)*100, 2)
| table backup_repository, avg_used, max_used, capacity_pct
```

### Restore Job Status
```spl
index=backup sourcetype=vbr OR sourcetype=veeam
| stats count by restore_job_name, restore_status, restore_time, restore_size
| table restore_job_name, restore_status, restore_time, restore_size, count
```

### Replication Status
```spl
index=backup sourcetype=vbr OR sourcetype=veeam
| stats count by replication_job, source_repo, dest_repo, replication_status, lag_time
| table replication_job, source_repo, dest_repo, replication_status, lag_time, count
```

### Backup Retention Compliance
```spl
index=backup sourcetype=vbr OR sourcetype=veeam
| stats count by job_name, retention_policy, retention_status, backup_age
| table job_name, retention_policy, retention_status, backup_age, count
```

### Offsite Backup Sync
```spl
index=backup sourcetype=vbr OR sourcetype=veeam
| stats count by offsite_job, sync_status, last_sync_time, sync_size
| table offsite_job, sync_status, last_sync_time, sync_size, count
```

### Backup Window Analysis
```spl
index=backup sourcetype=vbr OR sourcetype=veeam
| stats avg(job_duration) as avg_duration, max(job_duration) as max_duration by job_name, backup_window
| table job_name, backup_window, avg_duration, max_duration
```

## 10. Environmental & Power Dashboard

### Power Supply Status
```spl
index=ipmi sourcetype=power
| stats count by device_name, power_supply_id, power_status, power_load
| table device_name, power_supply_id, power_status, power_load, count
```

### UPS Battery Health
```spl
index=ipmi sourcetype=apc OR sourcetype=tripplite
| stats count by device_name, battery_status, battery_level, battery_age
| table device_name, battery_status, battery_level, battery_age, count
```

### Generator Status
```spl
index=ipmi sourcetype=generator
| stats count by device_name, generator_status, fuel_level, last_test
| table device_name, generator_status, fuel_level, last_test, count
```

### Temperature Monitoring
```spl
index=ipmi sourcetype=temperature
| stats avg(temperature) as avg_temp, max(temperature) as max_temp by device_name, sensor_location
| table device_name, sensor_location, avg_temp, max_temp
```

### Humidity Levels
```spl
index=ipmi sourcetype=humidity
| stats avg(humidity) as avg_humidity, max(humidity) as max_humidity by device_name, sensor_location
| table device_name, sensor_location, avg_humidity, max_humidity
```

### Cooling System Status
```spl
index=ipmi sourcetype=cooling
| stats count by device_name, cooling_unit, cooling_status, temperature_setpoint
| table device_name, cooling_unit, cooling_status, temperature_setpoint, count
```

### PDU (Power Distribution Unit) Health
```spl
index=ipmi sourcetype=pdu
| stats count by device_name, pdu_id, pdu_status, load_percentage
| table device_name, pdu_id, pdu_status, load_percentage, count
```

### Fire Suppression Status
```spl
index=ipmi sourcetype=fire_suppression
| stats count by device_name, system_status, agent_status, last_inspection
| table device_name, system_status, agent_status, last_inspection, count
```

### Physical Security Status
```spl
index=ipmi sourcetype=physical_security
| stats count by device_name, door_status, access_control, last_access
| table device_name, door_status, access_control, last_access, count
```

### Environmental Alerts Summary
```spl
index=ipmi sourcetype=environmental
| stats count by device_name, alert_type, severity, alert_time
| table device_name, alert_type, severity, alert_time, count
| sort - severity
```

## 11. Capacity Planning Dashboard

### CPU Capacity Forecasting
```spl
index=os sourcetype=cpu
| timechart span=1d avg(pct_cpu) as avg_cpu by host
| trendline ema7(avg_cpu) as forecast
| where forecast > 85
| table _time, host, avg_cpu, forecast
```

### Memory Capacity Forecasting
```spl
index=os sourcetype=mem
| timechart span=1d avg(pct_mem) as avg_mem by host
| trendline ema7(avg_mem) as forecast
| where forecast > 85
| table _time, host, avg_mem, forecast
```

### Storage Capacity Forecasting
```spl
index=os sourcetype=df
| timechart span=1d avg(used_pct) as avg_used by host, mount_point
| trendline ema7(avg_used) as forecast
| where forecast > 85
| table _time, host, mount_point, avg_used, forecast
```

### Network Bandwidth Forecasting
```spl
index=netflow sourcetype=netflow
| timechart span=1d sum(bytes) as total_bytes by interface
| eval bandwidth_pct=round((total_bytes / interface_capacity) * 100, 2)
| trendline ema7(bandwidth_pct) as forecast
| where forecast > 80
| table _time, interface, bandwidth_pct, forecast
```

### Growth Trend Analysis
```spl
index=capacity sourcetype=capacity_planning
| timechart span=1w avg(utilization) as avg_util by resource_type
| fields - NULL
```

### Resource Pool Utilization
```spl
index=vmware sourcetype=vmware:perf:cluster
| stats avg(cpu_usage) as avg_cpu, avg(mem_usage) as avg_mem by cluster_name, resource_pool
| table cluster_name, resource_pool, avg_cpu, avg_mem
```

### Capacity Threshold Alerts
```spl
index=capacity sourcetype=capacity_planning
| stats avg(utilization) as avg_util by resource_type, resource_name
| where avg_util > 85
| table resource_type, resource_name, avg_util
```

### Right-sizing Recommendations
```spl
index=cloud sourcetype=aws:cloudwatch
| stats avg(cpu_usage) as avg_cpu, avg(memory_usage) as avg_mem by instance_id, instance_type
| where avg_cpu < 20 AND avg_mem < 20
| table instance_id, instance_type, avg_cpu, avg_mem, recommendation
| eval recommendation="downsize"
```

### Budget vs Actual Usage
```spl
index=cost sourcetype=aws:cost
| stats sum(actual_cost) as total_cost, sum(budget) as total_budget by department, month
| table department, month, total_cost, total_budget, variance
| eval variance=total_cost - total_budget
```

### Capacity Planning Horizon
```spl
index=capacity sourcetype=capacity_planning
| stats count by resource_type, current_utilization, projected_utilization, months_until_capacity
| table resource_type, current_utilization, projected_utilization, months_until_capacity
```

## 12. Hardware Lifecycle Dashboard

### Hardware Inventory
```spl
index=hardware sourcetype=inventory
| stats count by device_type, manufacturer, model, serial_number
| table device_type, manufacturer, model, serial_number, count
```

### Warranty Status
```spl
index=hardware sourcetype=warranty
| stats count by device_id, device_type, warranty_expiry, warranty_status
| eval days_remaining=round((warranty_expiry-now)/86400, 0)
| where days_remaining < 90
| table device_id, device_type, warranty_expiry, warranty_status, days_remaining, count
```

### End-of-life Tracking
```spl
index=hardware sourcetype=eol
| stats count by device_id, device_type, eol_date, eol_status
| eval days_until_eol=round((eol_date-now)/86400, 0)
| table device_id, device_type, eol_date, eol_status, days_until_eol, count
```

### Hardware Age Distribution
```spl
index=hardware sourcetype=inventory
| stats count by device_type, age_years, age_category
| eval age_years=round((now-purchase_date)/31536000, 1)
| eval age_category=if(age_years < 3, "new", if(age_years < 5, "middle", "old"))
| table device_type, age_years, age_category, count
```

### Replacement Planning
```spl
index=hardware sourcetype=eol OR sourcetype=warranty
| stats count by device_id, device_type, replacement_priority, recommended_replacement_date
| table device_id, device_type, replacement_priority, recommended_replacement_date, count
```

### Maintenance Schedule
```spl
index=maintenance sourcetype=hardware_maintenance
| stats count by device_id, device_type, maintenance_type, scheduled_date, status
| table device_id, device_type, maintenance_type, scheduled_date, status, count
```

### Hardware Performance Trends
```spl
index=hardware sourcetype=performance
| timechart span=1w avg(performance_score) as avg_score by device_type
| fields - NULL
```

### Failure Rate Analysis
```spl
index=hardware sourcetype=failure
| stats count as total_failures by device_type, failure_type, age_years
| eval failure_rate=round(total_failures / device_count * 100, 2)
| table device_type, failure_type, age_years, total_failures, failure_rate
```

### Vendor Support Status
```spl
index=hardware sourcetype=support
| stats count by vendor, support_contract, support_expiry, support_level
| table vendor, support_contract, support_expiry, support_level, count
```

### Asset Depreciation Tracking
```spl
index=hardware sourcetype=financial
| stats count by asset_id, device_type, purchase_cost, current_value, depreciation_pct
| table asset_id, device_type, purchase_cost, current_value, depreciation_pct, count
```
