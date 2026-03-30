---
name: employees-tech-writer:onboarding
description: Used when a Python project has no documentation infrastructure — no docs/ directory with content, no .qarium/ai/employees/tech-writer.md file. Processes TECH_WRITER_* placeholders from template, sets up MkDocs, creates documentation structure, and records configuration in .qarium/ai/employees/tech-writer.md.
---

# Tech Writer Onboarding

## Overview

Setting up project documentation infrastructure.
Processes `${TECH_WRITER_*}` placeholders in template files, creates MkDocs documentation structure, and records configuration in `.qarium/ai/employees/tech-writer.md` for future sessions.

MkDocs is the only static site generator. No choice is offered.

## When to use

- The project has no `docs/` directory or it contains no `.md`/`.rst` files
- There is no `.qarium/ai/employees/tech-writer.md` file
- The `/qarium:employees:tech-writer` dispatch routes here automatically

**DO NOT use when:**
- The project already has documentation with content and `.qarium/ai/employees/tech-writer.md` exists — use `qarium:employees:tech-writer:feature`
- This is not a Python project
- `.qarium/ai/employees/tech-writer.md` already exists — warn the user and suggest using `qarium:employees:tech-writer:feature`

## Template

This skill processes `${TECH_WRITER_*}` placeholders that were left by the lead onboarding in template files.

### Placeholder processing

Read project files and find all `${TECH_WRITER_*}` placeholders:

| Variable | Expected in | How to compute |
|----------|-------------|----------------|
| TECH_WRITER_SITE_NAME | `mkdocs.yml` | Read `[project.name]` from pyproject.toml |
| TECH_WRITER_SITE_DESCRIPTION | `mkdocs.yml` | Read `[project.description]` from pyproject.toml |
| TECH_WRITER_SITE_URL | `mkdocs.yml` | Read `[project.urls] Homepage` from pyproject.toml, or leave empty |
| TECH_WRITER_REPO_URL | `mkdocs.yml` | From `git remote` or pyproject.toml |
| TECH_WRITER_QUICK_START | `README.md` | Based on source analysis — CLI command, basic usage example |
| TECH_WRITER_DOCS_LINK | `README.md` | Link to site_url or relative `./docs/` |

Also process `${DEVOPS_TRIGGER_BRANCH}` in `mkdocs.yml` if it's still a placeholder — read `default_branch` from `.qarium/ai/employees/lead.md` Config or determine via git.

Replace the entire `${VARIABLE:="prompt"}` with the computed value. Do NOT modify `${QA_*}`, `${DEVOPS_*}` placeholders (unless explicitly noted above).

## Virtual Environment

Before executing any shell commands (pip, python, mkdocs), detect the project's virtual environment:

1. Check for `.venv/` in the project root
2. If not found, check for `venv/`
3. If found → prefix all commands: `source .venv/bin/activate && <command>` (or `source venv/bin/activate && <command>`)
4. If not found → execute `<command>` as-is

This applies to Phase 5 (pip install, mkdocs build).

```dot
digraph flow {
    rankdir=LR;
    analyze [label="Phase 1: Project analysis\ntype, dependencies, Python version, source structure" shape=box];
    structure [label="Phase 2: Structure proposal\nsource analysis → pages" shape=box];
    config [label="Phase 3: Configuration\nbuild_cmd, deploy_cmd, examples_file, base_branch" shape=box];
    process [label="Phase 4: Process placeholders\nTECH_WRITER_* + create doc pages" shape=box];
    install [label="Phase 5: Installation and verification\npip install, build_cmd" shape=box];
    rules [label="Phase 6: Write .qarium/ai/employees/tech-writer.md\nConfig + Rules" shape=box];
    retro [label="Phase 7: Retrospective\nCLAUDE.md → Skill Retrospective" shape=box];
    done [label="Done" shape=box];

    analyze -> structure;
    structure -> config;
    config -> process;
    process -> install;
    install -> rules;
    rules -> retro;
    retro -> done;
}
```

## Phase 1: Project Analysis

Collecting information about the current state of the project.

1. **Project type** — read `pyproject.toml` and classify:
   - **Library** — contains `[project]` without `scripts` or `[project.scripts]` is empty
   - **CLI application** — contains `[project.scripts]` or uses click/typer
   - **Web application** — depends on fastapi, django, flask
2. **Python version** — read `requires-python` from `[project]` in `pyproject.toml`. Extract the minimum version (e.g., `>=3.10` → `py310`). If not specified, default to `py312`.
3. **Existing documentation** — check `docs/`, `mkdocs.yml`
4. **Existing README** — check `README.md`: does not exist, standard git template, or full content
5. **Existing dependencies** — check `[project.dependencies]` and `[project.optional-dependencies]` in `pyproject.toml` for documentation tools (mkdocs, mkdocs-material, etc.)
6. **Dependency group name** — check `[project.optional-dependencies]` for existing groups with documentation tools. Remember the group name (e.g., `docs`). If nothing is found, default to `docs`.
7. **Source structure** — scan the source directory to understand modules, CLI commands, configuration, API. This information is used in Phase 2 for proposing documentation structure.
8. **.gitignore** — check if `docs/plans/` is in `.gitignore`

Present a summary to the user before proceeding to Phase 2.

### Existing documentation without configuration

If `docs/` contains `.md` or `.rst` files but `.qarium/ai/employees/tech-writer.md` is missing, inform the user that documentation already exists. Ask whether to:

1. **Configure for existing documentation** — create a configuration pointing to the existing `docs/` structure, skip creating docs/ files in Phase 4, offer to supplement navigation with missing pages. README.md should be checked and created/overwritten according to the README.md section rules in Phase 4.
2. **Start fresh** — go through all phases and create a new documentation structure (existing docs/ files will not be overwritten per Phase 4 rules; README.md is an exception)

## Phase 2: Documentation Structure Proposal

Based on the project analysis from Phase 1, propose a list of documentation pages.

### Base pages by project type

| Project type    | Base pages                                           |
|-----------------|------------------------------------------------------|
| CLI application | `index.md`, `getting-started.md`, `cli-reference.md` |
| Library         | `index.md`, `getting-started.md`, `api-reference.md` |
| Web application | `index.md`, `getting-started.md`, `api-reference.md` |
| Any             | `configuration.md`, `examples.md`                    |

### Additional pages from source analysis

Based on the source structure from Phase 1, propose additional pages:

| Found in sources                       | Proposed page      |
|----------------------------------------|--------------------|
| Multiple CLI commands with subcommands | `cli-reference.md` |
| Configuration modules                  | `configuration.md` |
| Modules with metrics/calculations      | `metrics.md`       |
| API endpoints                          | `api-reference.md` |
| Plugins/extensions                     | `plugins.md`       |

The user confirms, adds, or removes pages. The approved list forms the navigation in `mkdocs.yml`.

Present the full navigation structure and request confirmation before Phase 3.

## Phase 3: Configuration

Ask the user to confirm or adjust documentation settings.

### Questions (one at a time)

| # | Setting         | Default                    | Notes                               |
|---|-----------------|----------------------------|-------------------------------------|
| 1 | `build_cmd`     | `mkdocs build`             | Build validation command            |
| 2 | `deploy_cmd`    | `mkdocs gh-deploy --force` | Documentation deploy command        |
| 3 | `examples_file` | none (optional)            | File for usage examples             |
| 4 | `base_branch`   | auto (from lead.md or git) | Base branch for git diff comparison |

> **`base_branch` determination algorithm:**
> 1. Read `default_branch` from `.qarium/ai/employees/lead.md` Config
> 2. If `lead.md` does not exist → try `git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null`
> 3. If not resolved → `master` (fallback)

After all choices, present the full configuration summary and request confirmation before Phase 4.

## Phase 4: Process Placeholders and Create Structure

### Process TECH_WRITER_* placeholders

Read each file copied from template and replace `${TECH_WRITER_*}` placeholders:

1. **`mkdocs.yml`** — replace all `${TECH_WRITER_*}` and `${DEVOPS_TRIGGER_BRANCH}` placeholders
2. **`README.md`** — replace all `${TECH_WRITER_*}` and remaining `${LEAD_*}` placeholders

Verify no `${TECH_WRITER_*}` placeholders remain.

### Update mkdocs.yml navigation

Update the `nav` section in `mkdocs.yml` to match the approved page list from Phase 2.

### Create documentation pages

Create all approved pages as stub files with a heading — the page name.

### README.md

If README.md still contains `${LEAD_*}` placeholders (lead onboarding left them), process those too:
- `${LEAD_PACKAGE_NAME}` — from pyproject.toml
- `${LEAD_DESCRIPTION}` — from pyproject.toml

If README.md was already fully processed by lead — skip, do not overwrite.

### Rules

- Create `docs/` if it does not exist
- Do not overwrite existing files — skip if already present (README.md is an exception, see the README.md section above)
- Add `docs/plans/` to `.gitignore` (append to the end if `.gitignore` exists; create `.gitignore` if it does not)
- `index.md` — the first page in navigation with the project name and description from pyproject.toml

## Phase 5: Installation and Verification

1. Add documentation dependencies to pyproject.toml under the dependency group name determined in Phase 1 (e.g., `docs`). If the group already exists, add to it.
   - Minimum set: `mkdocs`, `mkdocs-material`
2. Install dependencies via pip:
   ```
   pip install -e ".[docs]"
   ```
   If virtualenv was detected (see Virtual Environment), prefix with `source .venv/bin/activate &&`. If not, run as-is.
3. Run `build_cmd` to verify successful documentation build.

**On build error:**
- Fix the issue
- Re-run `build_cmd`
- If after 2 iterations the build still fails — explain and wait for user instructions

## Phase 6: Write `.qarium/ai/employees/tech-writer.md`

Create the tech writer configuration file. The file is written in English.

### File structure

```markdown
# Tech Writer Config

## Config

| Key           | Value                      | Description                         |
|---------------|----------------------------|-------------------------------------|
| build_cmd     | `mkdocs build`             | Build validation command            |
| deploy_cmd    | `mkdocs gh-deploy --force` | Deploy command                      |
| examples_file | `docs/examples.md`         | File for usage examples (optional)  |
| logo_url      | `https://avatars.githubusercontent.com/u/262344922?s=200&v=4` | Standard qarium logo |
| base_branch   | `master`                   | Base branch for git diff comparison |

## Rules

### Mapping

| Source path | Documentation files |
|-------------|---------------------|

### Conventions

## Lessons

| Problem | Why | How to prevent |
|---------|-----|----------------|
```

- Fill in Config values from the user's choices in Phase 3
- `logo_url` is always the standard qarium logo: `https://avatars.githubusercontent.com/u/262344922?s=200&v=4`
- If `examples_file` is left as optional — use an empty value in the table
- Mapping — empty template (table header only), will be filled in subsequent flow calls
- Conventions — empty placeholder for future expansion

### Rules

1. Create the `.qarium/ai/employees/` directory if it does not exist
2. If `.qarium/ai/employees/tech-writer.md` already exists — DO NOT overwrite. Explain to the user and suggest using `qarium:employees:tech-writer:feature`
3. Present the generated file for user approval before writing
4. After writing, verify the file correctness by reading it back

## Common mistakes

| Mistake                                                         | Fix                                                                                    |
|-----------------------------------------------------------------|----------------------------------------------------------------------------------------|
| Overwriting existing mkdocs.yml                                 | Check before creating — only create missing files                                      |
| Overwriting existing `.qarium/ai/employees/tech-writer.md`      | Check first, on discovery suggest `qarium:employees:tech-writer:feature`               |
| Skipping dependency installation                                | Always run `pip install -e ".[docs]"` after Phase 4                                    |
| Skipping build verification                                     | Always verify that build_cmd works                                                     |
| Writing configuration without user approval                     | Present for review first                                                               |
| Failing to detect existing dependencies                         | Carefully review Phase 1 results before asking questions                               |
| Using default table values when Phase 1 detected existing tools | Phase 1 analysis takes priority over default values                                    |
| Not adding the dependency group `[docs]` to pyproject.toml      | Always add documentation dependencies to pyproject.toml                                |
| Forgetting to add `docs/plans/` to `.gitignore`                 | Check in Phase 1 and add in Phase 4                                                    |
| Overwriting a full README.md                                    | Check README.md content — only overwrite if it is a git template                       |
| Skipping creation of `docs/overrides/main.html`                 | Always create alongside mkdocs.yml in Phase 4                                          |
| Forgetting `custom_dir` and `primary: custom` in mkdocs.yml     | The mkdocs.yml template in Phase 4 contains these settings                             |
| Running `pip`/`mkdocs` without virtualenv activation            | Always check for `.venv/` or `venv/` and use `source <venv>/bin/activate && <command>` |
| Hardcoding `main` as base branch in Config                      | Always determine from lead.md or git; fallback to `master`                             |
| Asking user for logo URL                                        | Always use the standard qarium logo, never ask the user                               |
| Leaving `${TECH_WRITER_*}` placeholders in files                | All TECH_WRITER_* placeholders must be resolved in Phase 4                             |

## Phase 7: Retrospective

After completing all main work, perform the retrospective as defined in CLAUDE.md → Skill Retrospective.