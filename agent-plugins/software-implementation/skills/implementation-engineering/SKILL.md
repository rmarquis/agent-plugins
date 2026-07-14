---
name: implementation-engineering
description: |
  Use this skill when generating implementation code from architecture and specifications,
  writing tests from executable specs, choosing language-specific idioms, or refining
  existing code to match a target language profile. Covers language conventions, TDD,
  code quality, and boundary implementation.
version: 1.0.0
---

# Implementation Engineering

Turn architecture and specification documents into clean, tested, production-ready code using language-aware profiles and test-driven workflows.

## Core Process

### 1. Understand the Inputs

Before writing code, read:

- **Architecture document**: module boundaries, interfaces, dependencies, complexity goals.
- **Specification document**: contracts, behaviors, properties, acceptance criteria.
- **Existing codebase**: conventions, patterns, dependencies, folder layout.

### 2. Select a Language Profile

Use `references/language-profiles.md` to match the target language's conventions for:

- Directory layout and file naming
- Build and test commands
- Idiomatic constructs
- Error handling
- Naming conventions
- Documentation style

### 3. Plan the Implementation

Map specification items to code:

| Specification Item | Typical Code Artifact |
|--------------------|----------------------|
| Domain entity/value object | Model/entity class or record |
| Business rule/invariant | Domain service or pure function |
| Repository contract | Interface + adapter implementation |
| Use case / behavior | Application service |
| API endpoint | Controller/handler |
| Error condition | Custom error type or exception |
| Acceptance criterion | Test case |

### 4. Test-Driven Development

1. Convert acceptance criteria into failing tests.
2. Write the minimum production code to pass each test.
3. Refactor while keeping tests green.
4. Add property-based tests for invariants.
5. Add integration tests for boundaries.

### 5. Write Production Code

Follow these principles:

- **Single Responsibility**: Each function/class does one thing well.
- **Open/Closed**: Extend behavior without modifying existing code.
- **Dependency Inversion**: Depend on abstractions (interfaces/ports).
- **Explicit Error Handling**: Use language idioms (Result, Optional, errors, etc.).
- **Immutability**: Prefer immutable data where practical.
- **Purity**: Push side effects to boundaries; keep domain logic pure.

## Code Quality Checklist

- [ ] Public APIs are documented.
- [ ] Functions are short and focused.
- [ ] Error paths are tested.
- [ ] Edge cases are covered.
- [ ] No hard-coded secrets or environment-specific values.
- [ ] No obvious performance traps (N+1 queries, excessive allocations).
- [ ] Existing style and naming conventions are followed.

## Testing Guidelines

### Unit Tests

- One test class/file per production class/function.
- Test behavior, not implementation details.
- Use descriptive test names.
- Arrange-Act-Assert structure.

### Property-Based Tests

- Identify invariants from specifications.
- Generate random valid inputs.
- Verify the invariant holds.

### Integration Tests

- Test repository/database adapters.
- Test HTTP endpoints or messaging boundaries.
- Use test containers, in-memory fakes, or mocks as appropriate.

## Boundary Implementation

When implementing boundaries (I/O, external APIs, persistence):

1. Define a port/interface in the domain/application layer.
2. Implement the adapter in the infrastructure layer.
3. Keep the adapter thin; most logic should be in domain services.
4. Make the adapter testable with fakes or mocks.

## Refactoring Guidance

When refining existing code:

1. Run existing tests to establish a baseline.
2. Make small, focused changes.
3. Run tests after each change.
4. Stop when the code meets the specification without gold-plating.

## Additional Resources

### Reference Files

- **`references/language-profiles.md`** — Conventions by programming language

### Example Files

- **`examples/repository-implementation.md`** — Implementing a repository contract in multiple languages
