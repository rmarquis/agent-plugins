# Software Implementation Plugin

Generates production-ready implementation from architecture and specification documents.

## Position in the Workflow

This is the fourth plugin in the specification-driven pipeline.

```
requirements → architecture → specification → implementation
                                               ↓
                                         src/ (or language-specific root)
```

## Components

- **Command**: `implementation-generate` — start implementation generation explicitly
- **Agent**: `implementation-generator` — autonomous implementation generation
- **Skill**: `implementation-engineering` — language-aware patterns and TDD guidance

## Usage

### Explicit command

```
implementation-generate
```

The agent will read the architecture and specification documents, select an appropriate language profile, and generate implementation code.

### With a target language

```
implementation-generate --language kotlin
```

### Agent activation

The `implementation-generator` agent can be invoked whenever the user wants to move from design documents to working code, or when refining an existing implementation against its specification.

## Output

Implementation is saved to the project source tree with:

1. **Production code** matching the architecture modules
2. **Unit tests** derived from executable specifications
3. **Integration tests** where specifications describe boundaries
4. **Documentation comments** explaining public APIs

## Language Profiles

The plugin uses language-aware profiles to match conventions:

- Kotlin
- TypeScript
- Python
- Java
- Go
- Rust
- C#

See `skills/implementation-engineering/references/language-profiles.md` for profile details.

## Installation

Copy the `software-implementation/` directory to your project's `.agents/plugins/` directory, or configure your agent system to load it.
