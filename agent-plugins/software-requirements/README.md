# Software Requirements Plugin

Transforms informal feature descriptions into structured software requirements documents.

## Position in the Workflow

This is the first plugin in the specification-driven pipeline.

```
requirements → architecture → specification → implementation
     ↓
  docs/requirements/
```

## Components

- **Command**: `requirements-refine` — start requirements refinement explicitly
- **Agent**: `requirements-refiner` — autonomous requirements analysis
- **Skill**: `requirements-engineering` — knowledge for structuring requirements

## Usage

### Explicit command

```
requirements-refine
```

Then describe what you want to build.

### With an initial description

```
requirements-refine "I want to build a user authentication system..."
```

### Agent activation

The `requirements-refiner` agent can be invoked whenever the user describes a feature, system, or functionality they want to build before designing or implementing it.

## Output

Requirements are saved to `docs/requirements/<feature-name>.md` with these sections:

1. **Overview** — purpose and goals
2. **Functional Requirements** — what the system must do
3. **Non-Functional Requirements** — performance, security, reliability
4. **Constraints** — technical and business limitations
5. **Assumptions** — dependencies and assumed context
6. **Open Questions** — items needing further clarification

## Installation

Copy the `software-requirements/` directory to your project's `.agents/plugins/` directory, or configure your agent system to load it.
