---
command: requirements-refine
description: Transform informal feature descriptions into structured requirements
arguments:
  - name: initial-description
    description: Optional starting description of the feature or system
    required: false
inputs:
  - User description of the feature or system
  - Answers to clarifying questions
outputs:
  - docs/requirements/<feature-name>.md
capabilities:
  - read
  - write
  - ask_user
---

Start the requirements refinement process for a software feature or system.

If an initial description was provided, analyze it. Otherwise, ask the user to describe what they want to build.

## Process

1. **Gather Initial Description**
   - If arguments are provided, use them as the starting point.
   - Otherwise, prompt the user: "Please describe the feature or system you want to build."

2. **Analyze the Description**
   - Identify core functionality and goals.
   - Note implied requirements.
   - Find ambiguities and vague terms.
   - List missing information.

3. **Ask Clarifying Questions**
   - Ask 3–5 focused questions.
   - Prioritize questions that unblock multiple requirements.
   - Cover users/roles, scale, integrations, error handling, and security.
   - Offer concrete options when possible.

4. **Generate Requirements Document**
   - Create structured markdown with these sections:
     - Overview: brief description and goals
     - Functional Requirements: FR-1, FR-2, etc. with description, acceptance criteria, priority
     - Non-Functional Requirements: performance, security, reliability
     - Constraints: technical and business limitations
     - Assumptions: dependencies and assumed context
     - Open Questions: items needing further clarification
   - Use priority levels: Must Have, Should Have, Nice to Have.
   - Include specific, measurable acceptance criteria.

5. **Save the Document**
   - Create `docs/requirements/` directory if it does not exist.
   - Save as `docs/requirements/<feature-name>.md`.
   - Use kebab-case filename derived from the feature name.

6. **Review with User**
   - Present a summary of the generated requirements.
   - Ask if any sections need revision.
   - Make adjustments as requested.

Use the `requirements-engineering` skill for guidance on structuring requirements and question patterns.
