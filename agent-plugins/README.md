# Agent Plugins

A collection of agent-agnostic plugins for specification-driven software development.

These plugins define a structured workflow where implementation is the final step, preceded by requirements, architecture, and executable specifications. They are designed to work with any agent system that can load plugin manifests, commands, agents, and skills.

## Plugins

### 1. software-requirements

Transform informal feature descriptions into structured requirements documents.

- **Command**: `requirements-refine`
- **Agent**: `requirements-refiner`
- **Skill**: `requirements-engineering`
- **Output**: `docs/requirements/<feature-name>.md`

### 2. software-architecture

Design internal software architecture using John Ousterhout's *A Philosophy of Software Design* principles.

- **Command**: `architecture-design`
- **Agent**: `architecture-designer`
- **Skill**: `architecture-design`
- **Output**: `docs/architecture/<feature-name>.md`

### 3. software-specification

Generate executable test specifications (contracts, behaviors, properties) before implementation.

- **Command**: `specification-write`
- **Agent**: `specification-writer`
- **Skill**: `specification-engineering`
- **Output**: `docs/specifications/<feature-name>/`

### 4. software-implementation

Generate production code from specifications. Detects the project's language and adapts idioms accordingly.

- **Command**: `implementation-generate`
- **Agent**: `implementation-generator`
- **Skill**: `implementation-engineering`
- **Output**: project source directory

### 5. kotlin-functional

Generate and review functional-first, idiomatic Kotlin implementations with KDoc.

- **Commands**: `kotlin-functional-implement`, `kotlin-functional-review`
- **Agents**: `kotlin-functional-implementer`, `kotlin-functional-reviewer`
- **Skill**: `kotlin-functional-idioms`
- **Output**: `src/main/kotlin/<package>/`

## Workflow

```
requirements → architecture → specification → implementation
     ↓              ↓               ↓               ↓
  requirements   architecture   specification   implementation
    -refine        -design         -write         -generate
```

For Kotlin projects, the language-specific `kotlin-functional` plugin can be used in place of, or after, `software-implementation`.

## Plugin Structure

Each plugin follows the same layout:

```
<plugin>/
├── README.md          # Plugin documentation
├── plugin.json        # Manifest (commands, agents, skills)
├── commands/          # Command definitions
├── agents/            # Agent personas / system prompts
└── skills/
    └── <skill>/
        ├── SKILL.md            # Reusable knowledge
        ├── references/         # Checklists and guides
        └── examples/           # Worked examples
```

## Installation

### Project-level

Copy a plugin directory into your project's `.agents/plugins/` directory:

```bash
cp -r agent-plugins/software-requirements my-project/.agents/plugins/
```

### Agent-level

Configure your agent to load plugins from this directory. The exact mechanism depends on your agent system; most accept a plugin directory path or a list of plugin manifests.

## Manifest Format

Each `plugin.json` declares the plugin's commands, agents, and skills:

```json
{
  "name": "software-requirements",
  "version": "1.0.0",
  "description": "...",
  "author": { "name": "..." },
  "keywords": ["requirements", "specification"],
  "commands": [
    { "name": "requirements-refine", "path": "commands/requirements-refine.md" }
  ],
  "agents": [
    { "name": "requirements-refiner", "path": "agents/requirements-refiner.md" }
  ],
  "skills": [
    { "name": "requirements-engineering", "path": "skills/requirements-engineering/SKILL.md" }
  ]
}
```

## License

MIT
