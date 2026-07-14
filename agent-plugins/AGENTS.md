# Workflow Orchestration

These rules guide agents when using the `agent-plugins` suite.

## 1. Plan Before Building

- Enter plan mode for any non-trivial task (3+ steps or architectural decisions).
- If the plan breaks, stop and re-plan instead of pushing forward.
- Use planning for verification steps, not just implementation.
- Write detailed specs upfront to reduce ambiguity.

## 2. Use Subagents Liberally

- Offload research, exploration, and parallel analysis to subagents.
- Use one subagent per focused tack.
- Keep the main context window clean.

## 3. Self-Improvement Loop

- After any correction from the user, capture the pattern in `tasks/lessons.md`.
- Write rules that prevent the same mistake.
- Review relevant lessons at session start.

## 4. Verify Before Done

- Never mark a task complete without proving it works.
- Run tests, check logs, and demonstrate correctness.
- Ask: "Would a staff engineer approve this?"

## 5. Demand Elegance (Balanced)

- For non-trivial changes, pause and ask if there is a simpler way.
- If a fix feels hacky, redo it cleanly with full context.
- Skip this for obvious fixes; do not over-engineer.

## 6. Autonomous Problem Solving

- When given a bug report, fix it. Point at logs, errors, or failing tests, then resolve them.
- Fix failing checks without being told how.

## Specification-Driven Workflow

Follow this order unless the user explicitly overrides it:

1. **Requirements** — Capture what the system must do.
2. **Architecture** — Design modules and interfaces.
3. **Specification** — Write executable contracts, behaviors, and properties.
4. **Implementation** — Write code that satisfies the specifications.

## Task Management

1. **Plan first**: Write a checkable plan to `tasks/todo.md`.
2. **Verify the plan**: Confirm scope before implementing.
3. **Track progress**: Mark items complete as you go.
4. **Explain changes**: Provide a high-level summary at each step.
5. **Document results**: Add a review section to `tasks/todo.md`.
6. **Capture lessons**: Update `tasks/lessons.md` after corrections.

## Core Principles

- **Simplicity first**: Make every change as simple as possible.
- **No laziness**: Find root causes, not temporary fixes.
- **Minimal impact**: Touch only what is necessary.
