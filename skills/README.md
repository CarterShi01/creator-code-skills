# Skills

This directory contains individual reusable agent skills.

## Layout

Each skill lives in its own subdirectory:

```
skills/
  <skill-name>/
    SKILL.md        # Required — skill definition
    *.md            # Optional supporting docs
```

## Naming

- Directory names must be **kebab-case**.
- Names should be descriptive and action-oriented (e.g., `retry-handler`, `paginate-query`, `validate-schema`).

## Adding a Skill

1. Create a new directory under `skills/` with a kebab-case name.
2. Copy `../templates/SKILL_TEMPLATE.md` into the directory as `SKILL.md`.
3. Fill in every section of the template.
4. Add an entry to `../registry/approved-skills.md`.
5. Record the change in `../registry/skill-changelog.md`.

## Finding a Skill

See `../registry/approved-skills.md` for the full index of available skills with short descriptions.
