# Kotlin Functional Plugin

Generates and reviews functional-first, idiomatic Kotlin implementations with comprehensive KDoc documentation.

## Position in the Workflow

This plugin is a language-specific implementation plugin for Kotlin projects.

```
requirements → architecture → specification → implementation
                                                  ↓
                                        kotlin-functional
                                                  ↓
                                          src/main/kotlin/
```

## Components

- **Commands**:
  - `kotlin-functional-implement` — generate functional Kotlin implementation from specifications
  - `kotlin-functional-review` — review code for functional/idiomatic patterns
- **Agents**:
  - `kotlin-functional-implementer` — autonomous Kotlin implementation generation
  - `kotlin-functional-reviewer` — autonomous Kotlin code review
- **Skill**: `kotlin-functional-idioms` — core functional Kotlin patterns and idioms

## Usage

### Generate implementation

```
kotlin-functional-implement [spec-or-module]
```

### Review code

```
kotlin-functional-review [file-or-dir]
```

## Output

Generated code is written to `src/main/kotlin/<package>/`.
Review reports are saved to `docs/reviews/functional-review-<timestamp>.md`.

## Installation

Copy the `kotlin-functional/` directory to your project's `.agents/plugins/` directory, or configure your agent system to load it.
