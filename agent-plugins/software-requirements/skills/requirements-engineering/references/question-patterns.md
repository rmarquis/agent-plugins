# Clarifying Question Patterns

Organized by domain, these question patterns help elicit complete requirements from informal descriptions.

## User and Access Questions

### Users and Roles
- Who are the primary users of this feature?
- Are there different user roles with different permissions?
- Do users need to be authenticated to use this?
- Should actions be auditable (who did what, when)?

### User Experience
- What is the expected user journey/flow?
- Are there specific UI/UX constraints or preferences?
- Should this work on mobile devices?
- What accessibility requirements exist?

## Data and Storage Questions

### Data Model
- What data needs to be stored?
- What are the relationships between entities?
- Are there any data validation rules?
- Should data be versioned or have history?

### Data Lifecycle
- How long should data be retained?
- Is there a need for soft delete vs hard delete?
- Should deleted data be recoverable?
- Are there compliance requirements (GDPR, etc.)?

## Integration Questions

### External Systems
- What existing systems does this need to integrate with?
- Are there APIs to consume or expose?
- What authentication methods do integrations use?
- How should integration failures be handled?

### Data Exchange
- What data formats are required (JSON, XML, CSV)?
- Is real-time sync needed or is batch acceptable?
- How should conflicts be resolved?
- What's the expected data volume for integrations?

## Performance Questions

### Scale
- How many users are expected (concurrent, total)?
- What's the expected data volume (records, storage)?
- Are there peak usage patterns to consider?
- What growth is anticipated over time?

### Response Time
- What response times are acceptable?
- Are there different SLAs for different operations?
- Is real-time processing required or can tasks be async?
- Are there batch processing needs?

## Security Questions

### Authentication
- How should users authenticate?
- Is multi-factor authentication required?
- Should sessions expire? After how long?
- Is single sign-on (SSO) needed?

### Authorization
- What permission model is needed (role-based, attribute-based)?
- Are there data-level access controls?
- Should permissions be configurable?
- Who can grant/revoke permissions?

### Data Protection
- Is data encryption required (at rest, in transit)?
- Are there sensitive fields requiring masking?
- What audit logging is required?
- Are there compliance standards to meet?

## Error Handling Questions

### Failure Scenarios
- What should happen if the operation fails?
- Should failed operations be retried automatically?
- How should users be notified of failures?
- Is there a fallback behavior?

### Recovery
- Can partially completed operations be resumed?
- How should data inconsistencies be handled?
- What manual intervention might be needed?
- Should there be an undo/rollback capability?

## Business Rules Questions

### Validation
- What validation rules apply to inputs?
- Are there business rules that constrain operations?
- When should validation occur (client, server, both)?
- How should validation errors be communicated?

### Workflow
- Is there an approval workflow?
- Are there status transitions to enforce?
- Should notifications be sent at certain points?
- Are there time-based triggers or deadlines?

## Deployment Questions

### Environment
- Where will this be deployed (cloud, on-premise)?
- Are there specific infrastructure constraints?
- Is containerization expected?
- What monitoring/observability is needed?

### Release
- How will this be rolled out (big bang, phased)?
- Is feature flagging needed?
- What's the rollback strategy?
- Are there dependencies on other releases?

## Question Selection Strategy

When selecting questions to ask:

1. **Prioritize blockers**: Questions whose answers affect multiple requirements.
2. **Address ambiguity**: Clarify vague terms before detailed questions.
3. **Consider impact**: Focus on high-risk or high-complexity areas.
4. **Limit quantity**: Ask 3–5 questions maximum per round.
5. **Offer options**: When possible, present choices to reduce cognitive load.

Example of a well-formed question:

> "How should the system handle authentication? Options:
> - Email/password only
> - OAuth (Google, GitHub, etc.)
> - Both email/password and OAuth
> - Single Sign-On (SSO) integration"

This provides context, shows you understand the domain, and makes it easy for the user to respond.
