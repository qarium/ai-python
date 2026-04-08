# DSL Specification for `CODEMANIFEST`

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

Signatures may be described in free form, close to the target programming language,
to help AI models navigate the contract more effectively.

---

## High-Level Model

A package may contain `CODEMANIFEST` files at different levels of the directory tree.

- **Package root** `CODEMANIFEST` — defines the top-level facade contract.
- **Subpackage** `CODEMANIFEST` — defines the contract for a subpackage.

Each `CODEMANIFEST` describes the entities and functions implemented at its level.

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
  - Module: helpers
    From: "some_pkg.utils"

Usages:
  - importlib: .specs/importlib.md
  - structures: .specs/structures.md
  - pattern: |
       Annotation describing how to use the `pattern` alias.

Annotations: |
  Description of this package or subpackage.

---

"->Object": {}
"->helpers": {}

"Loader(path: str, cls_prefix: str = 'Test', method_prefix: str = 'test_')":
  dest: loader.py
  annotations: |
    Description of the Loader entity.
  methods:
    "load() -> cls_instances:list[Any]": |
      Returns a list of loaded class instances.

"Object::structures.Data::SomeClass()":
  dest: some.py
  annotations: |
    Class with data model representation.
  properties:
    name: |
      `str` -> string with the name.

"fibonacci(n: int) -> number:int":
  dest: tools.py
  annotations: |
    Fibonacci number computation.
```

A single `CODEMANIFEST` may contain:
- an `Imports` section (external types and modules used in signatures),
- a `Usages` section (external dependencies, patterns, conventions),
- file-level `Annotations` (optional, describes the module or subpackage as a whole),
- re-export blocks (pass-through types and modules),
- entity blocks (classes with properties and/or methods),
- standalone function blocks (top-level blocks without properties or methods).

---

## Top-Level Sections

### `Imports`
Defines external types and modules used in the contract — literally imports from other `CODEMANIFEST` files or external packages.

Example:

```yaml
Imports:
  - Type: Object
    From: "identity.yaml"
  - Type: HTTPError
    From: "requests"
  - Module: url
    From: "some_pkg.utils"
```

Semantics:
- **`Type` import**: a type used in signatures but not locally defined.
  - `Type` is the external type name.
  - `From` is the source:
    - a `CODEMANIFEST` path for DSL contract dependencies (relative to working directory),
    - a Python package name for external library types (e.g., `"requests"`).
- **`Module` import**: a subpackage or module whose contract is defined in another `CODEMANIFEST`.
  - `Module` is the module/subpackage name.
  - `From` is the Python import path to the parent package (e.g., `"some_pkg.utils"`).
- Imported types are external dependencies.
- Importing a type or module does **not** automatically re-export it. Use the re-export syntax to make it available on the facade.

### `Usages`
Defines external dependencies, patterns, and conventions — anything that explains how to use an external resource in the package.

This is broader than just libraries. A usage entry can be:
- a library spec (how to install, configure, and use its API),
- a convention or pattern (how to follow a specific design approach),
- any usage guidance relevant to the implementation.

Example:

```yaml
Usages:
  - importlib: .specs/importlib.md
  - structures: .specs/structures.md
  - pattern: |
       Annotation describing how to use the `pattern` alias.
```

Semantics:
- Each entry key is the **usage name** (library name, pattern name, convention name).
- The value is either:
  - a **path** to a spec file with detailed documentation (relative to the `CODEMANIFEST` file location),
  - a **multiline text** annotation describing the usage directly.
- `Usages` provides **context only** — it does not create facade entities or import obligations. It informs the implementation agent about external dependencies and how to work with them.
- `Usages` is separate from `Imports`. `Imports` declares types and modules used in contract signatures. `Usages` describes external resources the implementation depends on.

### `Annotations` (file-level)
Optional. Provides structured metadata about the `CODEMANIFEST` file as a whole.

Example:

```yaml
Annotations: |
  Description of this package or subpackage.
  May include design rationale, notes, or caveats.
```

Semantics:
- Placed after `Usages` and before the `---` separator.
- Contains free-form text metadata about the module or subpackage.
- Does **not** define contract obligations.
- Persists through YAML parsing unlike `#` comments.

### `---` separator
Optional YAML document separator. Used to visually separate `Imports`, `Usages`, and `Annotations` from entity definitions.

### Re-export blocks
Each top-level re-export block makes an imported type or module available on the package facade without defining a contract for it.

Syntax:

```yaml
"->Name": {}
```

The `->` prefix marks the entity as a re-export.
The empty `{}` body means no contract obligations — the type or module is passed through as-is.

Example:

```yaml
Imports:
  - Type: HTTPError
    From: "requests"
  - Module: url
    From: "some_pkg.utils"

---

"->HTTPError": {}
"->url": {}
```

Semantics:
- The type or module must be importable from the package facade.
- No `dest`, `properties`, or `methods` are defined.
- The type or module is not a contract entity — no implementation obligation exists.
- The planning agent must ensure the name is available from the package `__init__.py`.
- Re-exports can only embed entities from files at lower levels in the filesystem hierarchy relative to the current `CODEMANIFEST`.

### Entity blocks
Each top-level entity block defines one facade-level class.

The entity name can be:
- a **simple class name** — for classes with no constructor parameters or when constructor details are not part of the contract,
- a **constructor signature** — the full signature including parameters, types, and default values.

Example (simple name):

```yaml
Example:
  dest: example.py
  properties:
  methods:
```

Example (constructor signature):

```yaml
"Loader(path: str, cls_prefix: str = 'Test', method_prefix: str = 'test_')":
  dest: loader.py
  annotations: |
    Recursively loads classes from the specified directory and creates instances
    with method names by prefix.
  methods:
    "load() -> cls_instances:list[Any]": |
      Returns a list of loaded class instances.
```

`Loader` or `Example` is the contract entity name. It represents a class with properties and methods.

Constructor parameters in the entity name serve as documentation for the implementation agent. They describe how the class should be instantiated but are not enforced as individual contract obligations — the entity as a whole is the contract unit.

### Standalone function blocks
A top-level entity block without `properties` or `methods` represents a standalone function or functor.

The entity name is the full function signature:

```yaml
"fibonacci(n: int) -> number:int":
  dest: tools.py
  annotations: |
    Fibonacci number computation.
```

Whether this is implemented as a function or functor depends on the target language characteristics.

`dest` is mandatory. `annotations` is optional.

---

## Mutation Syntax `Type::`

An entity block may use the mutation syntax to indicate that the entity extends (implements, inherits from) existing types or usage-defined types.

Syntax:

```yaml
"Type1::Type2::usage.Alias::ClassName()":
  dest: file.py
  properties:
  methods:
```

Each `Type::` segment before the class name declares a mutation:
- **`ImportedType::`** — mutates a type from `Imports`. The type must be declared in the `Imports` section.
- **`usage.Name::`** — mutates a type defined in a usage spec. The usage spec already explains what `Name` is, so the contract can extend it.

The last segment before `(` is always the class name (with optional constructor signature).

Example:

```yaml
Imports:
  - Type: Object
    From: "kotlin"

Usages:
  - structures: .specs/structures.md

---

"Object::structures.Data::SomeClass()":
  dest: some.py
  annotations: |
    Class with data model representation.
  properties:
    name: |
      `str` -> string with the name.
```

Here:
- `Object::` — mutates the `Object` type imported from `Imports`,
- `structures.Data::` — mutates the `Data` type described in the `structures` usage spec,
- `SomeClass()` — the class name with optional constructor signature.

Semantics:
- The syntax `Type::` before the class name means "extends an existing interface" — how the mutation is implemented is up to the implementation agent.
- Multiple `Type::` segments indicate multiple mutations (e.g., multiple inheritance, interface implementation).
- The class name after the last `::` may include a constructor signature: `Type::ClassName(arg: Type)`.
- Mutation blocks do not require a corresponding `Imports` entry for usage-defined types — the usage spec already defines them.

---

## Entity Block Semantics

Each entity block may contain:

- `dest` (mandatory)
- `annotations` (optional)
- `properties` (optional)
- `methods` (optional)

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

`dest` is relative to the `CODEMANIFEST` file that defines the entity.

### `annotations`
`annotations` is optional. Provides structured metadata about the entity that persists through YAML parsing.

Example:

```yaml
"Loader(path: str)":
  dest: loader.py
  annotations: |
    Description of the entity.
    May include notes about source, design decisions, or caveats.
```

Semantics:
- Contains free-form text metadata about the entity.
- Does **not** define contract obligations — no implementation requirement is derived from annotations.
- Persists through YAML parsing unlike `#` comments.
- May describe: source of the entity, extraction notes, design rationale, known limitations.
- Planning agents should use annotations as context hints but must not treat annotation text as contract requirements.

### `properties`
Properties describe facade-visible data access of the entity.

Properties use a simplified mapping format — the key is the property name, the value is a description containing the type information inline.

Example:

```yaml
properties:
  name: |
    `str` -> string with the name.
  items: |
    `list[Item]` -> list of items in the collection.
```

Properties section may be omitted when the entity has no facade-visible properties.

#### Property format

```text
PropertyName: |
  `Type` -> description text
```

Semantics:
- `PropertyName` = actual property name in code (left side of the mapping key)
- The value is a multiline text containing:
  - the type in backticks,
  - a `->` separator,
  - a human-readable description

Important:
- the key is the real code-facing property name,
- the type and description are in the value, not structured into the key,
- this format is language-agnostic and easy to read.

#### What counts as a property

The properties section covers any facade-visible data accessor, regardless of implementation mechanism:
- `@property` decorated methods,
- public instance attributes (set in `__init__`, no `_` prefix).

From the facade perspective these are indistinguishable — a consumer cannot tell whether `obj.name` is a `@property` or a plain attribute. The contract treats them the same.

#### Property description
The multiline text under the property is mandatory.
It describes:
- the type of the property (in backticks),
- what the property represents,
- constraints,
- behavior expectations where relevant.

### `methods`
Methods describe facade-visible callable API of a class.

Methods use a mapping format — the key is the full method signature, the value is the description.

Example:

```yaml
methods:
  "load() -> cls_instances:list[Any]": |
    Returns a list of loaded class instances with the specified prefix
    created using the pattern `Class(method_name)`, where `method_name`
    is each method in the class matching `method_prefix`.
  "get_address(order: Order) -> delivery:Address": |
    Returns the delivery address for the given order.
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

## Hierarchical Contracts

A package may have `CODEMANIFEST` files at multiple levels.

Example:

```
some_package/
├── CODEMANIFEST              ← top-level facade: re-exports, top-level entities
├── http/
│   ├── CODEMANIFEST          ← HTTP entities: HTTPClient, HTTPSession, etc.
│   └── ...
└── utils/
    ├── CODEMANIFEST          ← utility functions: join, add_subdomain_to_url
    └── ...
```

Rules:
- Each `CODEMANIFEST` describes entities and functions at its level only.
- `dest` paths are relative to the `CODEMANIFEST` file location.
- A parent `CODEMANIFEST` may import and re-export entities from child `CODEMANIFEST` files using `Module` imports and `->` re-exports.
- A child `CODEMANIFEST` does not need to reference its parent.

---

## Facade Scope

The package facade is the set of names available via `import package` or `from package import ...`.

Facade exposure may happen at different levels:
- **primary facade** — available from the top-level `__init__.py`,
- **secondary facade** — available from a subpackage `__init__.py` (e.g., `package.submodule`).

Every entity defined in `CODEMANIFEST` (including re-exports) must be available from at least one facade level.

The contract does not currently distinguish primary from secondary facade. Both are valid facade exposure. The planning agent must verify that each entity is importable from the declared facade location.

---

## Meaning of the DSL

The DSL defines:
- required facade entities,
- required API shape,
- required implementation locations,
- behavioral meaning from descriptions,
- external contract dependencies,
- re-exported types and modules,
- standalone functions (as top-level blocks without properties/methods),
- interface mutations via `Type::` syntax.

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
7. Re-exported types and modules must be available from the facade but carry no implementation obligation.
8. Re-exports can only embed entities from lower filesystem levels.
9. Package boundaries are user-defined and must be preserved.
10. Subpackage contracts are independent — each `CODEMANIFEST` owns its level.
11. Entity names may contain constructor signatures to document instantiation expectations.
12. The `Type::` mutation syntax declares interface mutation without prescribing implementation.

---

## Planning Consequences

A planning agent should use this DSL to answer:
- What must be present on the facade?
- What must be implemented in which file?
- What is already implemented?
- What is missing?
- What should be added internally to realize the contract?
- What must be re-exported without implementation?
- What standalone functions must be implemented?
- What interface mutations are required?
- What must be tested to prove contract compliance?

---

## Notes for Multi-Entity Contracts

A single `CODEMANIFEST` may describe multiple entities, functions, and re-exports.

Entities may:
- share a single `dest`,
- point to different `dest` files,
- depend on imported contract types,
- mutate other types via `Type::` syntax,
- belong to the same facade surface.

This is valid.

There is no global rule that one contract entity must correspond to one implementation file.

The user defines the contract.
The planning and coding agents must preserve it.
