# Application Dashboards - SPL Queries

## 1. Application Overview Dashboard

### Application Health Status
```spl
index=application sourcetype=healthcheck OR sourcetype=ping
| stats count by application, health_status, last_check
| table application, health_status, last_check, count
```

### Active Applications Count
```spl
index=application sourcetype=application:logs
| stats dc(application) as total_apps, count(eval(status="running")) as running, count(eval(status="stopped")) as stopped
| table total_apps, running, stopped
```

### Critical Application Alerts
```spl
index=application severity IN (critical, high)
| stats count by application, alert_name, severity
| table application, alert_name, severity, count
| sort - severity
```

### Application Availability Percentage
```spl
index=application sourcetype=uptime:monitoring
| stats avg(uptime_pct) as avg_uptime by application
| table application, avg_uptime
```

### Overall Error Rate
```spl
index=application sourcetype=access_combined OR sourcetype=nginx:access
| stats count as total, count(eval(status >= 400)) as errors by application
| eval error_rate=round((errors/total)*100, 2)
| table application, total, errors, error_rate
```

### Response Time Trends
```spl
index=application sourcetype=access_combined
| timechart span=5m avg(response_time) as avg_response by application
| fields - NULL
```

### Throughput Metrics
```spl
index=application sourcetype=access_combined
| timechart span=5m count as requests by application
| fields - NULL
```

### Active User Sessions
```spl
index=application sourcetype=session
| stats count as active_sessions by application, user
| table application, user, active_sessions
| sort - active_sessions
```

### SLA Compliance Status
```spl
index=sla sourcetype=application
| stats count as total, count(eval(status="compliant")) as compliant, count(eval(status="non_compliant")) as non_compliant by application
| eval compliance_pct=round((compliant/total)*100, 2)
| table application, total, compliant, non_compliant, compliance_pct
```

### Recent Deployment Activity
```spl
index=deploy sourcetype=deployment
| stats count by application, deployment_type, environment, deployment_time
| where deployment_time > now-86400
| table application, deployment_type, environment, deployment_time, count
| sort - deployment_time
```

## 2. Application Performance Dashboard

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

### Throughput (Requests Per Second)
```spl
index=application sourcetype=access_combined
| timechart span=1m count as rps by application
| fields - NULL
```

### Apdex Score (Application Performance Index)
```spl
index=application sourcetype=apm
| stats count as total, count(eval(response_time < 2)) as satisfied, count(eval(response_time >= 2 AND response_time < 4)) as tolerating, count(eval(response_time >= 4)) as frustrated by application
| eval apdex=round((satisfied + (tolerating/2)) / total, 2)
| table application, total, satisfied, tolerating, frustrated, apdex
```

### Database Query Performance
```spl
index=database sourcetype=slowquery OR sourcetype=database:performance
| stats avg(query_duration) as avg_duration, max(query_duration) as max_duration by application, query_type
| table application, query_type, avg_duration, max_duration
```

### Cache Hit/Miss Ratios
```spl
index=application sourcetype=redis OR sourcetype=memcached
| stats sum(cache_hits) as hits, sum(cache_misses) as misses by application
| eval hit_rate=round((hits/(hits+misses))*100, 2), miss_rate=round((misses/(hits+misses))*100, 2)
| table application, hits, misses, hit_rate, miss_rate
```

### External Service Latency
```spl
index=application sourcetype=apm OR sourcetype=api:logs
| stats avg(response_time) as avg_latency by application, external_service
| table application, external_service, avg_latency
```

### Memory Usage Trends
```spl
index=application sourcetype=jmx OR sourcetype=metrics
| timechart span=5m avg(heap_memory) as avg_heap by application
| fields - NULL
```

### CPU Utilization
```spl
index=application sourcetype=jmx OR sourcetype=metrics
| timechart span=5m avg(cpu_usage) as avg_cpu by application
| fields - NULL
```

### Garbage Collection Metrics
```spl
index=application sourcetype=jmx OR sourcetype=java:gc
| stats avg(gc_duration) as avg_gc, max(gc_duration) as max_gc, sum(gc_count) as total_gc by application
| table application, avg_gc, max_gc, total_gc
```

## 3. Web Application Dashboard

### HTTP Status Code Distribution
```spl
index=web sourcetype=access_combined OR sourcetype=nginx:access
| stats count by application, status
| chart count over application by status
```

### Page Load Times
```spl
index=web sourcetype=rum OR sourcetype=webperf
| stats avg(page_load_time) as avg_load, max(page_load_time) as max_load by page_url
| table page_url, avg_load, max_load
| sort - avg_load
```

### User Session Metrics
```spl
index=web sourcetype=session OR sourcetype=rum
| stats avg(session_duration) as avg_duration, dc(user) as unique_users by application
| table application, avg_duration, unique_users
```

### Geographic User Distribution
```spl
index=web sourcetype=access_combined
| stats count by application, country
| geostats count
```

### Browser/Device Breakdown
```spl
index=web sourcetype=rum OR sourcetype=access_combined
| stats count by application, user_agent, device_type
| table application, user_agent, device_type, count
```

### Top Endpoints by Traffic
```spl
index=web sourcetype=access_combined
| stats count by application, endpoint, method
| table application, endpoint, method, count
| sort - count
| head 20
```

### Slowest Endpoints
```spl
index=web sourcetype=access_combined
| stats avg(response_time) as avg_response by application, endpoint
| table application, endpoint, avg_response
| sort - avg_response
| head 20
```

### Error Rate by Endpoint
```spl
index=web sourcetype=access_combined
| stats count as total, count(eval(status >= 400)) as errors by application, endpoint
| eval error_rate=round((errors/total)*100, 2)
| table application, endpoint, total, errors, error_rate
| sort - error_rate
```

### User Journey Analysis
```spl
index=web sourcetype=rum OR sourcetype=analytics
| stats count by user_id, page_sequence, session_id
| table user_id, page_sequence, session_id, count
```

### Conversion Funnels
```spl
index=web sourcetype=analytics OR sourcetype=rum
| stats count as total, count(eval(step="complete")) as completed by funnel_name, step
| eval conversion_rate=round((completed/total)*100, 2)
| table funnel_name, step, total, completed, conversion_rate
```

## 4. API Performance Dashboard

### API Endpoint Health
```spl
index=api sourcetype=api:logs
| stats count as total, count(eval(status >= 200 AND status < 300)) as success, count(eval(status >= 400)) as errors by endpoint
| eval success_rate=round((success/total)*100, 2), error_rate=round((errors/total)*100, 2)
| table endpoint, total, success, errors, success_rate, error_rate
```

### Request/Response Latency
```spl
index=api sourcetype=api:logs
| stats avg(response_time) as avg_latency, max(response_time) as max_latency by endpoint, method
| table endpoint, method, avg_latency, max_latency
```

### Error Rate by Endpoint
```spl
index=api sourcetype=api:logs
| timechart span=5m count(eval(status >= 400)) as errors, count as total by endpoint
| eval error_rate=round((errors/total)*100, 2)
| fields - NULL
```

### Rate Limit Utilization
```spl
index=api sourcetype=api:logs
| stats count as requests, max(rate_limit) as limit by api_key, endpoint
| eval utilization_pct=round((requests/limit)*100, 2)
| table api_key, endpoint, requests, limit, utilization_pct
```

### API Version Usage
```spl
index=api sourcetype=api:logs
| stats count by api_version, endpoint
| table api_version, endpoint, count
```

### Authentication Success/Failure
```spl
index=api sourcetype=api:logs
| stats count as total, count(eval(auth_status="success")) as success, count(eval(auth_status="failed")) as failed by endpoint
| eval success_rate=round((success/total)*100, 2)
| table endpoint, total, success, failed, success_rate
```

### Top API Consumers
```spl
index=api sourcetype=api:logs
| stats count by api_key, endpoint
| table api_key, endpoint, count
| sort - count
| head 20
```

### Response Payload Size
```spl
index=api sourcetype=api:logs
| stats avg(response_size) as avg_size, max(response_size) as max_size by endpoint
| table endpoint, avg_size, max_size
```

### API Dependency Health
```spl
index=api sourcetype=api:logs
| stats count by endpoint, dependency_service, dependency_status
| table endpoint, dependency_service, dependency_status, count
```

### Gateway Performance
```spl
index=api sourcetype=api_gateway OR sourcetype=kong OR sourcetype=apigee
| stats avg(gateway_latency) as avg_gateway, avg(upstream_latency) as avg_upstream by endpoint
| table endpoint, avg_gateway, avg_upstream
```

## 5. Database Application Dashboard

### Database Connection Pool Status
```spl
index=database sourcetype=connection_pool OR sourcetype=hikari
| stats avg(active_connections) as avg_active, max(max_connections) as max_conn by application, database
| eval utilization_pct=round((avg_active/max_conn)*100, 2)
| table application, database, avg_active, max_conn, utilization_pct
```

### Query Execution Time
```spl
index=database sourcetype=slowquery OR sourcetype=database:performance
| stats avg(query_duration) as avg_duration, max(query_duration) as max_duration by application, query_type
| table application, query_type, avg_duration, max_duration
```

### Slow Query Analysis
```spl
index=database sourcetype=slowquery
| stats count by application, database, query_duration, query_hash
| where query_duration > 5
| table application, database, query_duration, query_hash, count
| sort - query_duration
```

### Database Error Rate
```spl
index=database sourcetype=mssql OR sourcetype=mysql
| stats count as total, count(eval(error_code IN (connection_refused, timeout))) as errors by application, database
| eval error_rate=round((errors/total)*100, 2)
| table application, database, total, errors, error_rate
```

### Transaction Throughput
```spl
index=database sourcetype=database:performance
| timechart span=5m count as transactions by application, database
| fields - NULL
```

### Lock Wait Times
```spl
index=database sourcetype=mssql OR sourcetype=postgresql
| stats avg(lock_wait_time) as avg_wait, max(lock_wait_time) as max_wait by application, database
| table application, database, avg_wait, max_wait
```

### Deadlock Detection
```spl
index=database sourcetype=mssql OR sourcetype=postgresql
| stats count by application, database, deadlock_type, victim_process
| table application, database, deadlock_type, victim_process, count
```

### Cache Utilization
```spl
index=database sourcetype=cache OR sourcetype=redis
| stats avg(cache_hit_rate) as avg_hit, avg(cache_size) as avg_size by application
| table application, avg_hit, avg_size
```

### Index Usage Statistics
```spl
index=database sourcetype=database:performance
| stats count by application, database, index_name, index_usage
| table application, database, index_name, index_usage, count
```

### Query Optimization Opportunities
```spl
index=database sourcetype=slowquery
| stats avg(query_duration) as avg_duration, count by application, query_pattern
| where avg_duration > 3
| table application, query_pattern, avg_duration, count
| sort - avg_duration
```

## 6. Microservices Dashboard

### Service Health Status
```spl
index=microservices sourcetype=service_mesh OR sourcetype=kubernetes
| stats count by service_name, health_status, namespace
| table service_name, namespace, health_status, count
```

### Service Dependency Map
```spl
index=microservices sourcetype=service_mesh OR sourcetype=istio
| stats count by service_name, dependency_service, dependency_type
| table service_name, dependency_service, dependency_type, count
```

### Inter-service Latency
```spl
index=microservices sourcetype=service_mesh OR sourcetype=istio
| stats avg(duration) as avg_latency by source_service, dest_service
| table source_service, dest_service, avg_latency
```

### Circuit Breaker Status
```spl
index=microservices sourcetype=hystrix OR sourcetype=resilience4j
| stats count by service_name, circuit_breaker_status, failure_threshold
| table service_name, circuit_breaker_status, failure_threshold, count
```

### Service Mesh Metrics
```spl
index=microservices sourcetype=istio OR sourcetype=linkerd
| stats avg(request_duration) as avg_duration, avg(success_rate) as avg_success by service_name
| table service_name, avg_duration, avg_success
```

### Message Queue Depth
```spl
index=microservices sourcetype=kafka OR sourcetype=rabbitmq
| stats avg(queue_depth) as avg_depth, max(queue_depth) as max_depth by queue_name, service
| table queue_name, service, avg_depth, max_depth
```

### Service Discovery Status
```spl
index=microservices sourcetype=consul OR sourcetype=eureka
| stats count by service_name, instance_count, registration_status
| table service_name, instance_count, registration_status, count
```

### Deployment Frequency
```spl
index=deploy sourcetype=kubernetes OR sourcetype=helm
| timechart span=1d count as deployments by service_name
| fields - NULL
```

### Rollback Rate
```spl
index=deploy sourcetype=kubernetes OR sourcetype=helm
| stats count as total, count(eval(action="rollback")) as rollbacks by service_name
| eval rollback_rate=round((rollbacks/total)*100, 2)
| table service_name, total, rollbacks, rollback_rate
```

### Service-level Objectives (SLOs)
```spl
index=slo sourcetype=application
| stats avg(availability) as avg_avail, avg(latency) as avg_latency, avg(error_rate) as avg_error by service_name
| table service_name, avg_avail, avg_latency, avg_error
```

## 7. Error & Exception Dashboard

### Error Rate Trends
```spl
index=application sourcetype=access_combined OR sourcetype=application:logs
| timechart span=5m count(eval(status >= 400)) as errors, count as total by application
| eval error_rate=round((errors/total)*100, 2)
| fields - NULL
```

### Error Type Distribution
```spl
index=application sourcetype=java:exception OR sourcetype=python:exception
| stats count by application, exception_type, error_message
| table application, exception_type, error_message, count
| sort - count
```

### Stack Trace Analysis
```spl
index=application sourcetype=java:exception
| stats count by application, exception_type, stack_trace
| table application, exception_type, stack_trace, count
```

### Error Frequency by Endpoint
```spl
index=application sourcetype=access_combined
| stats count as total, count(eval(status >= 400)) as errors by application, endpoint
| eval error_rate=round((errors/total)*100, 2)
| table application, endpoint, total, errors, error_rate
| sort - error_rate
```

### Unhandled Exceptions
```spl
index=application sourcetype=java:exception OR sourcetype=python:exception
| stats count by application, exception_type, is_handled
| where is_handled="false"
| table application, exception_type, is_handled, count
```

### Error Impact on Users
```spl
index=application sourcetype=access_combined
| stats count as total_errors, dc(user) as affected_users by application, status
| where status >= 400
| table application, status, total_errors, affected_users
```

### Error Resolution Time
```spl
index=incident sourcetype=application
| stats avg(resolution_time) as avg_resolution by application, error_type
| table application, error_type, avg_resolution
```

### Recurring Error Detection
```spl
index=application sourcetype=java:exception
| stats count by application, exception_type, error_hash
| where count > 10
| table application, exception_type, error_hash, count
```

### Error Hotspots
```spl
index=application sourcetype=access_combined
| stats count by application, endpoint, status
| where status >= 400
| table application, endpoint, status, count
| sort - count
```

### Error Correlation with Deployments
```spl
index=application sourcetype=access_combined OR sourcetype=deployment
| stats count by application, status, deployment_id
| where status >= 400
| table application, status, deployment_id, count
```

## 8. Transaction & Business Logic Dashboard

### Transaction Success Rate
```spl
index=application sourcetype=transaction OR sourcetype=application:logs
| stats count as total, count(eval(status="success")) as success, count(eval(status="failed")) as failed by application, transaction_type
| eval success_rate=round((success/total)*100, 2)
| table application, transaction_type, total, success, failed, success_rate
```

### Transaction Volume Trends
```spl
index=application sourcetype=transaction
| timechart span=1h count as transactions by application, transaction_type
| fields - NULL
```

### Payment Processing Metrics
```spl
index=application sourcetype=payment OR sourcetype=transaction
| stats count as total, count(eval(payment_status="success")) as success, count(eval(payment_status="failed")) as failed by application, payment_gateway
| eval success_rate=round((success/total)*100, 2)
| table application, payment_gateway, total, success, failed, success_rate
```

### Order Fulfillment Status
```spl
index=application sourcetype=ecommerce OR sourcetype=order:logs
| stats count by application, order_status, order_type
| table application, order_status, order_type, count
```

### Workflow Completion Rates
```spl
index=application sourcetype=workflow OR sourcetype=bpm
| stats count as total, count(eval(status="completed")) as completed, count(eval(status="failed")) as failed by application, workflow_name
| eval completion_rate=round((completed/total)*100, 2)
| table application, workflow_name, total, completed, failed, completion_rate
```

### Business Rule Violations
```spl
index=application sourcetype=business_rules
| stats count by application, rule_name, violation_type, severity
| table application, rule_name, violation_type, severity, count
```

### Integration Success Rates
```spl
index=application sourcetype=integration OR sourcetype=api:logs
| stats count as total, count(eval(status="success")) as success, count(eval(status="failed")) as failed by application, integration_name
| eval success_rate=round((success/total)*100, 2)
| table application, integration_name, total, success, failed, success_rate
```

### Batch Job Status
```spl
index=application sourcetype=batch OR sourcetype=etl
| stats count by application, job_name, job_status, job_duration
| table application, job_name, job_status, job_duration, count
```

### Data Quality Metrics
```spl
index=application sourcetype=data_quality
| stats count as total, count(eval(quality="valid")) as valid, count(eval(quality="invalid")) as invalid by application, data_type
| eval quality_pct=round((valid/total)*100, 2)
| table application, data_type, total, valid, invalid, quality_pct
```

### KPI Tracking
```spl
index=application sourcetype=kpi OR sourcetype=analytics
| stats avg(kpi_value) as avg_kpi, max(kpi_value) as max_kpi by application, kpi_name
| table application, kpi_name, avg_kpi, max_kpi
```

## 9. User Experience Dashboard

### User Satisfaction Score
```spl
index=web sourcetype=rum OR sourcetype=analytics
| stats avg(satisfaction_score) as avg_score by application
| table application, avg_score
```

### Session Duration
```spl
index=web sourcetype=rum OR sourcetype=session
| stats avg(session_duration) as avg_duration, max(session_duration) as max_duration by application
| table application, avg_duration, max_duration
```

### Bounce Rate
```spl
index=web sourcetype=rum OR sourcetype=analytics
| stats count as total, count(eval(page_views=1)) as single_page by application
| eval bounce_rate=round((single_page/total)*100, 2)
| table application, total, single_page, bounce_rate
```

### Page Views per Session
```spl
index=web sourcetype=rum OR sourcetype=analytics
| stats avg(page_views) as avg_pages by application
| table application, avg_pages
```

### User Journey Flows
```spl
index=web sourcetype=rum OR sourcetype=analytics
| stats count by user_id, page_sequence, application
| table user_id, page_sequence, application, count
```

### Feature Adoption Rates
```spl
index=web sourcetype=analytics OR sourcetype=feature_flags
| stats count as total, count(eval(feature_used="true")) as used by application, feature_name
| eval adoption_rate=round((used/total)*100, 2)
| table application, feature_name, total, used, adoption_rate
```

### User Segmentation Metrics
```spl
index=web sourcetype=rum OR sourcetype=analytics
| stats count by application, user_segment, activity_level
| table application, user_segment, activity_level, count
```

### A/B Test Results
```spl
index=web sourcetype=analytics OR sourcetype=ab_testing
| stats avg(conversion_rate) as avg_conversion by application, test_name, variant
| table application, test_name, variant, avg_conversion
```

### User Feedback Trends
```spl
index=web sourcetype=feedback OR sourcetype=csat
| timechart span=1d avg(rating) as avg_rating by application
| fields - NULL
```

### NPS (Net Promoter Score)
```spl
index=web sourcetype=nps OR sourcetype=feedback
| stats count as total, count(eval(score >= 9)) as promoters, count(eval(score <= 6)) as detractors by application
| eval nps=round(((promoters - detractors) / total) * 100, 2)
| table application, total, promoters, detractors, nps
```

## 10. Deployment & CI/CD Dashboard

### Deployment Frequency
```spl
index=deploy sourcetype=jenkins OR sourcetype=gitlab-ci
| timechart span=1d count as deployments by application, environment
| fields - NULL
```

### Deployment Success Rate
```spl
index=deploy sourcetype=jenkins OR sourcetype=gitlab-ci
| stats count as total, count(eval(status="success")) as success, count(eval(status="failed")) as failed by application, environment
| eval success_rate=round((success/total)*100, 2)
| table application, environment, total, success, failed, success_rate
```

### Lead Time for Changes
```spl
index=deploy sourcetype=jenkins OR sourcetype=gitlab-ci
| stats avg(lead_time) as avg_lead, max(lead_time) as max_lead by application
| table application, avg_lead, max_lead
```

### Change Failure Rate
```spl
index=deploy sourcetype=jenkins OR sourcetype=gitlab-ci
| stats count as total, count(eval(deployment_status="failed")) as failed by application
| eval failure_rate=round((failed/total)*100, 2)
| table application, total, failed, failure_rate
```

### Mean Time to Recovery
```spl
index=incident sourcetype=application
| stats avg(resolution_time) as avg_mttr by application, incident_type
| table application, incident_type, avg_mttr
```

### Deployment Pipeline Status
```spl
index=ci sourcetype=jenkins OR sourcetype=gitlab-ci
| stats count by application, pipeline_name, stage, status
| table application, pipeline_name, stage, status, count
```

### Build Success Rate
```spl
index=ci sourcetype=jenkins OR sourcetype=gitlab-ci
| stats count as total, count(eval(build_status="success")) as success, count(eval(build_status="failed")) as failed by application
| eval success_rate=round((success/total)*100, 2)
| table application, total, success, failed, success_rate
```

### Test Coverage
```spl
index=ci sourcetype=jenkins OR sourcetype=gitlab-ci
| stats avg(coverage_pct) as avg_coverage by application
| table application, avg_coverage
```

### Feature Flag Status
```spl
index=deploy sourcetype=feature_flags OR sourcetype=launchdarkly
| stats count by application, feature_name, flag_status, environment
| table application, feature_name, flag_status, environment, count
```

### Rollback Frequency
```spl
index=deploy sourcetype=kubernetes OR sourcetype=helm
| stats count as total, count(eval(action="rollback")) as rollbacks by application, environment
| eval rollback_rate=round((rollbacks/total)*100, 2)
| table application, environment, total, rollbacks, rollback_rate
```

## 11. Application Security Dashboard

### Authentication Failures
```spl
index=application sourcetype=auth OR sourcetype=api:logs
| stats count as total, count(eval(auth_status="failed")) as failed by application, endpoint
| eval failure_rate=round((failed/total)*100, 2)
| table application, endpoint, total, failed, failure_rate
```

### Authorization Violations
```spl
index=application sourcetype=access_combined OR sourcetype=api:logs
| stats count by application, status, endpoint, user
| where status=403
| table application, status, endpoint, user, count
```

### Injection Attack Attempts
```spl
index=web sourcetype=access_combined
| stats count by application, attack_type, endpoint, src_ip
| where attack_type IN (sql_injection, xss, command_injection)
| table application, attack_type, endpoint, src_ip, count
```

### XSS/CSRF Attempts
```spl
index=web sourcetype=access_combined
| stats count by application, attack_type, endpoint, src_ip
| where attack_type IN (xss, csrf)
| table application, attack_type, endpoint, src_ip, count
```

### API Abuse Detection
```spl
index=api sourcetype=api:logs
| stats count by application, api_key, endpoint, abuse_type
| where abuse_type="rate_limit_exceeded"
| table application, api_key, endpoint, abuse_type, count
```

### Data Access Violations
```spl
index=application sourcetype=access OR sourcetype=database
| stats count by application, user, resource, access_type
| where access_type="denied"
| table application, user, resource, access_type, count
```

### Security Event Trends
```spl
index=application sourcetype=security
| timechart span=1h count by application, event_type
| fields - NULL
```

### Vulnerability Scan Results
```spl
index=application sourcetype=vulnerability OR sourcetype=sast
| stats count by application, vulnerability_severity, cve_id
| table application, vulnerability_severity, cve_id, count
```

### WAF (Web Application Firewall) Alerts
```spl
index=web sourcetype=waf OR sourcetype=aws:waf
| stats count by application, rule_name, violation_type, action
| table application, rule_name, violation_type, action, count
```

### Compliance Status
```spl
index=application sourcetype=compliance OR sourcetype=security
| stats count as total, count(eval(status="compliant")) as compliant, count(eval(status="non_compliant")) as non_compliant by application, standard
| eval compliance_pct=round((compliant/total)*100, 2)
| table application, standard, total, compliant, non_compliant, compliance_pct
```

## 12. APM (Application Performance Monitoring) Dashboard

### Service Map Topology
```spl
index=apm sourcetype=apm:service_map
| stats count by service_name, dependency_service, connection_type
| table service_name, dependency_service, connection_type, count
```

### Distributed Tracing Overview
```spl
index=apm sourcetype=apm:trace
| stats count by trace_id, service_name, span_count, duration
| table trace_id, service_name, span_count, duration
```

### Transaction Trace Analysis
```spl
index=apm sourcetype=apm:trace
| stats avg(duration) as avg_duration, max(duration) as max_duration by service_name, transaction_name
| table service_name, transaction_name, avg_duration, max_duration
```

### Span Duration Metrics
```spl
index=apm sourcetype=apm:trace
| stats avg(span_duration) as avg_span, max(span_duration) as max_span by service_name, span_name
| table service_name, span_name, avg_span, max_span
```

### Service Dependency Health
```spl
index=apm sourcetype=apm:service_map
| stats avg(dependency_health) as avg_health by service_name, dependency_service
| table service_name, dependency_service, avg_health
```

### Trace Sampling Rate
```spl
index=apm sourcetype=apm:trace
| stats count as total_traces, count(eval(sampled="true")) as sampled by service_name
| eval sampling_rate=round((sampled/total_traces)*100, 2)
| table service_name, total_traces, sampled, sampling_rate
```

### Hot Path Identification
```spl
index=apm sourcetype=apm:trace
| stats avg(duration) as avg_duration by service_name, span_name
| sort - avg_duration
| head 20
```

### Performance Bottleneck Detection
```spl
index=apm sourcetype=apm:trace
| stats avg(duration) as avg_duration, count as span_count by service_name, span_name
| where avg_duration > 1
| table service_name, span_name, avg_duration, span_count
| sort - avg_duration
```

### Error Trace Analysis
```spl
index=apm sourcetype=apm:trace
| stats count by service_name, span_name, error_type, error_message
| where error_type!=""
| table service_name, span_name, error_type, error_message, count
```

### Call Graph Visualization
```spl
index=apm sourcetype=apm:trace
| stats count by parent_span, child_span, service_name
| table parent_span, child_span, service_name, count
```
