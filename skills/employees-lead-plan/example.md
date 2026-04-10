# Example: DSL-to-Ralphex Plan Compilation

This example shows how a `CODEMANIFEST` DSL is compiled into a ralphex-compatible execution plan.

---

## Input: CODEMANIFEST

```yaml
Imports:
  - Types:
      - User
    From: "identity"

Usages:
  - pydantic: .specs/pydantic.md

Annotations: |
  User management service with validation.

---

"->User": {}

"UserService()":
  location: service.py
  annotations: |
    Service for creating and managing users.
  methods:
    "create_user(name: str) -> result:User": |
      Creates a new user and validates input before creating the entity.
      Name must be non-empty and contain only alphanumeric characters.
    "get_user(user_id: int) -> user:User": |
      Returns user by ID. Raises ValueError if not found.

"format_user(user: User) -> formatted:str":
  location: formatters.py
  annotations: |
    Formats user data as a display string.
```

---

## Output: Ralphex Plan (`docs/plans/user-service.md`)

```markdown
# Plan: user-service

## Goal

Implement user management service package with:
- `UserService` class in `service.py` with `create_user` and `get_user` methods
- `format_user` standalone function in `formatters.py`
- Re-export `User` type through package facade
- Full contract test coverage

## Context

### Contract Surface

**Entity: `UserService()`**
- Kind: class
- Declared `location`: `service.py`
- Facade obligation: must be importable from package
- Properties: (none declared)
- Methods:
  - `create_user(name: str) -> result:User` — validates input (non-empty, alphanumeric), creates User entity
  - `get_user(user_id: int) -> user:User` — returns user by ID, raises ValueError if not found
- Semantic requirements from descriptions: name validation (non-empty, alphanumeric), ValueError on missing user
- Imported dependencies: `User` from `identity`
- Annotations context: "Service for creating and managing users."

**Entity: `format_user(user: User) -> formatted:str`**
- Kind: function
- Declared `location`: `formatters.py`
- Facade obligation: must be importable from package
- Semantic requirements from descriptions: formats user data into display string
- Imported dependencies: `User` from `identity`
- Annotations context: "Formats user data as a display string."

### Re-exports
- `User` — from `identity` via `Imports`, must be importable from `__init__.py`
- Hierarchy constraint (for `Imports` only): source must be at a lower filesystem level

### Usages Context
- `pydantic` — spec at `.specs/pydantic.md`, may be used for data validation

### External Dependencies
- `User` type from `identity` (internal contract dependency)
- `pydantic` from `Usages` (external library for data validation)

## Facts
- Package facade must expose `UserService`, `format_user`, and `User`
- `UserService` must be in `service.py`
- `format_user` must be in `formatters.py`
- `User` is an imported contract dependency, not locally defined
- `create_user` must validate name is non-empty and alphanumeric
- `get_user` must raise `ValueError` when user not found

## Assumptions
- Assumption: `User` from `identity` is a class with at least `name` and `id` attributes
- Basis: method signatures reference `User` as return type and parameter
- Criticality: medium
- Safe to proceed without confirmation: yes

## Open Questions
- (none in default mode)

## Gap Analysis
- No existing code in package — all entities need implementation from scratch
- No existing tests — full test suite needed
- No existing `__init__.py` — needs creation

---

## Tasks

### Task 1: Create package structure and facade

Set up the package directory structure and `__init__.py` to expose all contract entities on the facade. The package must make `UserService`, `format_user`, and re-exported `User` importable.

- [ ] Create `__init__.py` with imports for `UserService` (from `service.py`), `format_user` (from `formatters.py`), and re-export `User` from its source
- [ ] Create empty `service.py` and `formatters.py` placeholder files
- [ ] Verify facade availability: `python -c "from package import UserService, format_user, User"`
- [ ] Run existing tests to verify no regressions: `pytest tests/ -x` (skip this step if no test files exist yet)
- [ ] If any tests fail, fix the code written in this task (not test code) and re-run tests until they pass

### Task 2: Implement UserService skeleton in service.py

Implement the `UserService` class in `service.py` with constructor and method stubs. This class must be available from the package facade. `User` is imported from `identity` contract — do not redefine it locally.

Annotations context: "Service for creating and managing users."

- [ ] Implement `UserService` class in `service.py` with empty `__init__` constructor
- [ ] Add internal storage (e.g., `_users: dict[int, User]`) for user lookup by ID
- [ ] Add `create_user(name: str) -> User` method skeleton
- [ ] Add `get_user(user_id: int) -> User` method skeleton
- [ ] Verify facade availability: `python -c "from package import UserService; svc = UserService()"`
- [ ] Run existing tests to verify no regressions: `pytest tests/ -x` (skip this step if no test files exist yet)
- [ ] If any tests fail, fix the code written in this task (not test code) and re-run tests until they pass

### Task 3: Implement create_user method with validation

Implement the `create_user` method behavior. The method must:
- Validate that `name` is non-empty
- Validate that `name` contains only alphanumeric characters
- Create a `User` entity with validated input
- Store the user for later retrieval
- Return the created `User`

- [ ] Implement name validation: reject empty strings and non-alphanumeric characters
- [ ] Create `User` instance with validated name
- [ ] Store user in internal storage with generated ID
- [ ] Return created `User` instance
- [ ] Verify basic creation: `python -c "from package import UserService; svc = UserService(); u = svc.create_user('Alice'); print(u)"`
- [ ] Run existing tests to verify no regressions: `pytest tests/ -x` (skip this step if no test files exist yet)
- [ ] If any tests fail, fix the code written in this task (not test code) and re-run tests until they pass

### Task 4: Implement get_user method with error handling

Implement the `get_user` method behavior. The method must:
- Accept `user_id: int`
- Return the matching `User` if found
- Raise `ValueError` if no user with given ID exists

- [ ] Implement user lookup by ID from internal storage
- [ ] Raise `ValueError` with descriptive message when user not found
- [ ] Return matching `User` instance when found
- [ ] Verify retrieval: `python -c "from package import UserService; svc = UserService(); u = svc.create_user('Bob'); print(svc.get_user(u.id))"`
- [ ] Run existing tests to verify no regressions: `pytest tests/ -x` (skip this step if no test files exist yet)
- [ ] If any tests fail, fix the code written in this task (not test code) and re-run tests until they pass

### Task 5: Implement format_user function in formatters.py

Implement `format_user(user: User) -> str` in `formatters.py`. This is a standalone function (not a class method). It takes a `User` and returns a formatted display string.

Annotations context: "Formats user data as a display string."

- [ ] Implement `format_user(user: User) -> str` in `formatters.py`
- [ ] Format user data into readable string representation
- [ ] Verify facade availability: `python -c "from package import format_user; print(format_user(user))"`
- [ ] Run existing tests to verify no regressions: `pytest tests/ -x` (skip this step if no test files exist yet)
- [ ] If any tests fail, fix the code written in this task (not test code) and re-run tests until they pass

### Task 6: Contract tests for UserService

Create contract tests verifying facade availability, API shape, and behavioral requirements for `UserService`.

- [ ] Create test file `tests/test_user_service.py`
- [ ] Test facade availability: `from package import UserService`
- [ ] Test API shape: `UserService` has `create_user` and `get_user` methods
- [ ] Test positive `create_user`: valid name returns `User` instance
- [ ] Test negative `create_user`: empty name raises error
- [ ] Test negative `create_user`: non-alphanumeric name raises error
- [ ] Test positive `get_user`: existing user ID returns correct `User`
- [ ] Test negative `get_user`: non-existent user ID raises `ValueError`
- [ ] Run validation: `pytest tests/test_user_service.py -v`

### Task 7: Contract tests for format_user

Create contract tests verifying facade availability, API shape, and behavior for `format_user`.

- [ ] Create test file `tests/test_format_user.py`
- [ ] Test facade availability: `from package import format_user`
- [ ] Test API shape: `format_user` is callable with `(user: User) -> str`
- [ ] Test positive: returns non-empty string for valid `User`
- [ ] Test edge: handles `User` with unusual characters in name
- [ ] Run validation: `pytest tests/test_format_user.py -v`

---

## Validation Commands

- `python -c "from package import UserService, format_user, User"`: Verify all facade entities are importable
- `pytest tests/ -x`: Run all tests
- `ruff check src/`: Lint check

---

## Done Criteria

- [ ] `UserService` is implemented in `service.py`
- [ ] `format_user` is implemented in `formatters.py`
- [ ] `User` is re-exported from package facade
- [ ] All entities are importable from package `__init__.py`
- [ ] `create_user` validates name (non-empty, alphanumeric)
- [ ] `get_user` raises `ValueError` when user not found
- [ ] Contract tests cover all facade entities and behaviors
- [ ] All validation commands pass
- [ ] No new packages created outside the current package boundary
```

---

## Key Compilation Patterns

### Pattern 1: Infrastructure-first
Task 1 sets up the package structure and facade before any implementation.

### Pattern 2: Entity skeleton before behavior
Task 2 creates the class skeleton, Tasks 3-4 add method behavior one at a time.

### Pattern 3: Standalone functions as separate tasks
Task 5 implements `format_user` independently since it has its own `location`.

### Pattern 4: Tests follow coding tasks per-package
Tasks 6-7 are test tasks placed **immediately after the last coding task for the same package**. In a multi-package plan, each package's tests would follow its coding tasks before starting the next package.

**Single package example** (this case): all coding → all tests (tests naturally follow because there is only one package).
**Multi-package example** (conceptual):
```
Package A: Task 1 (infra) → Task 2 (entity) → Task 3 (methods) → Task 4 (tests for A)
Package B: Task 5 (infra) → Task 6 (entity) → Task 7 (tests for B)
Package C (depends on A,B): Task 8 (infra) → Task 9 (entity) → Task 10 (tests for C)
```

### Pattern 5: Each task is self-contained
Every task includes enough context for an AI agent to implement it without reading other tasks.

### Pattern 6: Validation after each task
Every task includes at least one verification command.

---

## Anti-Patterns to Avoid

Bad plan would:
- Put all implementation in one giant task
- Mix test writing with implementation in the same task
- Use vague steps like "implement the service" without specifics
- Omit validation commands from tasks
- Create tasks that require reading previous tasks for context
- Forget re-export obligations
- Ignore `location` placement requirements
