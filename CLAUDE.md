# Project rules

## LLM instructions

* Do not commit files in `.claude` and `docs/plans`

## Employees

The project has skills in `.claude/skills` and commands `.claude/commands`.
These skills are divided into roles: lead/qa/tech-writer/devops. Each role has a team responsible for dispatching and skills.

### Skill Retrospective

After completing the main work of any `employees-*` skill, the skill MUST perform a retrospective.

#### Execution

1. Call AskUserQuestion:
   - "Were there any issues with the approach or result?"
   - Options: "No, all good" / "Yes, here's what..."
   - If "No" → skip remaining retrospective steps and continue with the next phase of the current workflow or proceed to the next skill

2. Listen to problem description

3. Read current SKILL.md and determine related skills:
   - Common sections (venv detection, monorepo handling, etc.)
   - Same role (onboarding ↔ feature)
   - Shared infrastructure (CI workflow → devops + qa)

4. Show user the list:
   - Must update: {current skill}
   - Also affected: {related skills}
   - Ask which to update

5. Propose concrete changes for each selected skill (which lines/sections, show diff)

6. Get user confirmation for changes

7. Apply changes to selected SKILL.md files

8. For each modified SKILL.md:
   - Check consistency: phase sequence, digraph, contradictions
   - If problems found → propose fixes → back to step 6

9. Publish updated skills to the `.claude` git repository: stage all changes, commit with a descriptive message summarizing what was improved and why, push to the remote branch, then return to the project root

10. Confirm to user that skills are updated and published
