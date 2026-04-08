# Output Template for Contract-to-Execution Planning

Use this exact structure for every planning response.

If a section cannot be completed because information is unavailable, keep the section and explicitly state what is unavailable.

---

# LEVEL 1 — OVERVIEW

## Summary
Provide a concise overview covering:
- what must be implemented or changed,
- the most important contract-to-code gaps,
- the main execution phases.

## Contract Surface
Summarize:
- facade entities,
- required `dest` mapping,
- external contract dependencies,
- key facade obligations.

### Re-exports
For each re-export block (`->Name: {}`):
- Name:
- Source (corresponding `Imports` entry):
- Facade obligation: must be importable from `__init__.py`
- Hierarchy constraint: source must be at a lower filesystem level than the current `CODEMANIFEST.yml`

## Main Risks / Ambiguities
List the most important:
- risks,
- ambiguities,
- inferred areas,
- missing workspace or git context.

---

# LEVEL 2 — DETAILED EXECUTION PLAN

## Contract Extraction

For each contract entity, use the following shape.

### Entity: `<entity name>`
**Kind:** `<class | function | re-export | unknown>`

**Facts**
- Declared `dest`:
- Facade obligation:
- Extends: `[Type]` declarations (if any)
- Annotations context: (if present, use as context hint only)
- Properties:
- Methods:
- Semantic requirements from descriptions:
- Imported dependencies used by this entity:

**Assumptions**
- ...

**Open Questions**
- ...

Repeat this subsection for every contract entity.

## Package Boundary

State clearly:
- what belongs to the public contract surface,
- what may remain internal,
- which external dependencies are contract-based,
- what must not be changed or crossed.

## Gap Analysis

Compare the contract with the current visible package state.

Use these subsections when applicable:
- Missing contract entities
- Missing facade exposure
- Wrong `dest` placement
- API mismatches
- Behavioral mismatches
- Existing code that can be reused
- Test coverage gaps
- Missing workspace or git visibility

## Assumptions

List every non-explicit planning inference.

For each assumption use:
- Assumption:
- Basis:
- Criticality:
- Safe to proceed without confirmation: `<yes | no>`

## Open Questions

List unresolved questions.

For each question use:
- Question:
- Why it matters:
- Blocking in strict mode: `<yes | no>`

## Execution Plan

Break the work into phases.

For each phase use:

### Phase `<N>`: `<phase title>`
**Goal**
- ...

**Contract entities covered**
- ...

**Tasks**
- T...

**Expected outcome**
- ...

**Validation checkpoints**
- ...

**Tests required**
- Contract tests:
- Internal tests:

## Detailed Tasks

For each task use:

### Task `<ID>`
**Goal**
- ...

**Target files**
- ...

**Contract entities covered**
- ...

**Allowed internal additions**
- ...

**Implementation steps**
1. ...
2. ...
3. ...

**Result checks**
- ...

**Tests**
- Contract tests:
  - Facade availability:
  - API shape:
  - Behavior:
  - Positive scenarios:
  - Negative scenarios:
  - Edge cases:
- Internal tests:
  - ...

Every task must be specific.
Avoid vague tasks that do not identify files, intent, checks, and test obligations.

## Validation Plan

Explain how to verify final contract compliance.

Cover at least:
- correct implementation location in each `dest`,
- facade availability of each contract entity,
- API compatibility,
- semantic and behavioral alignment,
- imported contract dependency alignment,
- contract test coverage,
- internal test coverage where relevant.

## Done Criteria

Use a checklist.

- [ ] Every contract entity is implemented
- [ ] Every contract entity is implemented in the correct `dest`
- [ ] Every contract entity is available from the package facade
- [ ] Properties and methods match the declared API
- [ ] Descriptions are reflected in behavior
- [ ] Contract dependencies are respected
- [ ] Contract tests cover facade, API, and behavior
- [ ] Internal tests exist where internal decomposition requires them
- [ ] No package boundary has been expanded
- [ ] Assumptions and open questions are explicitly documented
