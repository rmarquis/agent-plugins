---
command: specification-write
description: Generate executable test specifications from architecture and requirements documents
arguments:
  - name: architecture-file-or-module-name
    description: Optional architecture file or module name to specify
    required: false
inputs:
  - docs/architecture/<feature-name>.md
  - docs/requirements/<feature-name>.md
outputs:
  - docs/specifications/<feature-name>/<README.md, contracts/, behaviors/, properties/>
capabilities:
  - read
  - write
  - ask_user
  - glob
---

Generate executable test specifications from architecture and requirements documentation. Specifications serve as formal contracts that must pass before implementation begins.

## Process

1. **Locate Architecture Document**
   - If an argument is provided, find the matching architecture file in `docs/architecture/`.
   - If no argument is provided, list available architecture files and ask which to use.
   - Read and analyze the architecture document, focusing on module structure and interfaces.

2. **Locate Requirements Document**
   - Find the corresponding requirements file in `docs/requirements/`.
   - Read and extract acceptance criteria from functional requirements.
   - Note non-functional requirements that imply property invariants.

3. **Detect Project Language and Test Framework**
   - Scan the project root for language markers:
     - File extensions (`.kt`, `.java`, `.ts`, `.js`, `.py`, `.rs`, `.go`, `.cs`, `.rb`, `.swift`, etc.)
     - Build files (`build.gradle.kts`, `build.gradle`, `pom.xml`, `package.json`, `Cargo.toml`, `go.mod`, `pyproject.toml`, `setup.py`, `Gemfile`, `Package.swift`, `*.csproj`, `CMakeLists.txt`, etc.)
   - Infer the primary language and default test framework from discovered markers.
   - If ambiguous (e.g., multiple languages present), confirm with the user.
   - Record the detected language, test framework, file extension, and naming conventions for use in subsequent steps.

4. **Identify Specification Targets**
   For each module in the architecture:
   - Extract interface methods and their responsibilities.
   - Map acceptance criteria from requirements to module behaviors.
   - Identify invariants and properties (roundtrip, idempotence, commutativity, state validity).

5. **Ask Clarifying Questions**
   Resolve with the user:
   - Which modules to prioritize for specification
   - Package naming conventions for the project
   - Any additional invariants or edge cases to cover
   - Whether to generate all three levels (contracts, behaviors, properties) or a subset
   - Target language and test framework (if not auto-detected)

6. **Generate Contract Specifications**
   For each module interface, create `docs/specifications/<feature-name>/contracts/<ModuleName>ContractSpec.<ext>`:

   ```
   // File: <ModuleName>ContractSpec.<ext>
   // Package/module: specifications.<feature>.contracts

   // Import: test framework assertions (assertEquals, assertNotNull, assertNull, assertThrows)

   /**
    * Contract specification for <ModuleName>.
    *
    * Verifies the formal contract each implementation must satisfy.
    *
    * Architecture: docs/architecture/<feature-name>.md
    * Module: <ModuleName>
    */
   class <ModuleName>ContractSpec:
       // Replace with real implementation when available
       sut: <InterfaceName> = unimplemented("Provide implementation")

       test "method returns expected result for valid input":
           // Arrange
           // Act
           // Assert

       test "method throws for invalid input":
           assertThrows(IllegalArgumentError):
               // Act with invalid input
   ```

7. **Generate Behavior Specifications**
   For each set of acceptance criteria, create `docs/specifications/<feature-name>/behaviors/<Feature>BehaviorSpec.<ext>`:

   ```
   // File: <Feature>BehaviorSpec.<ext>
   // Package/module: specifications.<feature>.behaviors

   // Import: test framework assertions (assertEquals, assertTrue)

   /**
    * Behavior specification for <Feature>.
    *
    * Derived from acceptance criteria in requirements.
    *
    * Requirements: docs/requirements/<feature-name>.md
    * Traces: FR-1, FR-2, ...
    */
   class <Feature>BehaviorSpec:

       test "given <precondition> when <action> then <expected outcome>":
           // Given
           // When
           // Then
   ```

8. **Generate Property-Based Specifications**
   For identified invariants, create `docs/specifications/<feature-name>/properties/<ModuleName>PropertySpec.<ext>`:

   ```
   // File: <ModuleName>PropertySpec.<ext>
   // Package/module: specifications.<feature>.properties

   // Import: test framework assertions, standard library random

   /**
    * Property-based specification for <ModuleName>.
    *
    * Verifies invariants that must hold for all inputs.
    *
    * Architecture: docs/architecture/<feature-name>.md
    * Module: <ModuleName>
    */
   class <ModuleName>PropertySpec:
       random = seededRandom(42)  // deterministic for reproducibility

       test "roundtrip property - save then find returns equivalent":
           repeat 100 times:
               // Generate random input
               // Perform operation
               // Assert invariant holds
   ```

9. **Include Generator Functions**
   When property specs need random data, include lightweight generators:

   ```
   // Generators using standard library random (no external dependencies)
   function randomAlphaString(length: 1..50) -> String:
       // generate random alphabetic string of given length

   function randomEmail() -> String:
       // generate random email address
       // e.g., "{randomAlphaString(3..10)}@{randomAlphaString(3..8)}.{randomAlphaString(2..3)}"
   ```

10. **Add Traceability Comments**
    Every specification file must include:
    - Reference to the source architecture document and module
    - Reference to the source requirements document and requirement IDs
    - Clear mapping between test names and acceptance criteria

11. **Create Specification Index**
    Generate `docs/specifications/<feature-name>/README.md`:

    ```markdown
    # [Feature] Specifications

    ## Contract Specifications
    | File | Module | Methods Covered |
    |------|--------|----------------|

    ## Behavior Specifications
    | File | Requirements Traced | Acceptance Criteria |
    |------|--------------------|--------------------|

    ## Property Specifications
    | File | Module | Properties Verified |
    |------|--------|-------------------|

    ## Traceability Matrix
    | Requirement | Contract Spec | Behavior Spec | Property Spec |
    |-------------|--------------|---------------|---------------|
    ```

12. **Review with User**
    Present a summary of the generated specifications and ask if adjustments are needed.
    Make revisions as requested.

## Tips

- Start with contract specs — they define the minimum interface obligations.
- Behavior specs should use language directly from acceptance criteria.
- Not every module needs property specs — focus on stateful operations and data transformations.
- Use the language's unimplemented/todo placeholder for implementations that don't exist yet.
- Keep generators simple using the language's standard random library.

Use the `specification-engineering` skill for guidance on specification patterns and property testing techniques.
