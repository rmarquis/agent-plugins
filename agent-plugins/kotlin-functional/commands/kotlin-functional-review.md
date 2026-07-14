---
command: kotlin-functional-review
description: Review Kotlin code for functional programming patterns and idiomatic usage
arguments:
  - name: file-or-dir
    description: Path to Kotlin file or directory to review
    required: true
inputs:
  - Kotlin source files
outputs:
  - docs/reviews/functional-review-<timestamp>.md
capabilities:
  - read
  - write
  - ask_user
  - glob
---

Review Kotlin code for functional programming patterns and idiomatic usage.

## Usage

```
kotlin-functional-review <file-or-dir>
```

## Arguments

- `<file-or-dir>`: Path to Kotlin file or directory to review
  - Can be a single file (e.g., `src/main/kotlin/UserService.kt`)
  - Can be a directory (e.g., `src/main/kotlin/userauth`)
  - Can be relative or absolute path

## What This Command Does

1. **Scans Code**
   - Reads all `.kt` files from the specified path
   - Recursively processes directories

2. **Analyzes Code**
   - **Immutability**: Checks for `val` vs `var`, mutable collections
   - **Expression-Oriented**: Checks for statements vs expressions, imperative loops
   - **Error Handling**: Checks for Result types vs exceptions, error documentation
   - **Null Safety**: Checks for `!!` usage, unnecessary nullability
   - **Functional Composition**: Checks for higher-order functions, extensions
   - **KDoc Documentation**: Checks for comprehensive documentation with contracts

3. **Scores Code**
   - Calculates score for each criterion (0-100)
   - Weights scores to calculate overall score
   - Assigns grade (Excellent/Good/Fair/Poor)

4. **Identifies Issues**
   - **Critical**: Correctness issues (null safety violations, race conditions)
   - **Major**: Significant functional/idiomatic violations
   - **Minor**: Style and idiom improvements

5. **Generates Report**
   - Creates `docs/reviews/functional-review-<timestamp>.md`
   - Includes:
     - Summary and scores
     - Issues with severity levels
     - Before/after refactoring examples
     - Positive patterns already in use
     - Recommendations (short/medium/long-term)
     - Refactoring opportunities

6. **Offers Interactive Refactoring**
   - Option to apply suggested refactorings
   - File-by-file approval process
   - Shows diffs before applying changes

## Review Criteria

### 1. Immutability (25% weight)
- Properties declared with `val` (not `var`)
- Immutable collections used
- State transformations via `copy()` or new instances
- No in-place mutation

### 2. Expression-Oriented (15% weight)
- `when` expressions instead of statements
- Single-expression functions
- Minimal temporary variables
- Functional pipelines instead of loops

### 3. Error Handling (20% weight)
- `Result<T>` for expected failures
- Sealed interfaces for domain errors
- No exceptions for flow control
- Documented failure modes

### 4. Null Safety (10% weight)
- Non-null types by default
- No `!!` operator
- Nullable only where semantic
- Result types preferred over nullable returns

### 5. Functional Composition (15% weight)
- Higher-order functions for abstraction
- Extension functions for domain operations
- Function composition and chaining
- No inheritance hierarchies for behavior

### 6. KDoc Documentation (15% weight)
- KDoc on all public declarations
- Preconditions and postconditions documented
- Invariants documented for classes
- Side effects explicitly noted
- Examples for complex APIs

## Output Files

### Review Report
- **Location**: `docs/reviews/functional-review-<timestamp>.md`
- **Format**: Markdown
- **Includes**: Scores, issues, suggestions, recommendations

### Refactoring Log (if changes applied)
- **Location**: `docs/reviews/refactoring-log-<timestamp>.md`
- **Format**: Markdown
- **Includes**: List of applied changes with diffs

## Success Criteria

A successful review:
- Identifies concrete, actionable improvements
- Provides before/after code examples
- Explains rationale for suggestions
- Highlights existing good patterns
- Categorizes issues by severity and effort
- Offers interactive refactoring assistance

## Notes

- **Non-Destructive**: Review process doesn't modify code unless explicitly approved
- **Context-Aware**: Considers existing project patterns and conventions
- **Educational**: Explains *why* patterns are beneficial, not just *what* to change
- **Pragmatic**: Focuses on high-value improvements, not dogmatic purity
- **Positive**: Celebrates good code as much as identifying improvements
