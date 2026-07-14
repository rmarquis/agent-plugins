# Software Architecture Plugin

Designs internal software architecture from requirements, applying principles from *A Philosophy of Software Design* by John Ousterhout.

## Position in the Workflow

This plugin follows requirements capture and precedes specification.

```
requirements → architecture → specification → implementation
     ↓              ↓
  docs/requirements/  docs/architecture/
```

## Components

- **Command**: `architecture-design` — design architecture from requirements
- **Agent**: `architecture-designer` — autonomous architecture design
- **Skill**: `architecture-design` — Ousterhout design principles

## Usage

```
architecture-design [requirements-file-or-feature-name]
```

If no argument is provided, the command lists available requirements files and asks which to use.

## Output

Architecture documents are saved to `docs/architecture/<feature-name>.md` with:

1. **Overview** — architectural approach and key decisions
2. **Design Principles Applied** — which Ousterhout principles guided the design
3. **Module Structure** — component diagram and per-module responsibilities
4. **Layer Architecture** — abstraction layers and their value
5. **Design Decisions** — options considered, choice, and rationale
6. **Complexity Analysis** — red flags avoided, complexity pulled downward, information hiding achieved
7. **Requirements Traceability** — mapping from requirements to modules

## Installation

Copy the `software-architecture/` directory to your project's `.agents/plugins/` directory, or configure your agent system to load it.
