---
name: specification-engineering
description: |
  Use this skill when the user asks to write specifications, create test specs, specify interfaces,
  perform contract testing, write behavior specifications, use property-based testing, BDD specs,
  given-when-then, specification-first, contract-first, executable contracts, test specifications,
  test before implementation, formal contracts, or interface contracts. Provides patterns for
  contract, behavior, and property-based specifications using the project's test framework.
version: 1.0.0
---

# Executable Specifications

Write executable test specifications that serve as formal contracts, behavioral documentation, and property invariants. Specifications are written BEFORE implementation and define what the system must do.

Adapt all patterns below to the project's language and test framework.

## Core Philosophy

**Specification-first development** means:

1. Architecture defines module structure and interfaces.
2. Requirements define acceptance criteria and constraints.
3. **Specifications formalize both into executable tests.**
4. Implementation satisfies the specifications.

Specifications bridge the gap between design documents and working code. They are the source of truth for what an implementation must do.

## Deriving Specifications from Architecture + Requirements

### Mapping Architecture to Specifications

| Architecture Artifact | Specification Type |
|----------------------|-------------------|
| Module interface | Contract specification class |
| Interface method | Contract test methods |
| Module responsibility | Behavior specification class |
| Module interaction | Integration contract spec |
| Design constraint | Property specification |

### Mapping Requirements to Specifications

| Requirement Artifact | Specification Type |
|---------------------|-------------------|
| Functional requirement | Behavior spec test(s) |
| Acceptance criterion | Individual behavior test |
| Non-functional requirement | Property or contract test |
| Error handling rule | Contract error test |
| Business invariant | Property-based test |

### Derivation Process

1. **Read architecture document** — extract modules, interfaces, responsibilities.
2. **Read requirements document** — extract FRs, NFRs, acceptance criteria.
3. **For each module interface**: create a contract spec class with tests for each method.
4. **For each acceptance criterion**: create a behavior spec test with Given-When-Then.
5. **For each invariant**: create a property spec test with random generation.

## Three Specification Levels

### Level 1: Interface Contract Specifications

Contract specs define the obligations of each interface method.

**What to specify:**
- Pre-conditions: what must be true before calling
- Post-conditions: what must be true after calling
- Return guarantees: what the return value must satisfy
- Error conditions: when and what exceptions/errors are produced
- Idempotency: whether repeated calls produce the same result

**Structure:**

```
// File: <Interface>ContractSpec.<ext>
// Package/module: specifications.<feature>.contracts

// Import: test framework assertions (assertEquals, assertNotNull, assertThrows, etc.)

// Contract specification for [InterfaceName].
//
// Architecture: docs/architecture/<feature>.md
// Module: [ModuleName]

class <Interface>ContractSpec:

    sut: <Interface> = unimplemented("Provide implementation")

    // --- Method: save ---

    test "save returns entity with generated id":
        input = createValidEntity()
        result = sut.save(input)
        assertNotNull(result.id)

    test "save throws for invalid entity":
        invalid = createInvalidEntity()
        assertThrows(IllegalArgumentException):
            sut.save(invalid)

    // --- Method: findById ---

    test "findById returns null for non-existent id":
        result = sut.findById(nonExistentId())
        assertNull(result)

    // --- Method: delete ---

    test "delete is idempotent":
        id = existingEntityId()
        sut.delete(id)
        sut.delete(id)  // second call should not throw
```

**Guidelines:**
- One spec class per interface
- Group tests by method using comments
- Test the happy path first, then error conditions
- Use helper methods for test data creation
- Name tests as statements of what must be true

### Level 2: Behavior Specifications (BDD-Style)

Behavior specs verify acceptance criteria using Given-When-Then structure.

**What to specify:**
- User-facing behaviors from acceptance criteria
- Workflow steps from functional requirements
- Error scenarios from edge case requirements

**Structure:**

```
// File: <Feature>BehaviorSpec.<ext>
// Package/module: specifications.<feature>.behaviors

// Import: test framework assertions

// Behavior specification for [Feature].
//
// Requirements: docs/requirements/<feature>.md
// Traces: FR-1, FR-2, FR-3

class <Feature>BehaviorSpec:

    // FR-1: User Registration
    test "given valid credentials when user registers then account is created":
        // Given
        credentials = validCredentials()

        // When
        result = registrationService.register(credentials)

        // Then
        assertTrue(result.isSuccess)
        assertNotNull(result.accountId)

    // FR-1: User Registration - duplicate check
    test "given existing email when user registers then registration fails with duplicate error":
        // Given
        existing = existingUserCredentials()

        // When
        result = registrationService.register(existing)

        // Then
        assertTrue(result.isFailure)
        assertEquals("EMAIL_DUPLICATE", result.errorCode)
```

**Guidelines:**
- Test names read as natural language sentences
- Comment trace requirement ID above each test or group
- Given section sets up preconditions
- When section performs exactly one action
- Then section verifies postconditions
- One spec class per feature or user story

### Level 3: Property-Based Specifications

Property specs verify invariants that must hold for all inputs using random generation.

**What to specify:**
- Roundtrip properties: encode then decode yields equivalent
- Idempotence: applying operation twice equals applying once
- Commutativity: order of operands doesn't matter (where applicable)
- Invariant preservation: state remains valid after any operation sequence

**Structure:**

```
// File: <Module>PropertySpec.<ext>
// Package/module: specifications.<feature>.properties

// Import: test framework assertions, standard library random

// Property-based specification for [ModuleName].
//
// Architecture: docs/architecture/<feature>.md
// Requirements: docs/requirements/<feature>.md
// Module: [ModuleName]

class <Module>PropertySpec:

    random = seededRandom(42)  // deterministic for reproducibility

    // --- Generators ---

    function randomAlphaString(length: 1..50) -> String:
        // generate random alphabetic string of given length

    function randomPositiveInt(max) -> Int:
        // generate random positive integer

    // --- Properties ---

    test "roundtrip - save then findById returns equivalent entity":
        for 100 iterations:
            entity = randomValidEntity()
            saved = sut.save(entity)
            found = sut.findById(saved.id)
            assertEquals(saved, found)

    test "idempotence - delete called twice has same effect as once":
        for 100 iterations:
            entity = randomValidEntity()
            saved = sut.save(entity)
            sut.delete(saved.id)
            sut.delete(saved.id)  // should not throw

    test "invariant - entity count never negative after operations":
        for 100 iterations:
            count = sut.count()
            assertTrue(count >= 0, "Entity count must never be negative")
```

**Guidelines:**
- Use a deterministic seed for reproducible tests
- Iterate each property 100 times for lightweight verification
- Write generators using the language's standard random library
- Prefer the standard library over external property testing dependencies
- Name tests as property statements: "roundtrip - ...", "idempotence - ...", "invariant - ..."

## Common Assertion Patterns

Most test frameworks provide these assertion types:

| Assertion | Purpose |
|-----------|---------|
| `assertEquals(expected, actual)` | Equality check |
| `assertNotEquals(a, b)` | Inequality check |
| `assertNotNull(value)` | Non-null check |
| `assertNull(value)` | Null/nil/None check |
| `assertTrue(condition)` | Boolean truth |
| `assertFalse(condition)` | Boolean falsity |
| `assertThrows(ExceptionType, block)` | Expected exception |
| `assertContains(collection, element)` | Collection membership |

Adapt names to your framework (e.g., `pytest.raises`, `expect(...).toBe(...)`, `assert.Equal`).

## Test Organization

Group tests by scenario or method for readability:

- **By method**: Group all tests for `save()` together, then `findById()`, etc.
- **By scenario**: Group "valid input" tests, "invalid input" tests, "not found" tests
- **Nesting**: Use your framework's grouping mechanism (nested classes, `describe`/`context` blocks, sub-tests) to create a hierarchy

## Test Data Helpers

- **Factory methods**: Create functions like `validUser()` and `invalidUser()` with sensible defaults
- **Parameterized defaults**: Allow overriding individual fields for specific test scenarios
- **Random generators**: For property tests, write small generator functions using the standard library's random module

## Test Lifecycle

- Each test should start with clean state
- Use your framework's setup/teardown mechanism (e.g., `@BeforeEach`, `setUp()`, `beforeEach`, `t.Cleanup()`)
- Prefer creating fresh instances over resetting shared state

## Specification Quality Checklist

### Contract Completeness
- [ ] Every public method has at least one contract test
- [ ] Happy path tested for each method
- [ ] Error conditions tested for each method
- [ ] Null/empty input behavior specified
- [ ] Return type guarantees verified
- [ ] Idempotency tested where relevant

### Behavior Coverage
- [ ] Every acceptance criterion has a corresponding test
- [ ] Given-When-Then structure consistently applied
- [ ] Test names readable as specification sentences
- [ ] Requirement IDs traced in comments
- [ ] Error scenarios from requirements covered

### Property Quality
- [ ] Roundtrip properties for all serialization/persistence operations
- [ ] Idempotence properties for all delete/cleanup operations
- [ ] State invariants verified after operation sequences
- [ ] Generators produce valid domain objects
- [ ] Deterministic seeds used for reproducibility

### Traceability
- [ ] Every spec file references its source architecture document
- [ ] Every behavior test traces to a requirement ID
- [ ] Module names match architecture document
- [ ] README.md index is complete and accurate

## Additional Resources

### Reference Files

- **`references/specification-checklist.md`** — Detailed completeness and quality checklist
- **`references/property-testing-guide.md`** — Property identification and lightweight generator patterns

### Example Files

- **`examples/repository-contract-spec.md`** — Language-neutral contract spec for a repository interface
- **`examples/behavior-spec.md`** — Language-neutral BDD-style behavior spec
- **`examples/property-based-spec.md`** — Language-neutral property-based spec with generators
