# Software Specification Plugin

Generates executable test specifications (contracts, behaviors, properties) from architecture and requirements documentation. Specifications serve as formal contracts that must pass before implementation begins.

## Position in the Workflow

This plugin follows architecture design and precedes implementation.

```
requirements → architecture → specification → implementation
                              ↓
                        docs/specifications/
```

## Components

- **Command**: `specification-write` — generate specs from architecture and requirements
- **Agent**: `specification-writer` — autonomous specification generation
- **Skill**: `specification-engineering` — patterns for contract, behavior, and property-based specs

## Usage

```
specification-write [architecture-file-or-module-name]
```

If no argument is provided, the command lists available architecture files and asks which to use.

## Output

Specifications are saved to `docs/specifications/<feature-name>/`:

```
docs/specifications/<feature-name>/
├── README.md                        # Specification index and traceability
├── contracts/
│   └── <Module>ContractSpec.<ext>   # Interface contract specs
├── behaviors/
│   └── <Feature>BehaviorSpec.<ext>  # BDD-style behavior specs
└── properties/
    └── <Module>PropertySpec.<ext>   # Property-based specs
```

## Language Adaptation

The plugin detects the project's primary language from source files and build manifests, then adapts spec syntax to the project's standard test framework. Supported languages include Kotlin, Java, TypeScript, JavaScript, Python, Go, Rust, C#, Ruby, and Swift.

## Installation

Copy the `software-specification/` directory to your project's `.agents/plugins/` directory, or configure your agent system to load it.
