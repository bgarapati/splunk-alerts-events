# Infrastructure Alerts

## Server/VM Health
- CPU utilization > 90% for 5+ minutes
- Memory utilization > 90% for 5+ minutes
- Disk space < 10% remaining
- Disk I/O latency high
- Server down/unreachable
- Server reboot without schedule
- Hardware failures (disk, memory, CPU)
- Temperature warnings
- Power supply issues
- Fan failures
- Server performance degradation

## Network Infrastructure
- Network device down (router, switch, firewall)
- High bandwidth utilization (> 80%)
- Network latency spikes
- Packet loss detection
- Interface errors (CRC, collisions)
- BGP/OSPF routing issues
- DNS server failures
- DHCP server issues
- VPN connection failures
- Load balancer health issues

## Storage Systems
- Storage array performance degradation
- Storage controller failures
- Disk drive failures (RAID)
- Storage capacity thresholds
- SAN fabric issues
- NAS device issues
- Backup storage failures
- Replication lag
- Deduplication issues
- Storage cache issues

## Database Health
- Database connection failures
- Database server down
- Query performance degradation
- Slow query alerts (> 10 seconds)
- Database locking issues
- Transaction log full
- Database replication lag
- Database backup failures
- Database corruption detection
- Resource contention (CPU, memory, I/O)

## Virtualization
- Host server down
- VM resource exhaustion
- vMotion/migration failures
- Hyper-V cluster issues
- VMware ESXi host issues
- Virtual network issues
- Storage vMotion failures
- DRS (Distributed Resource Scheduler) issues
- HA (High Availability) failures
- VM snapshot issues

## Cloud Infrastructure (AWS/Azure/GCP)
- EC2/VM instance termination
- Auto-scaling group issues
- ELB/ALB health check failures
- RDS database issues
- S3 access errors
- Lambda function failures
- EKS/AKS cluster issues
- CloudWatch/Azure Monitor alerts
- Cost anomaly detection
- Resource quota exceeded

## Container & Orchestration
- Kubernetes node not ready
- Pod crash loop back off
- Container resource limits exceeded
- Docker daemon failures
- Kubernetes API server down
- Helm deployment failures
- Image pull errors
- Pod eviction events
- Service endpoint issues
- Ingress controller failures

## Backup & Recovery
- Backup job failures
- Backup job running too long
- RPO (Recovery Point Objective) violations
- RTO (Recovery Time Objective) violations
- Backup storage full
- Restore job failures
- Backup validation failures
- Offsite backup sync issues
- Backup encryption failures
- Retention policy violations

## Power & Environmental
- UPS battery low
- Power failure detection
- Generator activation
- Cooling system failure
- Temperature/humidity thresholds
- Water leak detection
- Fire suppression system activation
- PDU (Power Distribution Unit) issues
- Environmental sensor alerts
- Physical security breaches

## License & Capacity
- License expiration warnings
- License utilization thresholds
- Capacity planning alerts
- Resource pool exhaustion
- Virtual machine limits reached
- Storage quota exceeded
- Network bandwidth limits
- Compute quota exceeded
- API rate limits
- Tenant capacity limits
