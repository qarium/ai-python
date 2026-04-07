# DSL Specification for `.agent.yml`

## Purpose

This DSL defines the **public contract surface** of a package.

It describes:
- which entities must be available from the package facade,
- where each entity must be implemented inside the package,
- what API each entity exposes,
- what semantic and behavioral requirements apply,
- which external types are re-exported through the facade.

The DSL defines **what must exist** and **what it must mean**.
It does not dictate the full internal implementation design.

---

## High-Level Model

A package contains a `.agent.yml` file inside the package directory.
That file defines the package contract.

The package is treated as an isolated unit.
Its external interaction is defined through the contract.

The implementation agent may decompose the package internally, but must preserve:
- declared facade entities,
- declared API,
- declared semantics,
- required implementation file locations.

---

## Top-Level Structure

A typical file may look like this:

```yaml
Imports:
  - Type: Object
    From: "identity.yaml"

---

->Object: {}

Example:
  dest: example.py
  properties:
    - "Name -> name:Type": |
        what is it
  methods:
    - "method(name: Object) -> void:None": |
        what is it, logic description
```

A single `.agent.yml` may describe one or more facade-level entities and re-exports.

Entity types:
- **class** — a class with properties and methods (default kind).
- **module** — a module imported as a whole, with functions as methods.
- **function** — a standalone facade-level function.

The kind is implicit unless `kind` is specified explicitly.

---

## Top-Level Sections

### `Imports`
Defines external types used in the contract.

Example:

```yaml
Imports:
  - Type: Object
    From: "identity.yaml"
  - Type: HTTPError
    From: "requests"
```

Semantics:
- `Type` is the external type name.
- `From` is the source:
  - a `.yaml`/`.agent.yml` path for DSL contract dependencies (relative to working directory),
  - a Python package name for external library types (e.g., `"requests"`).
- Imported types are external dependencies.
- They should be treated as contract-level references, not redefined locally by default.
- Importing a type does **not** automatically re-export it. Use the re-export syntax to make it available on the facade.

### `---` separator
Optional YAML document separator. May be used to visually separate `Imports` from entity definitions.

### Entity blocks
Each top-level entity block defines one facade-level contract entity.

Example:

```yaml
Example:
  dest: example.py
  properties:
  methods:
```

`Example` is the contract entity name.

### Re-export blocks
Each top-level re-export block makes an imported type available on the package facade without defining a contract for it.

Syntax:

```yaml
"->TypeName": {}
```

The `->` prefix marks the entity as a re-export.
The empty `{}` body means no contract obligations — the type is passed through as-is.

Example:

```yaml
Imports:
  - Type: HTTPError
    From: "requests"

---

"->HTTPError": {}
```

Semantics:
- The type must be importable from the package facade.
- No `dest`, `properties`, or `methods` are defined — the implementation simply re-exports the type.
- The type is not a contract entity — no implementation obligation exists.
- The planning agent must ensure the type is available from the package `__init__.py`.

---

## Entity Block Semantics

Each entity block may contain:

- `kind` (optional)
- `dest`
- `properties` (optional)
- `methods` (optional)

### `kind`
`kind` is optional. Default value is `class`.

Values:
- `class` — the entity is a class. Properties and methods map to class members.
- `module` — the entity is a module imported as a whole (e.g., `from .utils import url`). Methods map to module-level functions. No `properties` section is expected.
- `function` — the entity is a standalone facade-level function. A single method entry is expected, where the method name equals the entity name.

Example:

```yaml
url:
  kind: module
  dest: utils/url.py
  methods:
    - "join(*parts: str) -> joined:str": |
        Joins URL parts into a single URL string.
```

### `dest`
`dest` is mandatory for all entity kinds.

Example:

```yaml
dest: example.py
```

Semantics:
- defines the required file inside the package where the entity must be implemented,
- the entity must be implemented in that file,
- the entity must also be available from the package facade.

This means `dest` defines a **location obligation** and the contract defines a **facade obligation**.

For `kind: module`, `dest` points to the module file itself.

### `properties`
Properties describe facade-visible data access of the entity.

Example:

```yaml
properties:
  - "Name -> name:Type": |
      what is it
```

Properties section may be omitted when the entity has no facade-visible properties.

#### Property signature format

```text
PropertyName -> label:Type
```

Semantics:
- `PropertyName` = actual property name in code
- `label` = semantic hint used to clarify intent
- `Type` = actual property type

Important:
- the left side is the real code-facing property name,
- the label is explanatory,
- the label does not replace the actual property name.

#### What counts as a property

The properties section covers any facade-visible data accessor, regardless of implementation mechanism:
- `@property` decorated methods,
- public instance attributes (set in `__init__`, no `_` prefix).

From the facade perspective these are indistinguishable — a consumer cannot tell whether `obj.name` is a `@property` or a plain attribute. The contract treats them the same.

#### Property description
The multiline text under the property is mandatory.
It describes:
- what the property represents,
- constraints,
- behavior expectations where relevant,
- any semantic requirements that must influence planning and validation.

### `methods`
Methods describe facade-visible callable API.

For `kind: class` — these are methods of the class.
For `kind: module` — these are module-level functions.
For `kind: function` — this is a single entry matching the entity itself.

Example:

```yaml
methods:
  - "method(name: Object) -> void:None": |
      what is it, logic description
```

Methods section may be omitted when the entity has no facade-visible callable API (rare but valid).

#### Method signature format

```text
method(arg: Type) -> label:ReturnType
```

Semantics:
- `method` = actual method name in code
- `arg: Type` = actual API argument and its type
- `label` = semantic hint describing the return meaning
- `ReturnType` = actual return type

The return label is explanatory and may clarify meaning when the return type alone is too generic or ambiguous.

Example:

```text
get_address(order: Order) -> delivery:Address
```

Here:
- `get_address` is the actual method name,
- `order` is the argument,
- `delivery` explains which kind of address is meant,
- `Address` is the actual return type.

#### Method description
The multiline text under the method is mandatory.
It describes:
- purpose,
- behavior,
- constraints,
- expectations,
- implementation-relevant semantics.

It must be reflected in:
- execution planning,
- validation,
- tests.

---

## Annotation System

The DSL supports YAML comments as annotations for metadata that does not affect the contract structure.

### Inferred descriptions

When a description is inferred from code rather than extracted from a docstring:

```yaml
# NOTE: description inferred — review for accuracy
```

This annotation may appear:
- at the top of the file, if most descriptions are inferred,
- inline before a specific property or method.

Planning agents should treat inferred descriptions as lower-confidence semantic hints and may flag them for user review.

---

## Facade Scope

The package facade is the set of names available via `import package` or `from package import ...`.

Facade exposure may happen at different levels:
- **primary facade** — available from the top-level `__init__.py`,
- **secondary facade** — available from a subpackage `__init__.py` (e.g., `package.submodule`).

Every entity defined in `.agent.yml` (including re-exports) must be available from at least one facade level.

The contract does not currently distinguish primary from secondary facade. Both are valid facade exposure. The planning agent must verify that each entity is importable from the declared facade location.

---

## Meaning of the DSL

The DSL defines:
- required facade entities,
- required API shape,
- required implementation locations,
- behavioral meaning from descriptions,
- external contract dependencies,
- re-exported types.

The DSL does **not** automatically define:
- internal module decomposition,
- helper classes,
- helper functions,
- internal validation layers,
- private abstractions.

Those may be introduced by the implementation agent as internal decisions, as long as the contract is preserved.

---

## Boundary Rules Implied by the DSL

The contract implies the following:

1. The package facade is authoritative.
2. Contract entities must remain available from the facade.
3. Each contract entity must be implemented in its declared `dest`.
4. Internal decomposition is allowed.
5. Internal decomposition must not erase or distort the contract.
6. Imported contract types define external dependencies.
7. Re-exported types must be available from the facade but carry no implementation obligation.
8. Package boundaries are user-defined and must be preserved.

---

## Planning Consequences

A planning agent should use this DSL to answer:
- What must be present on the facade?
- What must be implemented in which file?
- What is already implemented?
- What is missing?
- What should be added internally to realize the contract?
- What must be re-exported without implementation?
- What must be tested to prove contract compliance?

---

## Notes for Multi-Entity Contracts

A single `.agent.yml` may describe multiple entities and re-exports.

Those entities may:
- share a single `dest`,
- point to different `dest` files,
- depend on imported contract types,
- belong to the same facade surface,
- have different `kind` values.

This is valid.

There is no global rule that one contract entity must correspond to one implementation file.

The user defines the contract.
The planning and coding agents must preserve it.
