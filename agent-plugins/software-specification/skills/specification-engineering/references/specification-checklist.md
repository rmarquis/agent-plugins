# Specification Checklist

Use this checklist to verify specification completeness and quality before considering the specification phase complete.

## Pre-Specification Setup

- [ ] Architecture document exists at `docs/architecture/<feature>.md`
- [ ] Requirements document exists at `docs/requirements/<feature>.md`
- [ ] Module structure is clearly defined in architecture
- [ ] Acceptance criteria are written for all functional requirements
- [ ] Non-functional requirements have measurable criteria

## Contract Specification Checklist

### Interface Coverage

- [ ] Every public interface has a contract spec class
- [ ] Every public method has at least one contract test
- [ ] Interface name matches `<Interface>ContractSpec` pattern

### Method Contracts

For each public method, verify:

- [ ] **Happy path** — normal operation with valid input
- [ ] **Return type** — correct type and structure returned
- [ ] **Null handling** — behavior for null/nil input (if applicable)
- [ ] **Empty input** — behavior for empty collections/strings (if applicable)
- [ ] **Boundary values** — edge cases at limits
- [ ] **Error conditions** — errors produced for invalid input
- [ ] **Preconditions** — documented and tested
- [ ] **Postconditions** — verified after method execution

### Stateful Operations

- [ ] **Create operations** return created entity with ID
- [ ] **Read operations** return null/empty for missing data
- [ ] **Update operations** return updated entity
- [ ] **Delete operations** are idempotent (don't throw on missing)
- [ ] **State transitions** are valid (can't skip states)

### Error Handling

- [ ] Invalid input produces a documented error
- [ ] Missing resource produces an appropriate error or returns null
- [ ] Error messages are descriptive
- [ ] Error types are documented

## Behavior Specification Checklist

### Requirements Traceability

- [ ] Every functional requirement (FR-n) has at least one behavior test
- [ ] Test names include or reference requirement ID
- [ ] All acceptance criteria are covered by tests
- [ ] Traceability comments link tests to requirements

### Given-When-Then Structure

For each behavior test:

- [ ] **Given** clearly establishes preconditions
- [ ] **When** performs exactly one action
- [ ] **Then** verifies expected outcomes
- [ ] Test name reads as a complete sentence

### User Story Coverage

- [ ] Primary success path tested
- [ ] Alternative paths tested
- [ ] Error/failure paths tested
- [ ] Edge cases from acceptance criteria tested

### Readability

- [ ] Test names use natural language
- [ ] Names describe behavior, not implementation
- [ ] Related tests are grouped together
- [ ] No technical jargon in test names

## Property Specification Checklist

### Property Identification

Review each module for these property types:

- [ ] **Roundtrip properties** identified
  - Serialization/deserialization
  - Save/load
  - Encode/decode
  - Convert to/from

- [ ] **Idempotence properties** identified
  - Delete operations
  - Cleanup operations
  - Normalization functions
  - Deduplication

- [ ] **Commutativity properties** identified (where applicable)
  - Set operations
  - Mathematical operations
  - Merge operations

- [ ] **Invariant properties** identified
  - State always valid
  - Counts never negative
  - Collections maintain ordering
  - Timestamps monotonically increase

### Generator Quality

- [ ] Generators produce valid domain objects
- [ ] Generators cover the full range of valid inputs
- [ ] Generators use the language's standard random library
- [ ] Seed is deterministic for reproducibility

### Test Structure

- [ ] Iterates each property ~100 times (adjust as needed)
- [ ] Assertions verify the property invariant
- [ ] Failure messages include the input that failed
- [ ] No heavy external property testing dependencies — prefer standard library

## Traceability Matrix

Complete this matrix for each feature:

| Requirement ID | Contract Spec | Behavior Spec | Property Spec | Status |
|---------------|---------------|---------------|---------------|--------|
| FR-1          | File:Method   | File:Test     | -             | [ ]    |
| FR-2          | File:Method   | File:Test     | File:Prop     | [ ]    |
| NFR-1         | -             | -             | File:Prop     | [ ]    |

## File Organization Checklist

### Directory Structure

```
docs/specifications/<feature>/
├── README.md              [ ] Created
├── contracts/
│   └── *ContractSpec.<ext>   [ ] All interfaces covered
├── behaviors/
│   └── *BehaviorSpec.<ext>   [ ] All FRs covered
└── properties/
    └── *PropertySpec.<ext>   [ ] Key modules covered
```

### README.md Index

- [ ] Lists all specification files
- [ ] Shows modules covered per file
- [ ] Shows requirements traced per file
- [ ] Contains complete traceability matrix
- [ ] Links to architecture and requirements documents

### File Content

Each specification file must have:

- [ ] Package/module declaration matching directory structure
- [ ] Test framework imports (prefer standard/built-in framework)
- [ ] Documentation header with:
  - [ ] Description of what is specified
  - [ ] Reference to architecture document
  - [ ] Reference to requirements document
  - [ ] Traced requirement IDs (for behavior specs)

## Quality Gates

### Minimum Viable Specification

A specification set is **minimally complete** when:

- [ ] All public interfaces have contract specs
- [ ] All functional requirements have behavior tests
- [ ] Architecture and requirements are fully traced

### Production-Ready Specification

A specification set is **production-ready** when all of the above plus:

- [ ] Property tests exist for stateful modules
- [ ] Edge cases are thoroughly tested
- [ ] Error conditions are specified
- [ ] README.md index is complete
- [ ] All checklist items pass

## Review Questions

Before finalizing specifications, answer:

1. **Coverage**: Can every requirement be traced to at least one test?
2. **Clarity**: Can a new developer understand expected behavior from test names alone?
3. **Completeness**: Are error conditions and edge cases specified?
4. **Executability**: Can these specs compile and run (once implementations exist)?
5. **Independence**: Can specs run in any order without affecting each other?
6. **Documentation**: Does the README provide a clear overview of all specifications?

## Common Issues to Check

- [ ] No implementation code in specification files (use TODO placeholders)
- [ ] No heavy external test dependencies beyond the standard test framework
- [ ] No hardcoded test data that should be generated
- [ ] No tests that depend on execution order
- [ ] No tests that depend on external state (files, network, etc.)
- [ ] No overly specific assertions that break on implementation details
