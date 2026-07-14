---
command: implementation-generate
description: Generate implementation code from architecture and specification documents
arguments:
  - name: language
    description: Target programming language (optional; inferred from project if omitted)
    required: false
  - name: spec-path
    description: Path to the specification document to implement (optional; discovers latest if omitted)
    required: false
inputs:
  - docs/architecture/*.md
  - docs/specifications/*.md
  - Existing project source files
outputs:
  - src/**/* (production code)
  - tests/**/* (test code)
  - docs/implementation-notes.md (optional)
capabilities:
  - read
  - write
  - ask_user
---

Generate production-ready implementation from the project's architecture and specification documents.

## Process

1. **Discover Inputs**
   - Read `docs/architecture/` for module boundaries and design decisions.
   - Read `docs/specifications/` for contracts, behaviors, and properties.
   - Inspect existing project structure to match conventions.
   - If `--language` is provided, use it; otherwise infer from existing source files.

2. **Select Language Profile**
   - Load the matching language profile from `implementation-engineering/references/language-profiles.md`.
   - Apply project conventions from existing code (naming, layout, dependencies).

3. **Plan Implementation**
   - Map specification sections to source modules/files.
   - Identify pure functions, services, repositories, controllers, and boundaries.
   - Decide test strategy (unit, property-based, integration).

4. **Generate Code**
   - Write production code in the appropriate directories.
   - Include doc comments for public APIs.
   - Follow the language profile for formatting, error handling, and idioms.

5. **Generate Tests**
   - Translate acceptance criteria into test cases.
   - Write property-based tests where specifications include invariants.
   - Include negative and edge-case tests.

6. **Save and Summarize**
   - Save files to the project source tree.
   - Present a summary of generated files and key decisions.
   - Note any ambiguities or missing specifications.

Use the `implementation-engineering` skill for language-specific patterns and TDD guidance.
