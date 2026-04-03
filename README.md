# AI Python

Claude Code extension with AI employee agents for Python project development.

## Installation

If you are using [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

```bash
git clone git@github.com:qarium/ai-python.git .claude
```

for another agent, please create symlink to folder

## Repository structure

```
commands/qarium/employees/   — command dispatchers (entry points)
skills/qarium/employees/     — skill definitions (onboarding + feature per employee)
```

## Custom commands and skills

Create your own commands and skills in `commands/custom` and `skills/custom-*`. These directories are gitignored and won't be overwritten when pulling updates.

## Employees

| Role            | Command                         | Artifact                               | Description                                                                                                                                                                                          |
|-----------------|---------------------------------|----------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Lead**        | `/qarium:employees:lead`        | `.qarium/ai/employees/lead.md`         | Senior technical lead. Reviews code, helps with code problems, collects and accumulates project knowledge (architecture, patterns, decisions) so AI agents don't repeat mistakes in future sessions. |
| **Developer**   | `/qarium:employees:developer`   | `.qarium/ai/employees/developer.md`    | Senior developer. Implements features based on `# AGENT:` markers left in code, and reviews code for quality and marker compliance. Works in vibe coding workflow — AI reads explicit instructions from comments and executes them. |
| **QA**          | `/qarium:employees:qa`          | `.qarium/ai/employees/qa.md`           | Senior QA automation engineer. Writes reliable tests, fixes broken tests after code changes, maintains test configuration and coverage, and accumulates project testing knowledge.                   |
| **DevOps**      | `/qarium:employees:devops`      | `.qarium/ai/employees/devops.md`       | Senior DevOps engineer. Configures, maintains, and evolves CI/CD pipelines (GitHub Actions) for lint, tests, docs, and publish — keeps them in sync with project configuration.                      |
| **Tech Writer** | `/qarium:employees:tech-writer` | `.qarium/ai/employees/tech-writer.md`  | Senior technical writer. Creates and updates developer documentation (MkDocs), keeps it in sync with source code changes, and runs audits for documentation accuracy.                                |

Each employee has three skills: **onboarding** (initial setup from scratch), **feature** (ongoing work after changes), and **audit** (check project against conventions). The command dispatcher automatically selects the appropriate skill. Developer has **onboarding**, **feature** (implement `# AGENT:` markers), and **review** (check code against markers and quality) instead of audit.

## Development Workflow

### New project

1. Create the main Python package — this will be the project root
2. Run `/qarium:employees:lead` — onboarding: analyzes the project, fills `lead.md` with architecture, structure, and code patterns
3. Run `/qarium:employees:developer` — onboarding: sets up developer configuration, code conventions, and `# AGENT:` marker standards in `developer.md`
4. Run `/qarium:employees:qa` — onboarding: configures pytest + ruff, creates test structure, writes test configuration to `qa.md`
5. Run `/qarium:employees:tech-writer` — onboarding: configures MkDocs, creates documentation structure, writes documentation configuration to `tech-writer.md`
6. Run `/qarium:employees:devops` — onboarding: configures GitHub Actions workflows (lint, tests, docs, publish), writes CI configuration to `devops.md`

After onboarding, each employee's configuration (`.qarium/ai/employees/*.md`) is ready for future sessions.

### Existing project (feature development)

1. Write the feature — implement the new code
2. Run `/qarium:employees:lead` — reviews code changes, places `# AGENT:` markers for problems found, extracts knowledge, accumulates decisions and patterns
3. Run `/qarium:employees:developer` — feature: scans `# AGENT:` markers placed by Lead, plans and implements tasks; review: checks code against markers and quality standards
4. Run `/qarium:employees:qa` — writes/updates tests for changed files, fixes broken tests, maintains test configuration
5. Run `/qarium:employees:tech-writer` — updates documentation for changed/added code, runs build validation
6. Run `/qarium:employees:devops` — analyzes changes, updates CI/CD pipelines if needed, syncs `devops.md`

**Important:** Make a git commit **after all employees have completed their work**, not between each one. Each employee can produce multiple commits (20+), and the AI agent may lose context and miss important changes if it processes too many commits at once.

## Contributing

Skills and commands can be modified directly during project work. To preserve your changes and contribute back:

1. Keep the `.claude` directory inside the project (not in `~/`) so you can push changes to the repository
2. Modify prompts as needed when workflows don't behave as expected
3. Publish changes to improve the shared skills for everyone
