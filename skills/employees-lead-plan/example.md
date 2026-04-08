# Example: Contract-to-Execution Planning

This example shows how a planning agent should interpret a small contract and derive an execution-oriented plan.

---

## Input Contract

```yaml
Imports:
  - Type: User
    From: "identity.agent.yml"

---

"UserService()":
  dest: service.py
  methods:
    "create_user(name: str) -> result:User": |
      Creates a new user and validates input before creating the entity.
```

---

## Expected Contract Interpretation

### Facts
- The package facade must expose `UserService`.
- `UserService` must be implemented in `service.py`.
- `UserService.create_user(name: str) -> User` is part of the facade API.
- The return label `result` clarifies the semantic role of the returned `User`.
- `User` is an imported contract dependency.
- The method description requires:
  - input validation,
  - entity creation behavior.

### Likely assumptions in default mode
- Validation may be implemented directly in `service.py` or delegated to an internal helper.
- If complexity grows, an internal validator helper is a reasonable decomposition.
- Tests should validate both successful creation and invalid input handling.

### Open questions in strict mode
- What qualifies as invalid input?
- Must duplicate names be handled?
- Is persistence required or is creation purely in-memory?

---

## Expected Planning Consequences

A strong plan should:
- preserve `UserService` on the package facade,
- implement it in `service.py`,
- include a task for validation design,
- include a task for creation behavior,
- define contract tests for:
  - facade availability,
  - API shape,
  - valid input,
  - invalid input,
  - edge cases such as empty or malformed names,
- optionally define internal tests if helper extraction is planned.

---

## Minimal Example of Good Task Decomposition

### Task T1
**Goal**
- Implement `UserService` in `service.py` and expose it on the package facade.

**Checks**
- `UserService` exists in `service.py`
- `UserService` is available from the package facade

**Tests**
- Contract tests:
  - facade availability
  - API shape for `create_user`
- Internal tests:
  - none yet

### Task T2
**Goal**
- Implement `create_user(name: str) -> User` behavior with input validation.

**Checks**
- valid input creates a `User`
- invalid input is handled according to project or contract rules

**Tests**
- Contract tests:
  - positive: valid name returns `User`
  - negative: invalid name is rejected
  - edge: empty, whitespace-only, or borderline valid names
- Internal tests:
  - validator helper tests if validation is extracted

---

## Example of Bad Planning

Bad planning would:
- move `UserService` to a different file than `service.py`,
- fail to expose it on the facade,
- ignore input validation from the description,
- test only internal validation helper behavior but not the facade contract,
- treat imported `User` as a locally redefined type without reason.
