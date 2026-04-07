# DSL Specification for `.agent.yml`

## Purpose

This DSL defines the **public contract surface** of a package.

It describes:
- which entities must be available from the package facade,
- where each entity must be implemented inside the package,
- what API each entity exposes,
- what semantic and behavioral requirements apply.

The DSL defines **what must exist** and **what it must mean**.
It does not dictate the full internal implementation design.

---

## High-Level Model

A package contains a `.agent.yml` file.
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

A single `.agent.yml` may describe one or more facade-level entities.

These entities may represent classes, functions, objects, or other public package members.

---

## Top-Level Sections

### `Imports`
Defines imported contract types from other DSL contracts.

Example:

```yaml
Imports:
  - Type: Object
    From: "identity.yaml"
```

Semantics:
- `Type` is the imported contract type name.
- `From` is the source DSL path, relative to the working directory.
- Imported types are external contract dependencies.
- They should be treated as contract-level references, not redefined locally by default.

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

---

## Entity Block Semantics

Each entity block may contain:

- `dest`
- `properties`
- `methods`

### `dest`
`dest` is mandatory.

Example:

```yaml
dest: example.py
```

Semantics:
- defines the required file inside the package where the entity must be implemented,
- the entity must be implemented in that file,
- the entity must also be available from the package facade.

This means `dest` defines a **location obligation** and the contract defines a **facade obligation**.

### `properties`
Properties describe facade-visible properties of the entity.

Example:

```yaml
properties:
  - "Name -> name:Type": |
      what is it
```

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

#### Property description
The multiline text under the property is mandatory.
It describes:
- what the property represents,
- constraints,
- behavior expectations where relevant,
- any semantic requirements that must influence planning and validation.

### `methods`
Methods describe facade-visible callable API.

Example:

```yaml
methods:
  - "method(name: Object) -> void:None": |
      what is it, logic description
```

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

## Meaning of the DSL

The DSL defines:
- required facade entities,
- required API shape,
- required implementation locations,
- behavioral meaning from descriptions,
- external contract dependencies.

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
7. Package boundaries are user-defined and must be preserved.

---

## Planning Consequences

A planning agent should use this DSL to answer:
- What must be present on the facade?
- What must be implemented in which file?
- What is already implemented?
- What is missing?
- What should be added internally to realize the contract?
- What must be tested to prove contract compliance?

---

## Notes for Multi-Entity Contracts

A single `.agent.yml` may describe multiple entities.

Those entities may:
- share a single `dest`,
- point to different `dest` files,
- depend on imported contract types,
- belong to the same facade surface.

This is valid.

There is no global rule that one contract entity must correspond to one implementation file.

The user defines the contract.
The planning and coding agents must preserve it.
