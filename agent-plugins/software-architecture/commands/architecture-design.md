---
command: architecture-design
description: Design software architecture from requirements using Ousterhout's principles
arguments:
  - name: requirements-file-or-feature-name
    description: Optional requirements file or feature name to design from
    required: false
inputs:
  - docs/requirements/<feature-name>.md
  - Answers to clarifying design questions
outputs:
  - docs/architecture/<feature-name>.md
capabilities:
  - read
  - write
  - ask_user
---

Design internal software architecture from requirements documentation, applying principles from *A Philosophy of Software Design* by John Ousterhout.

## Process

1. **Locate Requirements**
   - If an argument is provided, find the matching requirements file in `docs/requirements/`.
   - If no argument is provided, list available requirements files and ask which to use.
   - Read and analyze the requirements document.

2. **Identify Architectural Concerns**
   From the requirements, extract:
   - Core domain concepts and entities
   - Key operations and workflows
   - Integration points and boundaries
   - Performance and scale requirements
   - Security boundaries

3. **Ask Clarifying Questions**
   Clarify design decisions with the user:
   - Technology stack preferences (if not constrained)
   - Deployment model (monolith, microservices, serverless)
   - Critical trade-offs (consistency vs availability, simplicity vs flexibility)
   - Existing system constraints

4. **Design Module Structure**
   Apply Ousterhout principles to design modules:
   - **Deep modules**: maximize functionality behind simple interfaces
   - **Information hiding**: encapsulate decisions likely to change
   - **Different layer, different abstraction**: each layer adds value
   - Use the `architecture-design` skill for guidance.

5. **Generate Architecture Document**
   Create `docs/architecture/<feature-name>.md` with:

   ```markdown
   # [Feature] Architecture

   ## Overview
   [1–2 paragraphs: architectural approach and key decisions]

   ## Design Principles Applied
   [Which Ousterhout principles guided this design and why]

   ## Module Structure

   [Mermaid component diagram showing modules and relationships]

   ### Module: [Name]
   - **Responsibility**: [Single, clear responsibility]
   - **Interface**: [Public API — kept simple]
   - **Hidden Complexity**: [What this module encapsulates]
   - **Depth Score**: [Deep/Medium/Shallow with rationale]

   [Repeat for each module]

   ## Layer Architecture
   [Describe abstraction layers and what each provides]

   ## Design Decisions
   | Decision | Options Considered | Choice | Rationale |
   |----------|-------------------|--------|-----------|

   ## Complexity Analysis
   - Red flags avoided
   - Complexity pulled downward
   - Information hiding achieved

   ## Requirements Traceability
   | Requirement | Implementing Module(s) |
   |-------------|----------------------|
   ```

6. **Include Mermaid Diagrams**
   Generate a component diagram:

   ```mermaid
   graph TB
       subgraph "Public Interface"
           API[API Layer]
       end
       subgraph "Core Modules"
           M1[Module 1]
           M2[Module 2]
       end
       API --> M1
       API --> M2
   ```

7. **Offer Class-Level Detail**
   After presenting module-level architecture, ask:
   "Would you like me to drill into class-level design for any modules?"

   If yes, for selected modules:
   - Generate a Mermaid class diagram
   - Define interfaces and key classes
   - Document method signatures
   - Explain information hiding at class level

8. **Review with User**
   Present an architecture summary and ask if adjustments are needed.
   Make revisions as requested.

## Tips

- Start with the simplest design that meets requirements.
- Question every module: does it provide enough depth?
- Look for information leakage between modules.
- Consider "design it twice" — briefly note alternative approaches considered.
