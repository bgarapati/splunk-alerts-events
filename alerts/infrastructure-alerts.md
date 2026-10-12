# Infrastructure Alerts - SPL Queries

## Server/VM Health

### CPU Utilization > 90% for 5+ Minutes
```spl
index=os sourcetype=cpu OR sourcetype="vmware:perf:cpu"
| timechart span=5m avg(pct_cpu) as avg_cpu by host
| where avg_cpu > 90
| eval severity="critical"
| table _time, host, avg_cpu, severity
```

### Memory Utilization > 90% for 5+ Minutes
```spl
index=os sourcetype=mem OR sourcetype="vmware:perf:mem"
| timechart span=5m avg(pct_mem) as avg_mem by host
| where avg_mem > 90
| eval severity="critical"
| table _time, host, avg_mem, severity
```

### Disk Space < 10% Remaining
```spl
index=os sourcetype=df OR sourcetype="vmware:perf:disk"
| eval free_pct=round((capacity_used/total_capacity)*100, 2)
| where free_pct > 90
| table _time, host, mount_point, free_pct, capacity_used, total_capacity
| sort free_pct desc
```

### Disk I/O Latency High
```spl
index=os sourcetype=iostat OR sourcetype="vmware:perf:disk"
| timechart span=5m avg(wait_time) as avg_wait by host
| where avg_wait > 100
| eval severity="warning"
| table _time, host, avg_wait, severity
```

### Server Down/Unreachable
```spl
index=os sourcetype=ping OR sourcetype=updown
| stats count by host, status
| where status="down"
| table _time, host, status, count
| eval severity="critical"
```

### Hardware Failures (Disk, Memory, CPU)
```spl
index=hardware sourcetype=ipmi OR sourcetype=hp_ilo OR sourcetype=dell_om
| stats count by host, component, severity, message
| where severity IN (critical, major)
| table _time, host, component, severity, message, count
```

## Network Infrastructure

### Network Device Down (Router, Switch, Firewall)
```spl
index=network sourcetype=snmp OR sourcetype=pan:threat OR sourcetype=cisco:ios
| stats count by device_name, device_type, status
| where status="down"
| table _time, device_name, device_type, status, count
| eval severity="critical"
```

### High Bandwidth Utilization (> 80%)
```spl
index=netflow sourcetype=netflow OR sourcetype="cisco:asa"
| timechart span=5m sum(bytes) as total_bytes by interface
| eval bandwidth_pct=round((total_bytes / interface_capacity) * 100, 2)
| where bandwidth_pct > 80
| table _time, interface, bandwidth_pct, total_bytes
```

### Network Latency Spikes
```spl
index=network sourcetype=ping OR sourcetype=fping
| timechart span=5m avg(response_time) as avg_latency by dest
| where avg_latency > 100
| eval severity="warning"
| table _time, dest, avg_latency, severity
```

### Packet Loss Detection
```spl
index=network sourcetype=ping OR sourcetype=fping
| stats avg(packet_loss) as avg_loss by dest
| where avg_loss > 5
| table _time, dest, avg_loss
| eval severity="critical"
```

### BGP/OSPF Routing Issues
```spl
index=network sourcetype=cisco:ios OR sourcetype=juniper:junos
| stats count by router, protocol, status, message
| where status="down" OR message="neighbor_down"
| table _time, router, protocol, status, message, count
```

### DNS Server Failures
```spl
index=dns sourcetype=bind OR sourcetype=windows:dns
| stats count by dns_server, response_code, query_type
| where response_code="SERVFAIL" OR response_code="REFUSED"
| table _time, dns_server, response_code, query_type, count
```

## Storage Systems

### Storage Array Performance Degradation
```spl
index=storage sourcetype=emc:vxrail OR sourcetype=netapp:ontap
| timechart span=5m avg(response_time) as avg_response by array_name
| where avg_response > 50
| eval severity="warning"
| table _time, array_name, avg_response, severity
```

### Disk Drive Failures (RAID)
```spl
index=storage sourcetype=hp_msa OR sourcetype=dell_emc
| stats count by array_name, disk_id, raid_status, failure_type
| where raid_status="failed" OR raid_status="degraded"
| table _time, array_name, disk_id, raid_status, failure_type, count
| eval severity="critical"
```

### Storage Capacity Thresholds
```spl
index=storage sourcetype=netapp:ontap OR sourcetype=emc:vnx
| eval used_pct=round((used_capacity/total_capacity)*100, 2)
| where used_pct > 85
| table _time, array_name, volume, used_pct, used_capacity, total_capacity
| sort used_pct desc
```

### SAN Fabric Issues
```spl
index=storage sourcetype=brocade OR sourcetype=cisco_mds
| stats count by switch_name, port_status, error_type
| where port_status="offline" OR error_type="link_failure"
| table _time, switch_name, port_status, error_type, count
```

## Database Health

### Database Connection Failures
```spl
index=database sourcetype=mssql OR sourcetype=mysql OR sourcetype=oracle
| stats count by database_host, database_name, error_code, error_message
| where error_code IN (connection_refused, timeout, authentication_failed)
| table _time, database_host, database_name, error_code, error_message, count
```

### Database Server Down
```spl
index=database sourcetype=mssql OR sourcetype=mysql OR sourcetype=postgresql
| stats count by database_host, status
| where status="down"
| table _time, database_host, status, count
| eval severity="critical"
```

### Query Performance Degradation
```spl
index=database sourcetype=slowquery OR sourcetype=database:performance
| timechart span=5m avg(query_duration) as avg_duration by database_host
| where avg_duration > 10
| eval severity="warning"
| table _time, database_host, avg_duration, severity
```

### Slow Query Alerts (> 10 seconds)
```spl
index=database sourcetype=slowquery
| stats count by database_host, database_name, query_duration, query_hash
| where query_duration > 10
| table _time, database_host, database_name, query_duration, query_hash, count
| sort - query_duration
```

### Database Locking Issues
```spl
index=database sourcetype=mssql OR sourcetype=postgresql
| stats count by database_host, database_name, lock_type, wait_time
| where wait_time > 30
| table _time, database_host, database_name, lock_type, wait_time, count
```

### Transaction Log Full
```spl
index=database sourcetype=mssql OR sourcetype=oracle
| stats count by database_host, database_name, log_status, log_size
| where log_status="full" OR log_size > 10737418240
| table _time, database_host, database_name, log_status, log_size, count
| eval severity="critical"
```

## Virtualization

### Host Server Down
```spl
index=vmware sourcetype=vmware:perf:esxhost
| stats count by host_name, host_status, cluster_name
| where host_status="down" OR host_status="not responding"
| table _time, host_name, host_status, cluster_name, count
| eval severity="critical"
```

### VM Resource Exhaustion
```spl
index=vmware sourcetype=vmware:perf:vm
| stats avg(cpu_usage) as avg_cpu, avg(mem_usage) as avg_mem by vm_name, host_name
| where avg_cpu > 90 OR avg_mem > 90
| table _time, vm_name, host_name, avg_cpu, avg_mem
| eval severity="warning"
```

### vMotion/Migration Failures
```spl
index=vmware sourcetype=vmware:inv:vm
| stats count by vm_name, source_host, dest_host, migration_status
| where migration_status="failed"
| table _time, vm_name, source_host, dest_host, migration_status, count
```

### VMware ESXi Host Issues
```spl
index=vmware sourcetype=vmware:perf:esxhost
| stats count by host_name, issue_type, severity
| where issue_type IN (psod, purple_screen, hardware_issue)
| table _time, host_name, issue_type, severity, count
```

## Cloud Infrastructure (AWS/Azure/GCP)

### EC2/VM Instance Termination
```spl
index=aws sourcetype=aws:cloudtrail
| stats count by userIdentity.userName, eventName, instanceId, region
| where eventName="TerminateInstances"
| table _time, userIdentity.userName, eventName, instanceId, region, count
```

### Auto-scaling Group Issues
```spl
index=aws sourcetype=aws:cloudwatch
| stats count by AutoScalingGroupName, state, reason
| where state="unhealthy" OR reason="health_check_failed"
| table _time, AutoScalingGroupName, state, reason, count
```

### ELB/ALB Health Check Failures
```spl
index=aws sourcetype=aws:cloudwatch
| stats count by LoadBalancerName, TargetId, health_status
| where health_status="unhealthy"
| table _time, LoadBalancerName, TargetId, health_status, count
```

### RDS Database Issues
```spl
index=aws sourcetype=aws:cloudwatch:metrics
| stats avg(CPUUtilization) as avg_cpu, avg(DatabaseConnections) as avg_conn by DBInstanceIdentifier
| where avg_cpu > 90 OR avg_conn > 500
| table _time, DBInstanceIdentifier, avg_cpu, avg_conn
```

### S3 Access Errors
```spl
index=aws sourcetype=aws:s3:accesslogs
| stats count by bucket, operation, http_status, error_code
| where http_status >= 400
| table _time, bucket, operation, http_status, error_code, count
```

## Container & Orchestration

### Kubernetes Node Not Ready
```spl
index=kubernetes sourcetype=kube:objects
| stats count by node_name, status, condition
| where status="NotReady" OR condition="Ready" AND status="False"
| table _time, node_name, status, condition, count
| eval severity="critical"
```

### Pod Crash Loop Back Off
```spl
index=kubernetes sourcetype=kube:objects
| stats count by pod_name, namespace, phase, reason
| where phase="Failed" AND reason="CrashLoopBackOff"
| table _time, pod_name, namespace, phase, reason, count
```

### Container Resource Limits Exceeded
```spl
index=kubernetes sourcetype=kube:metrics
| stats avg(cpu_usage) as avg_cpu, avg(memory_usage) as avg_mem by pod_name, container_name
| where avg_cpu > 80 OR avg_mem > 80
| table _time, pod_name, container_name, avg_cpu, avg_mem
```

### Docker Daemon Failures
```spl
index=docker sourcetype=docker:logs
| stats count by host, docker_status, error_message
| where docker_status="stopped" OR error_message="daemon_not_running"
| table _time, host, docker_status, error_message, count
| eval severity="critical"
```

## Backup & Recovery

### Backup Job Failures
```spl
index=backup sourcetype=vbr OR sourcetype=veeam OR sourcetype=comvault
| stats count by job_name, job_status, backup_server
| where job_status="Failed" OR job_status="Warning"
| table _time, job_name, job_status, backup_server, count
| eval severity="critical"
```

### Backup Job Running Too Long
```spl
index=backup sourcetype=vbr OR sourcetype=veeam
| eval job_duration=now-_time
| where job_duration > 3600
| table _time, job_name, job_duration, job_status
| eval severity="warning"
```

### RPO (Recovery Point Objective) Violations
```spl
index=backup sourcetype=vbr OR sourcetype=veeam
| stats count by job_name, last_backup_time, rpo_threshold
| eval time_since_backup=now-last_backup_time
| where time_since_backup > rpo_threshold
| table _time, job_name, last_backup_time, time_since_backup, rpo_threshold
| eval severity="critical"
```

### Backup Storage Full
```spl
index=backup sourcetype=vbr OR sourcetype=veeam
| stats avg(used_capacity) as avg_used, max(used_capacity) as max_used by backup_repository
| eval capacity_pct=round((max_used/total_capacity)*100, 2)
| where capacity_pct > 90
| table _time, backup_repository, capacity_pct, max_used, total_capacity
```

## Power & Environmental

### UPS Battery Low
```spl
index=ipmi sourcetype=apc OR sourcetype=tripplite
| stats count by device_name, battery_status, battery_level
| where battery_status="low" OR battery_level < 20
| table _time, device_name, battery_status, battery_level, count
| eval severity="critical"
```

### Temperature Warnings
```spl
index=ipmi sourcetype=hp_ilo OR sourcetype=dell_om
| stats avg(temperature) as avg_temp by device_name, sensor_location
| where avg_temp > 35
| table _time, device_name, sensor_location, avg_temp
| eval severity="warning"
```

### Cooling System Failure
```spl
index=ipmi sourcetype=environmental
| stats count by device_name, component, status
| where component="cooling" AND status="failed"
| table _time, device_name, component, status, count
| eval severity="critical"
```

## License & Capacity

### License Expiration Warnings
```spl
index=license sourcetype=license_manager
| stats count by software_name, license_type, expiration_date, days_remaining
| where days_remaining < 30
| table _time, software_name, license_type, expiration_date, days_remaining
| eval severity=if(days_remaining < 7, "critical", "warning")
```

### Capacity Threshold Breaches
```spl
index=capacity sourcetype=capacity_planning
| stats avg(utilization) as avg_util by resource_type, resource_name
| where avg_util > 85
| table _time, resource_type, resource_name, avg_util
| eval severity="warning"
```
