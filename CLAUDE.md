# Project rules

## LLM instructions

* Do not commit files in `.claude` and `docs/plans`

## Employees

The project has skills in `.claude/skills` and commands `.claude/commands`.
These skills are divided into roles: lead/qa/tech-writer/devops. Each role has a team responsible for dispatching and skills.

### Education

Skills should be continuously improved, and if a problem arises during skill `employees-*` execution, the user should be asked whether they would like the role to learn it.

**If the user answers yes, then:**
- Clarify all requirements for changing the skill and command
- Build a plan for how the changes will be integrated into the current logic
- Ask the user for permission to make changes
- Implement the changes
- Check the modified prompts for consistency in logic after making changes, paying particular attention to the sequence of phases and steps, as well as the digraph
- Implement the changes after first confirming the user's permission
- Notify the user that the changes have been successfully made
- Continue with previously assigned tasks
