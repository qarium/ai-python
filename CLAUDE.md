# Project rules

## LLM instructions

* Do not commit files in `.claude` and `docs/plans`

## Employees

The project has skills in `.claude/skills` and commands `.claude/commands`.
These skills are divided into roles: lead/qa/tech-writer/devops/developer. Developer has two skill types (feature, review); other roles have three:
- **onboarding** — initial setup from template, processes `${ROLE_*}` placeholders
- **feature** — ongoing work (write tests, update docs, modify CI, accumulate knowledge)
- **audit** — check project against template and role conventions

Dispatchers ask the user to choose between `feature` and `audit` when infrastructure exists.

### Skill Retrospective

After completing the main work of any `employees-*` skill, the skill MUST perform a retrospective.

#### Execution

1. Call AskUserQuestion:
   - "Were there any issues with the approach or result?"
   - Options: "No, all good" / "Yes, here's what..."
   - If "No" → skip remaining retrospective steps and continue with the next phase of the current workflow or proceed to the next skill

2. Listen to problem description

3. Classify the problem:
   - **Project-specific** — local to the project, does not require skill improvement → record a lesson in `.qarium/ai/employees/<role>.md` → go to step 3a
   - **Skill problem** — the skill itself has a gap or incorrect instruction → propose a solution, get user approval, update the skill → go to step 3b

3a. **Record project lesson:**
   - Add a row to the `## Lessons` table in `.qarium/ai/employees/<role>.md`
   - Columns: Problem | Why | How to prevent
   - If `## Lessons` section does not exist — create it
   - Continue to the next phase of the current workflow

3b. **Improve the skill:**
   - Read current SKILL.md and determine related skills:
     - Common sections (venv detection, monorepo handling, etc.)
     - Same role (onboarding ↔ feature)
     - Shared infrastructure (CI workflow → devops + qa)
   - Show user the list: must update: {current skill}, also affected: {related skills}
   - Ask which to update
   - Propose concrete changes for each selected skill (which lines/sections, show diff)
   - Get user confirmation for changes
   - Apply changes to selected SKILL.md files
   - For each modified SKILL.md: check consistency (phase sequence, mermaid diagram, contradictions). If problems found → propose fixes → back to confirmation
   - Publish updated skills to the `.claude` git repository: `cd .claude`, stage all changes, commit with a descriptive message summarizing what was improved and why, push to the remote branch, then `cd ..` to return to the project root. **The working directory must always be the project root after this step.**
   - Confirm to user that skills are updated and published
