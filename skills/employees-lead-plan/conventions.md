# General Project Conventions for Contract-Driven Package Implementation

## Purpose

This file defines **general, language-agnostic project conventions** for implementing code from package contracts.

These conventions are not tied to a specific programming language.
They define how implementation should be organized relative to:
- package contracts,
- facade boundaries,
- internal decomposition,
- traceability,
- testing.

Language-specific conventions such as syntax, typing style, exception style, or naming mechanics should be defined separately in a language-specific conventions file.

---

## Rule Types

This file contains:
- **Required rules** — must be followed unless an explicit contract or project directive says otherwise
- **Recommended rules** — should be followed when they improve clarity and consistency

When a project contains both general and language-specific conventions:
1. obey the contract first,
2. obey package boundary and facade obligations second,
3. obey explicit project conventions next,
4. obey target-language idioms next.

---

## Scope

These conventions apply inside a single user-defined package.

They govern:
- internal code organization,
- public surface vs internal implementation,
- traceability from contract to code and tests,
- test classification,
- project consistency.

These conventions do **not** authorize:
- creation of new packages,
- redefinition of package boundaries,
- changes to user-defined facade contracts.

---

## Package Internal Structure

### Required
- Implementation may be decomposed into additional internal files and modules inside the current package.
- Internal helper elements may be extracted into dedicated internal modules when this improves clarity, cohesion, reuse, or testability.
- Public facade code and internal support code should remain clearly distinguishable.
- Internal decomposition must preserve the contract surface and required `location` placement.

### Recommended
- Group implementation by responsibility so that contract-facing code remains easy to locate.
- Keep facade-related assembly logic near the package surface, while placing reusable internal details deeper in internal modules.
- Prefer decomposition that makes contract coverage and testing easier to follow.

### Clarification
There is **no** general rule that one public contract entity must map to one implementation file.
Multiple contract entities may validly point to the same `location` if the contract says so.

---

## Public Surface vs Internal Implementation

### Required
- The public contract surface must remain explicit and stable.
- Internal support code may be introduced freely as long as it does not replace or obscure contract entities.
- Internal implementation details must not be treated as a substitute for facade-level contract behavior.
- Planning and coding should clearly distinguish:
  - public contract-facing elements,
  - internal implementation elements.

### Recommended
- Keep contract-facing entry points easy to identify.
- Avoid unnecessary mixing of facade-oriented behavior and private implementation details in the same place when separation improves clarity.

---

## Naming Principles

### Required
- Use naming conventions idiomatic for the target language and consistent with the existing project.
- Keep naming aligned with contract vocabulary wherever possible.
- Use naming that makes public entities, internal support elements, and tests distinguishable by role.

### If the project already has code
- Follow existing project naming patterns unless they directly conflict with the contract.

### If the project has no code yet
- Use naming idiomatic for the target language.
- Use contract vocabulary as the semantic baseline.
- Keep naming consistent across entities, helpers, and tests so the emerging project style starts from a coherent base.

### Recommended
- Prefer names that reflect responsibility and relation to the contract.
- Let internal helper names communicate support intent rather than facade intent.
- Let test names communicate covered entity, scenario, and expected behavior.

---

## Traceability Rules

### Required
Implementation and tests must be traceable back to the contract.

For any meaningful implementation unit or test, it should be possible to determine:
- which contract entity it supports,
- which contract property or method it covers,
- whether it supports facade behavior or internal behavior,
- which described requirement it validates.

### Recommended
- Preserve clear mapping from contract entity → implementation location → validation → tests.
- Make it easy to inspect whether each contract requirement has corresponding implementation and corresponding tests.

---

## Contract-to-Test Mapping

### Required
Every contract element must have explicit test coverage.

At minimum, for each meaningful contract entity and each meaningful contract behavior:
- facade availability must be tested,
- declared API shape must be tested,
- behavior described in the contract must be tested.

Descriptions in the contract are not optional.
If a behavior or constraint is described there, test coverage must reflect it.

### Recommended
- Make contract test coverage easy to audit by grouping tests around entities, methods, or behaviors in a way natural for the target language and project.

---

## Test Classification

Tests must be classified into two categories.

### 1. Contract tests
These verify the package contract.

They should cover:
- facade availability,
- public API shape,
- behavior required by contract descriptions,
- positive scenarios,
- negative scenarios,
- edge cases.

These tests are mandatory.

### 2. Internal tests
These verify internal helpers, extracted modules, or decomposition details.

They are useful when internal complexity justifies direct verification.

### Required rule
Internal tests do **not** replace contract tests.

A package is not sufficiently covered if only helper logic is tested while facade behavior remains unverified.

---

## Test Naming Principles

### Required
Test names should make clear:
- the contract entity or internal element under test,
- the scenario,
- the expected behavior or result.

### Language-agnostic rule
Do not enforce a specific casing style here.
Instead:
- use the naming mechanics natural to the target language and test framework,
- preserve clarity of covered subject, scenario, and expectation.

### Recommended
A good test name should communicate:
- **who/what** is under test,
- **under which condition**,
- **what should happen**.

---

## Recommendations for Internal Decomposition

### Recommended
- Introduce internal helpers when they reduce duplication or isolate behavior cleanly.
- Decompose internal logic when doing so improves contract traceability, readability, or testability.
- Prefer internal structure that makes future contract changes easier to absorb.

### Required boundary
Internal decomposition must remain inside the current package and must not create new packages.

---

## Relationship to Language-Specific Conventions

This file intentionally avoids language-specific rules such as:
- exact naming casing,
- typing mechanics,
- exception mechanics,
- async/sync details,
- import syntax,
- file naming syntax,
- language-specific test framework rules.

Those should be defined in an additional language-specific conventions file when needed.

This general convention layer defines **structural and traceability expectations**, not language syntax policy.

---

## Ralphex Task Design Conventions

### Task atomicity
Each task in the plan must be **atomic** — completable by an AI agent in a single Claude Code session without requiring context from other tasks.

Rules:
- One task covers one logical unit of work (one entity skeleton, one method implementation, one test suite).
- Tasks sharing the same `location` file may be grouped if they are small enough for one session.
- A task must not require the AI to read other tasks for implementation context.
- Each task restates relevant imports, usages, and annotations needed for implementation.

### Task ordering
Tasks must be ordered **per-package**: each package is completed (coding + testing) before moving to the next.

Within a single package, tasks follow this order:
1. Infrastructure (package structure, `__init__.py`, re-exports)
2. Entity skeletons (classes and functions in correct `location` files)
3. Property implementations
4. Method implementations (one per method or grouped by entity)
5. Interface mutations (`Type::` mutation)
6. Contract tests — **must be placed immediately after the last coding task for this package**
7. Integration/edge case tests — **also placed within the same package's block**

When the plan covers multiple packages (e.g., subpackages):
- Process leaf packages first (no child dependencies), then parent packages
- Respect dependency order: if package A imports from package B, complete B first
- Each package's test tasks follow directly after its coding tasks

### Checkbox granularity
Each `- [ ]` checkbox in a task must be:
- A specific, verifiable action (e.g., "Create file `service.py`", "Implement method `load()` returning `list[Any]`")
- Verifiable by running a command or checking file existence
- Not a vague goal (e.g., avoid "Implement the service" without specifics)

### Validation command placement
- Each task must include at least one inline validation step (a `- [ ]` checkbox with a verification command).
- The `## Validation Commands` section at the plan level lists all global validation commands.
- Task-level validation commands verify the specific task outcome.
- Plan-level validation commands verify overall contract compliance.

### Test-after-coding rule
Every **coding task** (implementation, infrastructure, entity creation, method implementation) must include two final checkboxes:
1. `- [ ] Run existing tests to verify no regressions: pytest tests/ -x (skip this step if no test files exist yet)`
2. `- [ ] If any tests fail, fix the code written in this task (not test code) and re-run tests until they pass`

These steps are part of the coding task itself, not separate tasks. The AI agent runs tests and fixes code within the same context window.

Test-writing tasks (creating new test files) do **not** include these steps — they validate via their own `pytest tests/test_<entity>.py -v` step.

### Self-contained task context
Each task must include:
- What contract entities it covers
- What `location` files are involved
- Relevant imports and usages
- Behavioral requirements from descriptions
- Annotations as prescriptive instructions for the implementation agent

This ensures ralphex can send any single task to Claude Code and the AI has sufficient context to implement it correctly.
