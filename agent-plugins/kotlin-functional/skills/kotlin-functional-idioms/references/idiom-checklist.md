# Idiomatic Kotlin Review Checklist

Use this checklist when reviewing Kotlin code for functional and idiomatic patterns.

## Immutability (25 points)

### Properties (10 points)
- [ ] **10 pts**: 90%+ of properties are `val`
- [ ] **7 pts**: 70-89% of properties are `val`
- [ ] **4 pts**: 50-69% of properties are `val`
- [ ] **0 pts**: <50% of properties are `val`

### Collections (10 points)
- [ ] **10 pts**: All collections are immutable (`List`, `Map`, `Set`)
- [ ] **5 pts**: Most collections immutable, some mutable with justification
- [ ] **2 pts**: Mix of mutable and immutable collections
- [ ] **0 pts**: Primarily mutable collections

### State Transformations (5 points)
- [ ] **5 pts**: State changes use `copy()` or create new instances
- [ ] **3 pts**: Mostly immutable transformations, some in-place mutation
- [ ] **0 pts**: Frequent in-place mutation

**Immutability Score: _____ / 25**

## Expression-Oriented Programming (15 points)

### Control Flow (7 points)
- [ ] **7 pts**: Consistently uses `when` expressions and expression bodies
- [ ] **4 pts**: Mix of expressions and statements
- [ ] **0 pts**: Primarily statement-based

### Loops vs Functional (5 points)
- [ ] **5 pts**: Uses `map`, `filter`, `fold` instead of imperative loops
- [ ] **3 pts**: Mix of functional and imperative loops
- [ ] **0 pts**: Primarily imperative loops with mutable accumulators

### Temporary Variables (3 points)
- [ ] **3 pts**: Minimal temporary variables, uses scope functions
- [ ] **2 pts**: Some temporary variables
- [ ] **0 pts**: Heavy use of temporary variables

**Expression-Oriented Score: _____ / 15**

## Error Handling (20 points)

### Result Types (10 points)
- [ ] **10 pts**: Consistently uses `Result<T>` for expected failures
- [ ] **6 pts**: Mix of Result and nullable return types
- [ ] **3 pts**: Primarily nullable return types
- [ ] **0 pts**: Exception-based flow control

### Error Types (5 points)
- [ ] **5 pts**: Sealed interfaces for domain-specific errors
- [ ] **3 pts**: Some sealed errors, some generic exceptions
- [ ] **0 pts**: Generic exception types only

### Documentation (5 points)
- [ ] **5 pts**: All failure modes documented in KDoc
- [ ] **3 pts**: Some failure modes documented
- [ ] **0 pts**: Failure modes not documented

**Error Handling Score: _____ / 20**

## Null Safety (10 points)

### Nullable Types (4 points)
- [ ] **4 pts**: Nullable types only where semantically necessary
- [ ] **2 pts**: Some unnecessary nullable types
- [ ] **0 pts**: Many unnecessary nullable types

### `!!` Operator (4 points)
- [ ] **4 pts**: Zero uses of `!!` operator
- [ ] **2 pts**: Rare `!!` with clear justification
- [ ] **0 pts**: Frequent or unjustified `!!` usage

### Safe Navigation (2 points)
- [ ] **2 pts**: Consistently uses `?.`, `?:`, and `let` for null handling
- [ ] **1 pt**: Some safe navigation, some null checks
- [ ] **0 pts**: Primarily null checks with `if (x != null)`

**Null Safety Score: _____ / 10**

## Functional Composition (15 points)

### Higher-Order Functions (6 points)
- [ ] **6 pts**: Extensive use of higher-order functions for abstraction
- [ ] **4 pts**: Some higher-order functions
- [ ] **2 pts**: Rare higher-order functions
- [ ] **0 pts**: No higher-order functions

### Extension Functions (5 points)
- [ ] **5 pts**: Rich extension functions for domain operations
- [ ] **3 pts**: Some extension functions
- [ ] **1 pt**: Rare extension functions
- [ ] **0 pts**: No extension functions (utility classes instead)

### Function Composition (4 points)
- [ ] **4 pts**: Functions composed and chained effectively
- [ ] **2 pts**: Some composition
- [ ] **0 pts**: No function composition

**Functional Composition Score: _____ / 15**

## KDoc Documentation (15 points)

### Coverage (5 points)
- [ ] **5 pts**: KDoc on all public declarations
- [ ] **3 pts**: KDoc on most public declarations
- [ ] **1 pt**: KDoc on some public declarations
- [ ] **0 pts**: Minimal or no KDoc

### Contracts (5 points)
- [ ] **5 pts**: Preconditions and postconditions clearly documented
- [ ] **3 pts**: Some contracts documented
- [ ] **1 pt**: Rare contract documentation
- [ ] **0 pts**: No contract documentation

### Quality (5 points)
- [ ] **5 pts**: Comprehensive docs with invariants, side effects, examples
- [ ] **3 pts**: Good descriptions, some additional details
- [ ] **1 pt**: Basic descriptions only
- [ ] **0 pts**: Superficial or "useless" docs (e.g., "Gets the user")

**KDoc Score: _____ / 15**

---

## Overall Score Calculation

| Category | Score | Weight | Weighted Score |
|----------|-------|--------|----------------|
| Immutability | ___/25 | 1.0 | ___ |
| Expression-Oriented | ___/15 | 1.0 | ___ |
| Error Handling | ___/20 | 1.0 | ___ |
| Null Safety | ___/10 | 1.0 | ___ |
| Functional Composition | ___/15 | 1.0 | ___ |
| KDoc Documentation | ___/15 | 1.0 | ___ |
| **Total** | **___/100** | | |

## Grade Mapping

| Score | Grade | Assessment |
|-------|-------|------------|
| 90-100 | Excellent | Exemplary functional and idiomatic Kotlin |
| 80-89 | Good | Strong functional patterns with minor improvements |
| 70-79 | Fair | Decent code with several areas for improvement |
| 60-69 | Poor | Significant non-functional patterns |
| <60 | Needs Improvement | Major refactoring recommended |

## Detailed Sub-Checks

### Immutability Deep Dive

**Data Classes:**
- [ ] All properties are `val`
- [ ] `init` blocks validate invariants
- [ ] State changes use `copy()` or dedicated methods
- [ ] No mutable collections in properties

**Collections:**
- [ ] Uses `listOf()`, `mapOf()`, `setOf()`
- [ ] Transformations return new collections
- [ ] No `mutableListOf()`, `mutableMapOf()`, etc.

**Variables:**
- [ ] Local variables are `val` where possible
- [ ] `var` usage is justified and documented
- [ ] No reassignment in loops (use `fold` instead)

### Expression-Oriented Deep Dive

**When Expressions:**
- [ ] Uses `when` instead of cascading `if`/`else`
- [ ] `when` returns values (expression, not statement)
- [ ] Exhaustive handling for sealed types

**Function Bodies:**
- [ ] Single-expression functions use `=` syntax
- [ ] Avoids `{ return expr }` for single expressions
- [ ] Multi-expression functions use scope functions

**Scope Functions:**
- [ ] Uses `let` for transformations and null handling
- [ ] Uses `also` for side effects in pipeline
- [ ] Uses `apply` for object configuration
- [ ] Uses `run` for multi-statement expressions
- [ ] Uses `with` for operating on context objects

### Error Handling Deep Dive

**Result Usage:**
- [ ] Functions that can fail return `Result<T>`
- [ ] Uses `flatMap` for chaining Result operations
- [ ] Uses `fold` or pattern matching to handle errors
- [ ] No naked `getOrThrow()` without justification

**Sealed Hierarchies:**
- [ ] Domain errors are sealed interfaces
- [ ] Each error case carries relevant data
- [ ] Exhaustive when-expression handling
- [ ] Clear error naming and documentation

**Exception Usage:**
- [ ] Exceptions only for truly exceptional cases
- [ ] No exception-based flow control
- [ ] Exceptions documented with `@throws`

### Null Safety Deep Dive

**Semantic Nullability:**
- [ ] Nullable types only for optional values
- [ ] Uses `Result<T>` instead of nullable returns
- [ ] Non-null types by default
- [ ] No `String?` where `Result<String>` is better

**Safe Operations:**
- [ ] Zero `!!` operators
- [ ] Uses `?.` for safe navigation
- [ ] Uses `?:` Elvis for defaults
- [ ] Uses `let` for null-safe transformations

### Functional Composition Deep Dive

**Higher-Order Functions:**
- [ ] Abstracts patterns with function parameters
- [ ] Uses `inline` for zero overhead
- [ ] Function types as strategy pattern
- [ ] No inheritance for behavior sharing

**Extension Functions:**
- [ ] Domain operations as extensions
- [ ] Chainable operations
- [ ] No utility classes with static methods
- [ ] Extensions grouped logically

**Composition:**
- [ ] Functions composed with `compose` or similar
- [ ] Pipelines use `let`, `map`, `flatMap`
- [ ] Currying/partial application where appropriate
- [ ] Readable left-to-right or top-to-bottom flow

### KDoc Deep Dive

**Required Sections:**
- [ ] Summary (one sentence)
- [ ] Detailed description
- [ ] Preconditions (where applicable)
- [ ] Postconditions (for all public functions)
- [ ] Invariants (for classes)
- [ ] Side effects (if any)
- [ ] Examples (for complex APIs)
- [ ] Cross-references (`@see`, `@property`, `@param`, `@return`)

**Quality Checks:**
- [ ] Explains "why", not just "what"
- [ ] Documents business rules
- [ ] Explains design decisions
- [ ] Includes code examples for complex usage
- [ ] Uses proper markdown formatting
- [ ] Links to related functions/types

## Red Flags (Automatic Deductions)

- **-10 pts**: Any `!!` operator without strong justification
- **-10 pts**: Exception-based flow control (throwing for expected failures)
- **-5 pts**: Mutable collections as public API return types
- **-5 pts**: Missing KDoc on public API surface
- **-5 pts**: Inheritance hierarchies for behavior composition
- **-3 pts**: `var` properties in data classes without justification
- **-3 pts**: Imperative loops where functional would work

## Quick Assessment Mode

For rapid assessment, check these 10 items:

1. [ ] 90%+ properties are `val`
2. [ ] No mutable collections
3. [ ] Uses `when` expressions (not statements)
4. [ ] Result types for failures
5. [ ] Zero `!!` operators
6. [ ] Extension functions present
7. [ ] Higher-order functions used
8. [ ] KDoc on all public APIs
9. [ ] Preconditions/postconditions documented
10. [ ] No exception-based flow control

**Quick Score:**
- **10/10**: Excellent
- **8-9/10**: Good
- **6-7/10**: Fair
- **4-5/10**: Poor
- **<4/10**: Needs Improvement
