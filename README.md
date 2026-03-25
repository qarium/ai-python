# AI Python

Agent commands and skills for python project development

## Installation

**Using with claude code:**

```bash
git clone git@github.com:qarium/ai-python.git .claude
```

## Add custom commands and skills

Use `commands/custom` and `skills/custom` for you owen commands creation

## Employees

| Role            | Command                         | Description                                                                                                                                                                                          |
|-----------------|---------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Lead**        | `/qarium:employees:lead`        | Senior technical lead. Reviews code, helps with code problems, collects and accumulates project knowledge (architecture, patterns, decisions) so AI agents don't repeat mistakes in future sessions. |
| **QA**          | `/qarium:employees:qa`          | Senior QA automation engineer. Writes reliable tests, fixes broken tests after code changes, maintains test configuration and coverage, and accumulates project testing knowledge.                   |
| **DevOps**      | `/qarium:employees:devops`      | Senior DevOps engineer. Configures, maintains, and evolves CI/CD pipelines (GitHub Actions) for lint, tests, docs, and publish — keeps them in sync with project configuration.                      |
| **Tech Writer** | `/qarium:employees:tech-writer` | Senior technical writer. Creates and updates developer documentation (MkDocs), keeps it in sync with source code changes, and runs audits for documentation accuracy.                                |

Each employee has two skills: **onboarding** (initial setup from scratch) and **feature** (ongoing work after changes). The command dispatcher automatically selects the appropriate skill.

## Evolution

During project work, we may encounter skills that don't work correctly or don't behave exactly as we'd like, and we want to change their behavior.
We can always modify prompts during work and push changes to the repository directly from the project — to do this, keep the `.claude` directory inside the project rather than in `~/`.
Improve workflows and employee skills directly during project work and publish changes.
