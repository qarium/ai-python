# Code-to-Contract Extraction Agent

## Purpose

Analyze an existing Python package and produce `CODEMANIFEST` contract files following the DSL defined in `../employees-lead-plan/dsl-spec.md`.

You do **not** write implementation code.
You do **not** create any files other than `CODEMANIFEST`.
You **extract** the public contract surface from working source code.

---

## Input

The skill receives arguments with the following meaning:

- **First argument**: path to the **source package** to analyze (e.g., a path to an existing project like `../other-project/other_pkg/`).
- **Second argument** (optional): path to the **target package** where the contract will be placed. If not provided, the target is the current project's main package.

If no arguments are provided, ask the user:
1. Which package to analyze?
2. Where to place the contract?

The source path must point to a Python package directory (containing `__init__.py`) or to a project root with a single importable package.

---

## Output

One or more `CODEMANIFEST` files:

- **Package root** `CODEMANIFEST` — placed inside the target package directory (e.g., `some_package/CODEMANIFEST`).
- **Subpackage** `CODEMANIFEST` files — placed inside subpackage directories that have their own facade surface (e.g., `some_package/utils/CODEMANIFEST`).

No other files are created. The `plan` skill is responsible for creating the directory structure and implementation files.

All output must conform to the DSL specification defined in `../employees-lead-plan/dsl-spec.md`.

---

## Extraction Phases

### Phase 1: Discovery

1. Locate the source package directory.
2. Read `pyproject.toml` or `setup.py` to understand the project name and dependencies.
3. List all `.py` files inside the package, excluding:
   - `__pycache__`
   - test directories
   - private modules starting with `_` (unless they contain entities re-exported in `__init__.py`)
4. Build a complete file tree.
5. Identify subpackages (directories with `__init__.py`).

### Phase 2: Facade Detection

1. Read the top-level `__init__.py` of the source package.
2. Extract `__all__` if present — this defines the **primary facade**.
3. If `__all__` is absent, analyze imports to determine what is re-exported.
4. Read subpackage `__init__.py` files to detect secondary facade entities.
5. Build the **facade set**: entity names that are publicly available from the package.
6. For each subpackage, build a **subpackage facade set**.

Entities NOT in the facade set are considered **internal** and must not appear in `CODEMANIFEST`.

#### Module expansion
The DSL does not have a `Module` concept. If the facade set contains a **module reference** (e.g., `from . import http` or `from .http import http` in `__init__.py`), the module itself is NOT a contract entity. Instead:

1. Read the module's `__init__.py` and source files to discover its public entities.
2. Trace all usages of that module across the package (e.g., `http.Client(...)`, `http.Session(...)`).
3. Determine which specific objects from the module are actually used or re-exported.
4. Add those objects to the facade set **individually** — each as its own entity or re-export.
5. Replace the module-level import with specific object imports.

Example — before expansion:
```python
# some_package/__init__.py
from . import http  # module re-export
```

After expansion, the `CODEMANIFEST` should contain individual re-exports or entities from `http/`:
```yaml
"->HTTPClient": {}
"->HTTPSession": {}
```

This applies at every level of the hierarchy — root `__init__.py`, subpackage `__init__.py`, etc.

### Phase 3: Dependency Detection

Analyze all facade entities to detect external dependencies and external knowledge.

#### Import detection
For each type used in signatures that is not locally defined:
- Determine whether it comes from another `CODEMANIFEST` contract (e.g., a type defined in a sibling package within the same project).
- Record each as an `Imports` entry with `Type` kind.
- **External library types** (e.g., `requests.Response`, `pydantic.BaseModel`) must **not** be recorded in `Imports`. They are recorded in `Usages` instead.

#### External resource detection
`Usages` is a general-purpose section — it can hold anything the implementation agent needs to know from outside the package. Detect and record all external resources the code depends on:

For each external Python package used in the source code:
- Identify the package name.
- Extract the purpose from usage patterns in the code.
- Record as a `Usages` entry with a spec path or annotation describing the library's role, the types it provides, or the pattern/convention it follows.
- If specific types from the external library are used in signatures (e.g., `HTTPError`, `Response`) or as base classes in `Type::` mutations (e.g., `BaseModel`, `Session`), include them in the usage annotation so the implementation agent knows what to import. When generating `Type::` mutations for external types, use the qualified syntax: `usage.Type::` (e.g., `pydantic.BaseModel::`), where `usage` is the key in the `Usages` section.

For any other external resource the code interacts with:
- FFI/bridge calls to another language (Rust via PyO3, C via ctypes, Go via cgo, etc.)
- External tools or build steps required by the package
- Protocols, API schemas, or configuration formats the package implements
- Spec files or documentation the implementation must follow
- Record each as a `Usages` entry with a path to the relevant spec or an annotation explaining the resource.

#### Re-export detection
For each entity in the facade that is **not locally defined** but imported from an external package:
- This is a re-export — the entity passes through the facade as-is.
- Record as a `"->Name": {}` re-export block (for internal types) or `"->usage.Type": {}` (for external types from `Usages`).
- If the source is an internal `CODEMANIFEST` type, also record the corresponding `Imports` entry.
- If the source is an external library, ensure the library is described in `Usages` so the implementation agent knows where to import from.

Re-exports from `Imports` can only embed entities from files at lower levels in the filesystem hierarchy relative to the current `CODEMANIFEST`. Never create an `Imports` re-export that references a sibling or parent level. Re-exports from `Usages` (external types) use qualified names (e.g., `"->requests.HTTPError": {}`) and are not subject to hierarchy constraints. Re-exporting from `Usages` is allowed but not recommended — prefer re-exporting internal types from `Imports`.

### Phase 4: Entity Extraction

For each file containing facade entities:

1. Read the source file.
2. Identify top-level classes that are part of the facade set.
3. For each class:
   - Detect **parent classes** (bases in the class definition):
     - For each base class → generate `Type::` mutation syntax (e.g., `pydantic.BaseModel::ClassName()`). A mutation means the entity extends an existing type — the mechanism is not prescribed. See `dsl-spec.md` → *Mutation Syntax `Type::`* for the conceptual explanation.
     - Types from `Imports` use simple names (e.g., `Object::`). Types from `Usages` use qualified names: `usage.Type::` (e.g., `pydantic.BaseModel::`).
     - Every base class referenced in `Type::` **must** have a corresponding entry in `Imports` (for internal types) or `Usages` (for external types) that explicitly declares that type with its import path.
     - Multiple bases → multiple `Type::` segments before the class name.
   - Extract all **public methods** (not starting with `_`, unless `__call__`).
   - Extract all **public properties and attributes**:
     - `@property` decorated methods.
     - Public instance attributes set in `__init__` (no `_` prefix). These are facade-equivalent to properties — a consumer cannot distinguish them from `@property` through the public API.
   - For each method:
     - Record the full signature: name, parameters with types, return type.
     - Extract the docstring as the method description.
   - For each property or public attribute:
     - Record name and return type.
     - Extract the docstring as the description (for `@property`), or infer from name and type (for plain attributes).
   - `__init__` parameters are NOT included in the contract — they are implementation details. Exception: public instance attributes set from `__init__` parameters ARE extracted as properties if they are publicly accessible (no `_` prefix).
4. For each standalone function that is part of the facade set:
   - Record the full signature.
   - Extract the docstring as the function description.

### Phase 5: Description Handling

Descriptions are **mandatory** in the DSL.

### Language

All descriptions, annotations, and text content in `CODEMANIFEST` files must be written in English.
When the source code contains non-English docstrings, translate the description to English
and mark it as inferred.

Priority order for descriptions:
1. **Google-style docstrings** — extract the first paragraph (summary line + extended description).
2. **Any docstring present** — use the first meaningful paragraph.
3. **No docstring** — infer a brief description from the entity name, method name, and return type. Mark it as inferred.

For inferred descriptions, include an `annotations` entry to mark them. Use entity-level `annotations` for entities:

```yaml
"EntityName()":
  location: path/to/file.py
  annotations: |
      Descriptions inferred from code — review for accuracy.
  properties:
  methods:
```

Standalone function blocks use `annotations` for inferred descriptions:

```yaml
"func(arg: Type) -> result:Type":
  location: path.py
  annotations: |
    What the function does.
```

### Phase 6: Location Mapping

For each extracted entity:
- `location` is the file path **relative to the `CODEMANIFEST` file location**.
- For the package root `CODEMANIFEST`: strip the leading package name directory.
- For subpackage `CODEMANIFEST` files: `location` is relative to the subpackage directory.

Examples:
- Package root `CODEMANIFEST`: package `some_package/`, file `some_package/http/client.py` → `location: http/client.py`
- Subpackage `CODEMANIFEST`: subpackage `some_package/utils/`, file `some_package/utils/url.py` → `location: url.py`

Multiple entities in the same file share the same `location`. This is valid per the DSL spec.

### Phase 7: Signature Formatting

Transform Python signatures into DSL format.

#### Entity name format
- If the class has a constructor with parameters → use full constructor signature: `"ClassName(arg: Type)"`
- If the class has no constructor parameters → use empty constructor signature: `ClassName()`
- If the class extends other types → prepend `Type::` segments: `"ParentType::ClassName(arg: Type)":` for internal types, `"usage.ParentType::ClassName(arg: Type)":` for external types

#### Property format
```
PropertyName -> Type: |
  Description text
```

Rules:
- `PropertyName` = actual property or attribute name from code.
- `Type` = the Python type annotation (after `->` in the key).
- Description text in the value explains what the property represents.

#### Method format
```
method(arg: Type, arg2: Type2) -> label:ReturnType
```

Rules:
- Method name as-is from code.
- Parameters with their type annotations.
- `self` and `cls` are excluded.
- `*args` and `**kwargs` are included as `*args: t.Any` and `**kwargs: t.Any`.
- Return type from annotation. If missing, use `Any`.
- `label` = a semantic hint for the return value.

#### Function format
Same as method format but as a top-level block without `properties` or `methods`:
```yaml
"function_name(arg: Type) -> label:ReturnType":
  location: path/to/file.py
  annotations: |
    What the function does.
```

### Phase 8: Hierarchical Decomposition

Determine which subpackages need their own `CODEMANIFEST` files.

Rules:
- A subpackage needs its own `CODEMANIFEST` if it contains facade entities or standalone functions.
- The subpackage `CODEMANIFEST` describes entities and functions at its level only.
- The parent `CODEMANIFEST` uses `->` re-exports to make subpackage entities available on the top-level facade.

Example decomposition:
```
some_package/
├── CODEMANIFEST              ← entities + re-exports
├── http/
│   ├── CODEMANIFEST          ← HTTP entities (if applicable)
│   └── ...
└── utils/
    ├── CODEMANIFEST          ← function blocks for join, add_subdomain_to_url
    └── ...
```

Parent `CODEMANIFEST`:
```yaml
"->join": {}
```

Subpackage `CODEMANIFEST` (`some_package/utils/CODEMANIFEST`):
```yaml
"join(*parts: str) -> joined:str":
  location: url.py
  annotations: |
    Joins URL parts into a single URL string.
```

### Phase 9: Assembly

Assemble each `CODEMANIFEST` file with the following structure (in order):

```yaml
Imports:
  - Type: SomeType
    From: "other_package"

Usages:
  - requests: .specs/requests.md
  - pydantic: |
       External library for data validation.
       Import `BaseModel` for creating data models.
  - ex_type: |
       External library providing `ExternalType`.
  - pattern: |
       Annotation describing a usage pattern.

Annotations: |
    Optional file-level description.

---

"->ex_type.ExternalType": {}

"EntityName()":
  location: path/to/file.py
  annotations: |
      Optional entity-level description.
  properties:
    name -> str: |
      Description text.
  methods:
    "method(arg: Type) -> label:ReturnType": |
      Description text.

"function_name(arg: Type) -> label:ReturnType":
  location: path/to/file.py
  annotations: |
    What the function does.
```

Ordering rules:
- Sections ordered: `Imports`, `Usages`, `Annotations`, `---`, re-exports, entity blocks, standalone function blocks.
- Re-exports ordered alphabetically.
- Entities ordered by `location` path, then by name.
- Properties before methods within each entity.
- Methods ordered by: `__call__` first (if present), then alphabetical.
- Standalone function blocks ordered by `location` path, then by name.

---

## What NOT to Extract

Do **not** include in `CODEMANIFEST`:
- Private methods (starting with `_`, except `__call__`)
- Private properties or private instance attributes (starting with `_`)
- `__init__` parameters as such — only the resulting public instance attributes
- Class-level configuration attributes (e.g., `__session_class__`, `__response_class__`) — these are internal customization points, not facade contract
- Internal helper functions not exposed in `__init__.py`
- Re-exported external types as contract entities — they are re-exports (`->Name: {}`)

---

## Reasoning Discipline

Separate clearly:
- **Extracted** — directly observed in the source code
- **Inferred** — deduced from context, naming, or patterns
- **Ambiguous** — unclear classification

Mark all inferred descriptions explicitly.

---

## Output Presentation

After assembling all `CODEMANIFEST` files and before writing:

1. Show each proposed `CODEMANIFEST` content with its file path.
2. Show a summary table of all extracted entities across all files:

| File | Entity | Type | location | Properties | Methods/Functions | Description source |
|------|--------|------|------|-----------|-------------------|--------------------|
| CODEMANIFEST | ... | class | ... | N | M | docstring / inferred |
| utils/CODEMANIFEST | join | function | url.py | - | - | docstring / inferred |

3. Show a summary of detected dependencies:

| Category | Name | Source |
|----------|------|--------|
| Import (Type) | User | identity |
| Usage (external type) | HTTPError | requests |
| Usage (library) | requests | pyproject.toml |
| Usage (pattern) | pydantic | .specs/pydantic.md |
| Re-export | HTTPError | requests (from Usages) |
| Re-export | join | some_package.utils (discovered from child CODEMANIFEST) |

4. List any **ambiguities** or **decisions** made during extraction:
   - Entities excluded from the facade (with reason).
   - Types that could not be fully resolved.
   - Methods with missing return type annotations.
   - Descriptions that were inferred rather than extracted.
   - Public instance attributes treated as properties.
   - Subpackages that do not need their own `CODEMANIFEST`.

5. Ask the user to review and confirm before writing `CODEMANIFEST` files.

---

## Validation

Before finalizing each `CODEMANIFEST`, verify:

1. Every entity listed is available from the package facade (importable via `__init__.py`).
2. Every `location` points to an actual source file.
3. Every property has a name and a description with type information.
4. Every method has a name, parameters with types, a return label, and a return type.
5. Every property and method has a description (extracted or inferred).
6. Every standalone function block has a `location` and description in `annotations`.
7. No private entity is included unless explicitly requested.
8. Every re-export has a corresponding `Imports` entry (for internal types) or `Usages` entry (for external types).
9. Every `Usages` entry has either a spec path or annotation text.
10. `location` paths are relative to the `CODEMANIFEST` file location (not absolute).
11. The YAML is syntactically valid.

---

## Quality Bar

A good extraction:
- captures every public facade entity,
- preserves exact type annotations from source,
- provides meaningful descriptions,
- correctly maps `location` relative paths,
- clearly distinguishes extracted from inferred information,
- correctly identifies re-exports vs contract entities,
- generates appropriate `Imports`, `Usages`, and re-export sections,
- produces hierarchical `CODEMANIFEST` files for subpackages with their own facade,
- only creates `CODEMANIFEST` files — no other files.

A bad extraction:
- contains non-English text in descriptions, annotations, or labels,
- includes internal-only helpers as contract entities,
- drops type annotations or invents missing ones without marking as inferred,
- uses absolute paths in `location`,
- includes `__init__` parameters as properties,
- omits descriptions,
- fails to detect the facade boundary,
- treats re-exported types as contract entities instead of re-exports,
- creates files other than `CODEMANIFEST`,
- misses subpackage contracts,
- mixes parent and child entities in a single `CODEMANIFEST`.

---

## Final Self-Check

Before returning the result, verify:

1. Did I identify the correct package boundary?
2. Did I detect the complete facade set?
3. Did I extract every facade entity?
4. Did I include public instance attributes (not just `@property`)?
5. Did I preserve exact type annotations?
6. Did I provide descriptions for every property, method, and function?
7. Are all `location` paths relative to the correct `CODEMANIFEST` file location?
8. Did I exclude private/internal entities?
9. Did I handle re-exports correctly (`->Name: {}` with corresponding `Imports` for internal types or `Usages` for external types)?
10. Did I clearly mark inferred descriptions?
11. Is the YAML syntactically valid?
12. Did I create **only** `CODEMANIFEST` files — no other files?
13. Did I generate `Imports` only for internal types (from other `CODEMANIFEST` files)?
14. Did I generate `Usages` for all external resources — libraries, types, patterns, conventions, external code references, build tools, protocols, specs, and any other external knowledge?
15. Did I produce separate `CODEMANIFEST` files for subpackages that have their own facade?
16. Did I include `annotations` where appropriate (file-level, entity-level, function-level)?
17. Is all text content in `CODEMANIFEST` files written in English?
18. Did I detect `Type::` mutations for parent classes correctly?
19. Did I use constructor signature syntax for entity names (e.g., `ClassName()` or `ClassName(arg: Type)`)?

---

## Retrospective

After completing all main work, perform the retrospective as defined in CLAUDE.md → Skill Retrospective.

Related skills for improvement: `employees-lead-plan`, `dsl-spec.md`.
