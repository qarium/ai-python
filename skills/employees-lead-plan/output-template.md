# Output Template for DSL-to-Ralphex Planning

Use this exact structure for every planning response.
This format is compatible with ralphex execution.

If a section cannot be completed because information is unavailable, keep the section and explicitly state what is unavailable.

---

# Plan: `<feature-name>`

## Goal

Concise statement of what will be implemented or changed.
Cover:
- what the package must provide after implementation,
- the most important contract-to-code gaps,
- the overall implementation strategy.

## Context

### Contract Surface

For each contract entity:

**Entity: `<entity name>`**
- Kind: `<class | function | re-export>`
- Declared `location`: `<file path>`
- Facade obligation: must be importable from `<package>`
- Mutations: `Type::` declarations (if any)
- Properties: (list with types and descriptions)
- Methods: (list with signatures and descriptions)
- Semantic requirements from descriptions: (key behavioral expectations)
- Imported dependencies: (types from `Imports` used by this entity)
- Annotations context: (if present)

Repeat for every contract entity.

### Re-exports

For each re-export block (`->Name: {}` or `->usage.Type: {}`):
- Name:
- Source: corresponding `Imports` entry (for internal types, resolved from `Types` list with optional `AS` aliases) or `Usages` entry (for external types)
- Facade obligation: must be importable from `__init__.py`
- Hierarchy constraint (for `Imports` only): source must be at a lower filesystem level

### Usages Context

For each usage entry:
- Name:
- Description or spec reference:
- Relevance to implementation:

### External Dependencies

List all external dependencies the implementation relies on:
- External library types from `Usages` (third-party packages)
- Pattern/convention references from `Usages`
- Required tools or frameworks

## Facts

List all facts directly stated in the contract or observed in the workspace:
- ...

## Assumptions

List every non-explicit planning inference:
- Assumption:
- Basis:
- Criticality:
- Safe to proceed without confirmation: `<yes | no>`

## Open Questions

List unresolved questions:
- Question:
- Why it matters:
- Blocking in strict mode: `<yes | no>`

## Gap Analysis

Compare the contract with the current visible package state:
- Missing contract entities:
- Missing facade exposure:
- Wrong `location` placement:
- API mismatches:
- Behavioral mismatches:
- Existing code that can be reused:
- Test coverage gaps:
- Missing workspace or git visibility:

---

## Tasks

> **Per-package ordering rule**: Each package's coding tasks are followed immediately by its test tasks. When the plan covers multiple packages, complete one package (coding + tests) before starting the next.

<!-- For each package, repeat this block: -->

### Task 1: `<descriptive title>`

<Context paragraph: what this task does, which contract entities it covers, relevant imports/usages/annotations. Provide enough context for an AI agent to implement this task independently.>

**CRITICAL: `CODEMANIFEST` files are read-only contract definitions. Do NOT modify them. If the implementation does not match the contract, fix the implementation — never fix the contract.**

- [ ] <specific implementation step 1 — e.g., "Create file `path/to/location.py`">
- [ ] <specific implementation step 2 — e.g., "Implement class `EntityName` with constructor accepting `(param: Type)`">
- [ ] <specific implementation step 3 — e.g., "Add `EntityName` to `package/__init__.py`">
- [ ] <specific implementation step N>
- [ ] Verify facade availability: `python -c "from package import EntityName"`
- [ ] Run existing tests to verify no regressions: `pytest tests/ -x` (skip this step if no test files exist yet)
- [ ] If any tests fail, fix the code written in this task (not test code) and re-run tests until they pass

### Task 2: `<descriptive title>`

<Context paragraph>

- [ ] <step>
- [ ] <step>
- [ ] <step>
- [ ] Run existing tests to verify no regressions: `pytest tests/ -x` (skip this step if no test files exist yet)
- [ ] If any tests fail, fix the code written in this task (not test code) and re-run tests until they pass

Continue for all coding tasks **for this package**.

### Task N: Contract tests for `<entity or scope>`

<Context: what contract aspects are being tested for this package>

- [ ] Create test file `tests/test_<entity>.py`
- [ ] Test facade availability: `from package import Entity`
- [ ] Test API shape: verify properties and methods exist with correct signatures
- [ ] Test positive scenario: `<specific behavior from description>`
- [ ] Test negative scenario: `<specific error case>`
- [ ] Test edge case: `<specific boundary condition>`
- [ ] Run validation: `pytest tests/test_<entity>.py -v`

<!-- Repeat the entire block (coding tasks + test tasks) for the next package -->

---

## Validation Commands

- `<command>`: <what it verifies>
- `<command>`: <what it verifies>
- `pytest tests/ -x`: Run all tests
- `python -c "from package import Entity1, Entity2"`: Verify all facade entities are importable

---

## Done Criteria

- [ ] Every contract entity is implemented in the correct `location`
- [ ] Every contract entity is available from the package facade
- [ ] Properties and methods match the declared API
- [ ] Descriptions are reflected in behavior
- [ ] Contract dependencies are respected
- [ ] Re-exports are available from facade
- [ ] Contract tests cover facade, API, and behavior
- [ ] Internal tests exist where internal decomposition requires them
- [ ] No package boundary has been expanded
- [ ] No `CODEMANIFEST` files were modified (read-only contract)
- [ ] All validation commands pass
- [ ] Assumptions and open questions are explicitly documented
