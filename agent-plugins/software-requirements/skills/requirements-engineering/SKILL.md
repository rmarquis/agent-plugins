---
name: requirements-engineering
description: |
  Use this skill when the user asks to refine requirements, create a requirements document,
  turn a description into requirements, write functional requirements, produce a PRD or BRD,
  or when analyzing informal feature descriptions. Produces structured markdown requirements
  documents with functional requirements, non-functional requirements, constraints, and
  acceptance criteria.
version: 1.0.0
---

# Requirements Engineering

Transform informal feature descriptions into structured, actionable software requirements documents through systematic analysis and targeted clarification.

## Core Process

### 1. Analyze the Initial Description

When receiving an informal description, identify:

- **Core functionality**: What the system must do
- **Implied requirements**: Unstated but necessary capabilities
- **Ambiguities**: Vague terms needing clarification
- **Missing information**: Gaps that need user input

### 2. Ask Clarifying Questions

Focus on 3–5 targeted questions addressing the most significant gaps:

**Priority question areas:**

1. **Users and roles**: Who uses this? Different permission levels?
2. **Scale and performance**: Expected load, response times, data volume?
3. **Integration points**: What existing systems must it work with?
4. **Error handling**: What happens when things go wrong?
5. **Security concerns**: Authentication, authorization, data sensitivity?
6. **Edge cases**: Boundary conditions, unusual scenarios?

**Question formulation guidelines:**

- Ask specific, concrete questions (not "tell me more").
- Offer options when possible to reduce cognitive load.
- Group related questions logically.
- Prioritize questions that unblock multiple requirements.

### 3. Structure the Requirements Document

Generate a markdown document with these sections:

```markdown
# [Feature Name] Requirements

## Overview
Brief description of the feature, its purpose, and primary goals.

## Functional Requirements
### FR-1: [Requirement Name]
**Description**: What the system must do
**Acceptance Criteria**: How to verify it works
**Priority**: Must Have | Should Have | Nice to Have

### FR-2: [Next Requirement]
...

## Non-Functional Requirements
### NFR-1: Performance
- Response time targets
- Throughput expectations
- Scalability requirements

### NFR-2: Security
- Authentication requirements
- Authorization rules
- Data protection needs

### NFR-3: Reliability
- Uptime expectations
- Recovery requirements
- Data integrity constraints

## Constraints
- Technical limitations
- Business rules
- Regulatory requirements
- Timeline constraints

## Assumptions
- Dependencies on other systems
- User capabilities assumed
- Environment assumptions

## Open Questions
- Items requiring further stakeholder input
- Technical decisions pending investigation
- Scope items needing clarification
```

### 4. Save the Document

Save requirements to: `docs/requirements/<feature-name>.md`

- Create the `docs/requirements/` directory if it does not exist.
- Use kebab-case for the filename.
- Derive the name from the feature being specified.

## Requirement Writing Guidelines

### Functional Requirements

Write each requirement to be:

- **Specific**: Clear, unambiguous language
- **Measurable**: Includes acceptance criteria
- **Achievable**: Technically feasible
- **Relevant**: Directly supports a user need
- **Traceable**: Can be linked to user goals

**Good example:**
```markdown
### FR-3: Password Reset
**Description**: Users can reset their password via email verification link.
**Acceptance Criteria**:
- Reset link sent within 30 seconds of request
- Link expires after 24 hours
- User receives confirmation after successful reset
**Priority**: Must Have
```

**Bad example:**
```markdown
### FR-3: Password stuff
Users should be able to reset passwords easily.
```

### Non-Functional Requirements

Quantify where possible:

- "Response time under 200ms for 95th percentile" (not "fast")
- "Support 1000 concurrent users" (not "scalable")
- "99.9% uptime" (not "highly available")

### Priority Levels

- **Must Have**: Core functionality; system will not work without it.
- **Should Have**: Important but not critical for initial release.
- **Nice to Have**: Enhances experience; can be deferred.

## Interactive Refinement Pattern

When refining requirements interactively:

1. **Acknowledge** the description received.
2. **Summarize** your understanding of the core request.
3. **Identify** the key gaps or ambiguities.
4. **Ask** 3–5 focused clarifying questions.
5. **Draft** requirements based on answers.
6. **Review** with the user for accuracy.
7. **Finalize** and save the document.

Present structured questions with options when multiple choices exist.

## Common Patterns

### CRUD Operations

For data management features, ensure requirements cover:

- **Create**: Validation rules, required fields
- **Read**: Filtering, sorting, pagination
- **Update**: Partial vs full updates, versioning
- **Delete**: Soft vs hard delete, cascading effects

### User Authentication

Standard requirements to consider:

- Login methods (email/password, OAuth, SSO)
- Session management (duration, concurrent sessions)
- Password policies (complexity, expiration)
- Account recovery mechanisms
- Audit logging

### API Endpoints

For each endpoint, specify:

- HTTP method and path
- Request/response formats
- Authentication requirements
- Rate limiting
- Error responses

## Additional Resources

### Reference Files

- **`references/question-patterns.md`** — Common clarifying questions by domain
- **`references/nfr-checklist.md`** — Non-functional requirements checklist

### Example Files

- **`examples/user-auth-requirements.md`** — Complete authentication requirements example
