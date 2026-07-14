# Complexity Review Checklist

Use this checklist when reviewing architecture designs for complexity issues.

## Module Interface Review

### Interface Simplicity

- [ ] Can the interface be described in one sentence?
- [ ] Are there 5 or fewer primary methods/operations?
- [ ] Do method names clearly describe what they do?
- [ ] Are parameters minimal and meaningful?
- [ ] Can a new developer understand the interface in minutes?

### Interface Stability

- [ ] Would common implementation changes require interface changes?
- [ ] Are implementation details hidden from the interface?
- [ ] Is the interface designed around "what" not "how"?
- [ ] Are optional parameters used for advanced features?

### Interface Consistency

- [ ] Do similar operations have similar interfaces?
- [ ] Are naming conventions consistent?
- [ ] Are return types consistent for similar operations?
- [ ] Is error handling consistent across methods?

## Information Hiding Review

### Encapsulation Check

For each module, identify what should be hidden:

- [ ] **Data structures**: Are internal data structures hidden?
- [ ] **Algorithms**: Are algorithm choices encapsulated?
- [ ] **External dependencies**: Are external APIs abstracted?
- [ ] **Configuration**: Are configuration details internal?
- [ ] **State management**: Is state hidden from callers?

### Information Leakage Detection

- [ ] Do multiple modules share knowledge of the same design decision?
- [ ] Would changing an implementation detail require changes to other modules?
- [ ] Do interfaces expose implementation-specific types?
- [ ] Are there constants/enums shared across module boundaries?
- [ ] Do error messages reveal internal implementation details?

## Depth Analysis

### Per-Module Assessment

For each module, evaluate:

| Criterion | Deep | Medium | Shallow |
|-----------|------|--------|---------|
| Interface vs implementation ratio | 1:10+ | 1:5 | 1:2 or worse |
| Lines of interface code | <50 | 50-200 | >200 |
| Knowledge required to use | Minimal | Moderate | Extensive |
| Time to understand | Minutes | Hours | Days |

### Depth Score Questions

- [ ] Does this module hide significant complexity?
- [ ] Can callers use it without understanding internals?
- [ ] Does it provide more value than it costs to use?
- [ ] Is the implementation substantially larger than the interface?

## Red Flag Detection

### Pass-Through Patterns

- [ ] **Pass-through methods**: Do any methods just call another method with same/similar parameters?
- [ ] **Pass-through classes**: Do any classes just wrap another class without adding value?
- [ ] **Pass-through variables**: Are variables passed through multiple layers unchanged?

### Conjoined Code

- [ ] Can each module be understood independently?
- [ ] Are there circular dependencies between modules?
- [ ] Do changes to one module frequently require changes to another?
- [ ] Is there duplicated logic across modules?

### Configuration Issues

- [ ] Are there too many configuration options?
- [ ] Are sensible defaults provided?
- [ ] Is configuration validated and normalized internally?
- [ ] Can the common case be used without configuration?

### Error Handling Issues

- [ ] Are exceptions necessary or could errors be defined out?
- [ ] Is error handling pushed to callers unnecessarily?
- [ ] Are errors specific and actionable?
- [ ] Could operations be made idempotent?

## Layer Review

### Abstraction Levels

For each layer, answer:
- What abstraction does this layer provide?
- How is it different from adjacent layers?
- What complexity does it hide?

### Layer Justification

- [ ] Does each layer add a meaningful abstraction?
- [ ] Are there unnecessary intermediate layers?
- [ ] Is the progression of abstraction logical?
- [ ] Do layers align with natural problem boundaries?

## Complexity Distribution

### Pulling Complexity Down

- [ ] Is complexity in implementations rather than interfaces?
- [ ] Do modules handle edge cases internally?
- [ ] Are common use cases simple even if uncommon cases are complex?
- [ ] Would "suffering" in one module save suffering in many places?

### Complexity Hotspots

Identify modules with highest complexity:
- [ ] Are complex modules providing proportional value?
- [ ] Can complexity be redistributed or eliminated?
- [ ] Are complex modules well-tested and documented?
- [ ] Is complexity isolated from spreading?

## Design Decision Review

### Design It Twice

- [ ] Were alternative approaches considered?
- [ ] Are trade-offs documented?
- [ ] Was the simplest viable approach chosen?
- [ ] Are rejected alternatives noted for future reference?

### Future-Proofing

- [ ] Are likely changes easy to accommodate?
- [ ] Are unlikely changes not over-engineered for?
- [ ] Is the design simple enough to be replaced if wrong?
- [ ] Are extension points provided where variability is expected?

## Final Quality Gates

### Overall Architecture

- [ ] Can the full system be explained in a few sentences?
- [ ] Is the module count appropriate (not too many, not too few)?
- [ ] Are dependencies minimal and well-structured?
- [ ] Is there a clear separation of concerns?

### Cognitive Load

- [ ] Can a developer understand one module without reading all modules?
- [ ] Are there fewer than 7±2 primary concepts to hold in mind?
- [ ] Is the architecture diagram simple and clear?
- [ ] Can new team members onboard in reasonable time?

### Maintainability

- [ ] Are changes likely to be localized?
- [ ] Is testing straightforward at each level?
- [ ] Are components independently deployable (if applicable)?
- [ ] Is the design documented appropriately?

## Scoring Guide

Rate each area and compute overall score:

| Rating | Description |
|--------|-------------|
| 5 | Excellent - exemplifies Ousterhout principles |
| 4 | Good - minor improvements possible |
| 3 | Acceptable - some complexity issues |
| 2 | Concerning - significant complexity problems |
| 1 | Critical - major redesign needed |

**Overall score interpretation**:
- 4.0-5.0: Ship it
- 3.0-3.9: Address issues before implementation
- 2.0-2.9: Significant rework needed
- 1.0-1.9: Fundamental redesign required
