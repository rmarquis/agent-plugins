---
name: architecture-design
description: |
  Use this skill when the user asks to design architecture, design software, design modules,
  reduce complexity, create deep modules, apply information hiding, review module depth,
  discuss shallow modules, pull complexity down, define errors out of existence, design it twice,
  or mentions Ousterhout or 'A Philosophy of Software Design'. Provides John Ousterhout's
  principles for creating software with minimal complexity.
version: 1.0.0
---

# Ousterhout Design Principles

Apply principles from *A Philosophy of Software Design* by John Ousterhout to create software architectures that minimize complexity and maximize developer productivity.

## Core Philosophy

**Complexity is the root of all software problems.** The primary goal of software design is managing complexity — making systems easier to understand, modify, and maintain. Complexity accumulates from many small decisions; reducing it requires constant vigilance.

## The Five Core Principles

### 1. Deep Modules Over Shallow Modules

A module's depth is the ratio of functionality to interface complexity.

**Deep modules** provide powerful functionality behind simple interfaces:
- Hide implementation complexity from callers
- Offer significant capability with minimal cognitive load
- Examples: Unix file I/O (5 calls do everything), TCP/IP socket interface

**Shallow modules** expose complexity without hiding it:
- Interface nearly as complex as implementation
- Force callers to understand internal details
- Red flag: methods that just pass through to other methods

**Design for depth:**
- Maximize what the module does
- Minimize what callers need to know
- Ask: "Can I make this interface simpler while keeping the functionality?"

### 2. Information Hiding

Each module should encapsulate design decisions that are likely to change:
- Data structures and algorithms
- Lower-level implementation details
- Platform-specific behavior
- External service interactions

**Information leakage** (a red flag) occurs when:
- Multiple modules share knowledge of the same design decision
- Changes ripple across module boundaries
- Implementation details appear in interfaces

**Apply information hiding:**
- Identify decisions likely to change
- Encapsulate each decision in one module
- Expose behavior through interfaces, not implementation

### 3. Different Layer, Different Abstraction

Each layer in a software system should provide a distinctly different abstraction:

**Good layering:**
- Each layer transforms the problem representation
- Lower layers handle details, upper layers handle concepts
- Clear separation of concerns between layers

**Pass-through methods** indicate bad layering:
- Method does little except call another method with similar signature
- Indicates missing or wrong abstraction level
- Adjacent layers may need consolidation

**Design distinct layers:**
- Define what abstraction each layer provides
- Ensure each layer adds genuine value
- Eliminate pass-through patterns

### 4. Pull Complexity Downward

When complexity is unavoidable, push it into implementation rather than interfaces:

**Pulling down complexity:**
- Handle edge cases internally rather than requiring callers to handle them
- Provide sensible defaults instead of requiring configuration
- Make common cases simple, even if uncommon cases become more complex internally

**The principle in practice:**
- Module developers should suffer for their users
- A little extra complexity in one place saves complexity in many places
- Ask: "Can I handle this case internally rather than exposing it?"

### 5. Define Errors Out of Existence

Reduce complexity by designing interfaces that cannot produce errors:

**Techniques:**
- Change semantics so "errors" become valid operations
- Handle edge cases internally with defined behavior
- Use default values instead of requiring explicit values

**Examples:**
- Substring beyond string end: return empty string (not error)
- Delete non-existent item: succeed silently (idempotent)
- Read from empty queue: block or return null (not exception)

**Design error-free interfaces:**
- Question every exception: is it truly necessary?
- Define behavior for all inputs
- Make operations idempotent where possible

## Design Process: Design It Twice

Before committing to an architecture, consider at least two fundamentally different approaches:

1. **Generate alternatives**: Create genuinely different designs, not variations.
2. **Compare trade-offs**: Evaluate each against requirements.
3. **Learn from comparison**: Even rejected designs teach what matters.
4. **Combine insights**: The best design may combine elements.

This principle applies at all scales: system architecture, module design, API design.

## Red Flags for Complexity

Watch for these warning signs when reviewing architecture:

| Red Flag | Symptom | Solution |
|----------|---------|----------|
| **Shallow modules** | Interface as complex as implementation | Consolidate or redesign for depth |
| **Information leakage** | Same knowledge in multiple places | Centralize in one module |
| **Pass-through methods** | Method just calls similar method | Eliminate layer or add value |
| **Pass-through variables** | Variable passed through many layers | Consolidate or use context object |
| **Conjoined methods** | Can't understand A without understanding B | Make each independently understandable |
| **Excessive configuration** | Many knobs for every behavior | Provide sensible defaults |
| **Special-case code** | Many if statements for edge cases | Define edge cases out of existence |

## Module Depth Analysis

Evaluate each module's depth score:

**Deep (Target):**
- Simple, stable interface
- Significant hidden complexity
- Callers need minimal knowledge
- Changes rarely affect interface

**Medium (Acceptable):**
- Moderate interface complexity
- Some hidden complexity
- Callers need some understanding
- Occasional interface changes

**Shallow (Refactor):**
- Interface matches implementation complexity
- Little hidden complexity
- Callers need detailed knowledge
- Frequent interface changes

## Applying Principles to Architecture

When designing architecture from requirements:

1. **Identify core abstractions**: What are the fundamental concepts?
2. **Define module boundaries**: Where should information be hidden?
3. **Design interfaces first**: What's the simplest interface for each module?
4. **Evaluate depth**: Is each module providing significant hidden functionality?
5. **Check for red flags**: Review against the complexity indicators.
6. **Design it twice**: Consider alternative module decompositions.

## Additional Resources

### Reference Files

- **`references/module-patterns.md`** — Common patterns for deep module design
- **`references/complexity-checklist.md`** — Detailed checklist for complexity review

### Example Files

- **`examples/deep-vs-shallow.md`** — Concrete examples of deep vs shallow modules
