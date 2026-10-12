# Application Alerts - SPL Queries

## Application Availability

### Application Down/Unreachable
```spl
index=application sourcetype=access_combined OR sourcetype=nginx:access OR sourcetype=apache:access
| stats count by host, status
| where status >= 500 OR count = 0
| table _time, host, status, count
| eval severity="critical"
```

### Application Crash Detection
```spl
index=application sourcetype=application:logs OR sourcetype=java:exception
| stats count by application, error_type, message
| where error_type IN (crash, fatal, segmentation_fault)
| table _time, application, error_type, message, count
| eval severity="critical"
```

### Service Restart Events
```spl
index=application sourcetype=systemd OR sourcetype=supervisord
| stats count by service_name, action, host
| where action="restart" OR action="stop"
| table _time, service_name, action, host, count
```

### Application Startup Failures
```spl
index=application sourcetype=application:logs
| stats count by application, startup_status, error_message
| where startup_status="failed"
| table _time, application, startup_status, error_message, count
| eval severity="critical"
```

### Health Check Failures
```spl
index=application sourcetype=healthcheck OR sourcetype=ping
| stats count by application, health_status, endpoint
| where health_status="unhealthy" OR health_status="down"
| table _time, application, health_status, endpoint, count
| eval severity="critical"
```

## Performance Metrics

### Response Time > Threshold (> 5 seconds)
```spl
index=application sourcetype=access_combined OR sourcetype=nginx:access
| stats avg(response_time) as avg_response by application, endpoint
| where avg_response > 5
| table _time, application, endpoint, avg_response
| eval severity="warning"
```

### Error Rate Increase (> 5%)
```spl
index=application sourcetype=access_combined OR sourcetype=nginx:access
| stats count as total, count(eval(status >= 400)) as errors by application
| eval error_rate=round((errors/total)*100, 2)
| where error_rate > 5
| table _time, application, total, errors, error_rate
| eval severity="warning"
```

### Throughput Degradation
```spl
index=application sourcetype=access_combined
| timechart span=5m count as requests by application
| eventstats avg(requests) as avg_requests
| where requests < (avg_requests * 0.5)
| eval severity="warning"
| table _time, application, requests, avg_requests
```

### Slow Page Load Times
```spl
index=web sourcetype=rum OR sourcetype=webperf
| stats avg(page_load_time) as avg_load by page_url
| where avg_load > 3
| table _time, page_url, avg_load
| eval severity="warning"
```

### API Latency Spikes
```spl
index=api sourcetype=api:logs
| timechart span=5m avg(response_time) as avg_latency by endpoint
| where avg_latency > 2
| table _time, endpoint, avg_latency
| eval severity="warning"
```

### Database Query Slowness
```spl
index=database sourcetype=slowquery OR sourcetype=database:performance
| stats avg(query_duration) as avg_duration by application, query_type
| where avg_duration > 5
| table _time, application, query_type, avg_duration
```

### Cache Miss Rate High
```spl
index=application sourcetype=redis OR sourcetype=memcached
| stats sum(cache_misses) as misses, sum(cache_hits) as hits by application
| eval miss_rate=round((misses/(misses+hits))*100, 2)
| where miss_rate > 30
| table _time, application, misses, hits, miss_rate
```

### Memory Leaks Detection
```spl
index=application sourcetype=jmx OR sourcetype=metrics
| timechart span=1h avg(heap_memory) as avg_heap by application
| trendline sma5(avg_heap) as trend
| where avg_heap > (trend * 1.5)
| eval severity="warning"
| table _time, application, avg_heap, trend
```

### Thread Pool Exhaustion
```spl
index=application sourcetype=jmx OR sourcetype=metrics
| stats avg(active_threads) as avg_threads, max(threads) as max_threads by application
| where avg_threads > (max_threads * 0.9)
| table _time, application, avg_threads, max_threads
| eval severity="critical"
```

## Error & Exception Handling

### Application Errors (500, 502, 503, 504)
```spl
index=application sourcetype=access_combined OR sourcetype=nginx:access
| stats count by application, status, endpoint
| where status IN (500, 502, 503, 504)
| table _time, application, status, endpoint, count
| eval severity="critical"
```

### Unhandled Exceptions
```spl
index=application sourcetype=java:exception OR sourcetype=python:exception
| stats count by application, exception_type, stack_trace
| where exception_type IN (NullPointerException, IOException, RuntimeError)
| table _time, application, exception_type, stack_trace, count
```

### Stack Overflow Errors
```spl
index=application sourcetype=java:exception
| stats count by application, exception_type
| where exception_type="StackOverflowError"
| table _time, application, exception_type, count
| eval severity="critical"
```

### Out of Memory Errors
```spl
index=application sourcetype=java:exception OR sourcetype=application:logs
| stats count by application, error_type
| where error_type="OutOfMemoryError" OR error_type="memory_exhausted"
| table _time, application, error_type, count
| eval severity="critical"
```

### Timeout Errors
```spl
index=application sourcetype=api:logs OR sourcetype=application:logs
| stats count by application, error_type, timeout_duration
| where error_type="timeout" OR error_type="connection_timeout"
| table _time, application, error_type, timeout_duration, count
```

### Database Connection Errors
```spl
index=database sourcetype=mssql OR sourcetype=mysql OR sourcetype=postgresql
| stats count by application, error_code, error_message
| where error_code IN (connection_refused, timeout, cannot_connect)
| table _time, application, error_code, error_message, count
```

## Transaction & Business Logic

### Transaction Failures
```spl
index=application sourcetype=transaction OR sourcetype=application:logs
| stats count by application, transaction_type, status
| where status="failed" OR status="error"
| table _time, application, transaction_type, status, count
```

### Payment Processing Failures
```spl
index=application sourcetype=payment OR sourcetype=transaction
| stats count by application, payment_status, payment_gateway
| where payment_status="failed" OR payment_status="declined"
| table _time, application, payment_status, payment_gateway, count
| eval severity="critical"
```

### Order Processing Errors
```spl
index=application sourcetype=ecommerce OR sourcetype=order:logs
| stats count by application, order_status, error_code
| where order_status="failed" OR order_status="error"
| table _time, application, order_status, error_code, count
```

### Workflow Failures
```spl
index=application sourcetype=workflow OR sourcetype=bpm
| stats count by application, workflow_name, step_name, status
| where status="failed"
| table _time, application, workflow_name, step_name, status, count
```

### Integration Failures
```spl
index=application sourcetype=integration OR sourcetype=api:logs
| stats count by application, integration_name, external_service, error_type
| where error_type="connection_failed" OR error_type="timeout"
| table _time, application, integration_name, external_service, error_type, count
```

## Web Application Specific

### HTTP 4xx Errors (Client Errors)
```spl
index=web sourcetype=access_combined OR sourcetype=nginx:access
| stats count by application, status, endpoint
| where status >= 400 AND status < 500
| table _time, application, status, endpoint, count
```

### High Error Rate by Endpoint
```spl
index=web sourcetype=access_combined
| stats count as total, count(eval(status >= 400)) as errors by endpoint
| eval error_rate=round((errors/total)*100, 2)
| where error_rate > 10
| table _time, endpoint, total, errors, error_rate
| sort - error_rate
```

### Bot Activity Detection
```spl
index=web sourcetype=access_combined
| stats count by user_agent, endpoint
| where user_agent IN (*bot*, *crawler*, *spider*)
| table _time, user_agent, endpoint, count
```

### API Rate Limit Hits
```spl
index=api sourcetype=api:logs
| stats count by api_key, endpoint, rate_limit_status
| where rate_limit_status="429" OR rate_limit_status="rate_limit_exceeded"
| table _time, api_key, endpoint, rate_limit_status, count
```

### SSL/TLS Certificate Expiration
```spl
index=web sourcetype=ssl OR sourcetype=certificate
| stats count by domain, certificate_expiry, days_remaining
| where days_remaining < 30
| table _time, domain, certificate_expiry, days_remaining
| eval severity=if(days_remaining < 7, "critical", "warning")
```

## Database Application Layer

### Slow ORM Queries
```spl
index=database sourcetype=orm OR sourcetype=hibernate OR sourcetype=entityframework
| stats avg(query_duration) as avg_duration by application, query_type
| where avg_duration > 3
| table _time, application, query_type, avg_duration
```

### N+1 Query Detection
```spl
index=database sourcetype=orm OR sourcetype=hibernate
| stats count by application, query_pattern, execution_count
| where execution_count > 100
| table _time, application, query_pattern, execution_count
```

### Database Connection Pool Exhaustion
```spl
index=database sourcetype=connection_pool OR sourcetype=hikari
| stats avg(active_connections) as avg_active, max(max_connections) as max_conn by application
| where avg_active > (max_conn * 0.9)
| table _time, application, avg_active, max_conn
| eval severity="critical"
```

### Deadlock Detection
```spl
index=database sourcetype=mssql OR sourcetype=postgresql
| stats count by application, deadlock_type, victim_process
| where deadlock_type="deadlock"
| table _time, application, deadlock_type, victim_process, count
| eval severity="critical"
```

## API & Microservices

### Service Dependency Failures
```spl
index=microservices sourcetype=service_mesh OR sourcetype=istio
| stats count by service_name, dependency_service, health_status
| where health_status="unhealthy" OR health_status="down"
| table _time, service_name, dependency_service, health_status, count
```

### Circuit Breaker Activation
```spl
index=microservices sourcetype=hystrix OR sourcetype=resilience4j
| stats count by service_name, circuit_breaker_status
| where circuit_breaker_status="open"
| table _time, service_name, circuit_breaker_status, count
| eval severity="warning"
```

### gRPC Connection Failures
```spl
index=microservices sourcetype=grpc
| stats count by service_name, grpc_status, error_message
| where grpc_status >= 10
| table _time, service_name, grpc_status, error_message, count
```

### REST API Failures
```spl
index=api sourcetype=api:logs
| stats count by endpoint, method, status
| where status >= 400
| table _time, endpoint, method, status, count
```

### Message Broker Connection Issues
```spl
index=microservices sourcetype=kafka OR sourcetype=rabbitmq
| stats count by service_name, broker_status, connection_error
| where broker_status="disconnected" OR connection_error="true"
| table _time, service_name, broker_status, connection_error, count
```

## Authentication & Authorization (App Level)

### Authentication Token Expiration
```spl
index=application sourcetype=auth OR sourcetype=jwt
| stats count by application, token_status, error_type
| where token_status="expired" OR error_type="token_expired"
| table _time, application, token_status, error_type, count
```

### Authorization Failures (403)
```spl
index=application sourcetype=access_combined OR sourcetype=api:logs
| stats count by application, status, endpoint, user
| where status=403
| table _time, application, status, endpoint, user, count
```

### Permission Denied Events
```spl
index=application sourcetype=authorization OR sourcetype=rbac
| stats count by application, user, permission, resource
| where access="denied"
| table _time, application, user, permission, resource, count
```

## Logging & Monitoring

### Log File Rotation Issues
```spl
index=application sourcetype=logrotate OR sourcetype=application:logs
| stats count by application, log_file, rotation_status
| where rotation_status="failed"
| table _time, application, log_file, rotation_status, count
```

### APM Agent Disconnection
```spl
index=apm sourcetype=apm:agent OR sourcetype=newrelic OR sourcetype=datadog
| stats count by application, agent_status, last_heartbeat
| where agent_status="disconnected" OR (now-last_heartbeat) > 300
| table _time, application, agent_status, last_heartbeat, count
```

## Deployment & CI/CD

### Deployment Failures
```spl
index=deploy sourcetype=jenkins OR sourcetype=gitlab OR sourcetype=github
| stats count by application, deployment_status, environment
| where deployment_status="failed"
| table _time, application, deployment_status, environment, count
| eval severity="critical"
```

### Rollback Events
```spl
index=deploy sourcetype=kubernetes OR sourcetype=helm
| stats count by application, action, previous_version, new_version
| where action="rollback"
| table _time, application, action, previous_version, new_version, count
```

### Canary Deployment Issues
```spl
index=deploy sourcetype=istio OR sourcetype=argocd
| stats count by application, deployment_type, canary_status, error_rate
| where deployment_type="canary" AND canary_status="failed"
| table _time, application, deployment_type, canary_status, error_rate, count
```

### Build Failures
```spl
index=ci sourcetype=jenkins OR sourcetype=gitlab-ci
| stats count by application, build_status, build_number
| where build_status="failed"
| table _time, application, build_status, build_number, count
```
