# agent-plugins

A collection of agent-agnostic plugins and reusable skills.

## Software development pipeline

The plugins in `agent-plugins/` define a sequential workflow where each step feeds the next:

```
requirements → architecture → specification → implementation
```

1. **`software-requirements`** — Capture what to build.
2. **`software-architecture`** — Decide how to structure it.
3. **`software-specification`** — Write executable contracts and tests.
4. **`software-implementation`** — Generate production code from the specs.

`kotlin-functional` is a language-specific specialization for Kotlin projects; use it alongside or in place of `software-implementation` when writing functional, idiomatic Kotlin.

See [`agent-plugins/README.md`](agent-plugins/README.md) for full plugin details, structure, and installation.

## Writing and speaking skills

- **`elements_of_style/`** — Rewrite, edit, and polish prose for clarity, brevity, and vigor using Strunk & White's *The Elements of Style*.
- **`how_to_speak/`** — Shape presentation decks according to Patrick Winston's MIT *How to Speak* lecture.

## Directory layout

The software plugins are grouped under `agent-plugins/` so the pipeline stays together and can be installed or referenced as a unit. The writing and speaking skills live at the repository root because they are independent, reusable capabilities unrelated to the software workflow.
