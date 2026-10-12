# Application Alerts

## Application Availability
- Application down/unreachable
- Application crash detection
- Service restart events
- Process termination
- Application startup failures
- Health check failures
- Endpoint not responding
- API endpoint downtime
- Web server down
- Application pool recycling

## Performance Metrics
- Response time > threshold (e.g., > 5 seconds)
- Error rate increase (> 5%)
- Throughput degradation
- Slow page load times
- API latency spikes
- Database query slowness
- Cache miss rate high
- Memory leaks detection
- Thread pool exhaustion
- Connection pool exhaustion

## Error & Exception Handling
- Application errors (500, 502, 503, 504)
- Unhandled exceptions
- Stack overflow errors
- Out of memory errors
- Null pointer exceptions
- Timeout errors
- Database connection errors
- Network timeout errors
- File system errors
- Serialization errors

## Transaction & Business Logic
- Transaction failures
- Payment processing failures
- Order processing errors
- Workflow failures
- Business rule violations
- Data validation failures
- Integration failures
- API call failures
- Message queue processing failures
- Batch job failures

## Web Application Specific
- HTTP 4xx errors (client errors)
- HTTP 5xx errors (server errors)
- High error rate by endpoint
- Bot activity detection
- Web crawler anomalies
- API rate limit hits
- CORS errors
- SSL/TLS certificate expiration
- CSRF token failures
- Session timeout issues

## Database Application Layer
- Slow ORM queries
- N+1 query detection
- Database connection pool exhaustion
- Deadlock detection
- Foreign key constraint violations
- Unique constraint violations
- Database migration failures
- ORM-specific errors
- Entity framework issues
- Hibernate/JPA errors

## API & Microservices
- Service dependency failures
- Circuit breaker activation
- Service mesh issues
- gRPC connection failures
- REST API failures
- GraphQL errors
- Message broker connection issues
- Service discovery failures
- API gateway issues
- Rate limiting triggers

## Authentication & Authorization (App Level)
- Authentication token expiration
- Invalid JWT tokens
- Authorization failures (403)
- Session fixation attempts
- CSRF token validation failures
- OAuth token refresh failures
- SAML assertion failures
- Permission denied events
- Role-based access violations
- API key authentication failures

## Logging & Monitoring
- Log file rotation issues
- Log aggregation failures
- Metric collection failures
- APM agent disconnection
- Distributed tracing gaps
- Log parsing errors
- Metric shipping failures
- Alert delivery failures
- Dashboard data gaps
- Monitoring system issues

## Deployment & CI/CD
- Deployment failures
- Rollback events
- Canary deployment issues
- Blue-green deployment issues
- Build failures
- Test failures in pipeline
- Deployment pipeline stuck
- Feature flag issues
- Configuration drift
- Infrastructure as code drift
