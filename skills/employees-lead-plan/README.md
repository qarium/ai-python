# Agent Skill Bundle

This bundle contains the first production-oriented version of the contract planning system.

## Files

- `skill.md` — main planning skill prompt
- `dsl-spec.md` — DSL specification for `.agent.yml`
- `output-template.md` — strict response template
- `conventions.md` — general language-agnostic project conventions
- `example.md` — compact planning example

## Recommended layering

1. Contract (`.agent.yml`)
2. Planning skill (`skill.md`)
3. Output schema (`output-template.md`)
4. General conventions (`conventions.md`)
5. Language-specific conventions (future layer)

## Next recommended step

Add a separate language-specific conventions file, for example:
- `python-conventions.md`
- `typescript-conventions.md`
- `go-conventions.md`

That file should define syntax-level and idiom-level rules that this general conventions file intentionally leaves open.
