# Non-Functional Requirements Checklist

Use this checklist to ensure comprehensive coverage of non-functional requirements.

## Performance

### Response Time
- [ ] Page load time targets (e.g., < 2 seconds)
- [ ] API response time targets (e.g., < 200ms p95)
- [ ] Search/query response times
- [ ] File upload/download times
- [ ] Report generation times

### Throughput
- [ ] Requests per second capacity
- [ ] Transactions per minute
- [ ] Data processing rate
- [ ] Batch job throughput

### Scalability
- [ ] Concurrent user capacity
- [ ] Data storage growth plan
- [ ] Horizontal scaling requirements
- [ ] Auto-scaling triggers and limits

## Reliability

### Availability
- [ ] Uptime target (e.g., 99.9%)
- [ ] Planned maintenance windows
- [ ] Geographic redundancy needs
- [ ] Failover requirements

### Fault Tolerance
- [ ] Single point of failure analysis
- [ ] Graceful degradation behavior
- [ ] Circuit breaker patterns
- [ ] Retry policies

### Recovery
- [ ] Recovery Time Objective (RTO)
- [ ] Recovery Point Objective (RPO)
- [ ] Backup frequency and retention
- [ ] Disaster recovery plan

### Data Integrity
- [ ] Transaction consistency requirements
- [ ] Data validation rules
- [ ] Duplicate detection
- [ ] Referential integrity

## Security

### Authentication
- [ ] Authentication methods supported
- [ ] Password policy requirements
- [ ] Multi-factor authentication
- [ ] Session management rules
- [ ] Account lockout policy

### Authorization
- [ ] Role-based access control (RBAC)
- [ ] Resource-level permissions
- [ ] API authorization (OAuth, API keys)
- [ ] Admin/superuser access

### Data Protection
- [ ] Encryption at rest
- [ ] Encryption in transit (TLS)
- [ ] Sensitive data handling (PII, PCI)
- [ ] Data masking requirements
- [ ] Key management

### Audit & Compliance
- [ ] Audit logging requirements
- [ ] Log retention period
- [ ] Compliance standards (SOC2, HIPAA, GDPR)
- [ ] Penetration testing requirements

## Usability

### Accessibility
- [ ] WCAG compliance level (A, AA, AAA)
- [ ] Screen reader compatibility
- [ ] Keyboard navigation
- [ ] Color contrast requirements

### Internationalization
- [ ] Supported languages
- [ ] Right-to-left (RTL) support
- [ ] Date/time/number formatting
- [ ] Currency handling

### User Experience
- [ ] Browser compatibility
- [ ] Mobile responsiveness
- [ ] Offline capabilities
- [ ] Error message clarity

## Maintainability

### Code Quality
- [ ] Coding standards
- [ ] Code review requirements
- [ ] Test coverage targets
- [ ] Documentation requirements

### Deployment
- [ ] Deployment frequency target
- [ ] Zero-downtime deployment
- [ ] Rollback procedures
- [ ] Feature flag usage

### Monitoring
- [ ] Logging standards
- [ ] Metrics collection
- [ ] Alerting thresholds
- [ ] Dashboard requirements

## Compatibility

### Integration
- [ ] API versioning strategy
- [ ] Backward compatibility requirements
- [ ] Third-party service dependencies
- [ ] Data format standards

### Platform
- [ ] Operating system support
- [ ] Browser version support
- [ ] Mobile OS support
- [ ] Database compatibility

## Operational

### Support
- [ ] Support hours/SLA
- [ ] Escalation procedures
- [ ] On-call requirements
- [ ] Incident response time

### Capacity Planning
- [ ] Growth projections
- [ ] Resource monitoring
- [ ] Capacity alerts
- [ ] Scaling procedures

## Documentation Requirements

- [ ] API documentation (OpenAPI/Swagger)
- [ ] User documentation
- [ ] Admin/operations guides
- [ ] Architecture documentation
- [ ] Runbooks for common issues

## How to Use This Checklist

1. **Initial review**: Go through categories relevant to the feature.
2. **Prioritize**: Mark which NFRs are critical vs nice-to-have.
3. **Quantify**: Add specific, measurable targets where possible.
4. **Validate**: Confirm requirements with stakeholders.
5. **Document**: Include relevant NFRs in the requirements document.

Not every item applies to every feature. Focus on categories relevant to the specific functionality being specified.
