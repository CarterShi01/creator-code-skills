# Agent Guidelines for creator-code-skills

Rules and conventions all agents must follow when working in this repository.

## Skill Structure

- Follow the Agent Skills structure defined in `templates/SKILL_TEMPLATE.md`.
- Each skill lives in its own directory under `skills/` and must contain a `SKILL.md` file.
- Skill directories use **kebab-case** names (e.g., `skills/api-error-handler/`).

## Writing Skills

- Every skill must clearly define **when to use it** so agents can make correct routing decisions.
- Prefer small, focused, reusable skills over large monolithic ones.
- A skill should do one thing well. If it does two things, split it.
- Define explicit inputs and outputs. Avoid implicit side effects.

## Security

- **Do not store secrets**, credentials, API keys, tokens, or environment-specific configuration in this repository.
- Do not hardcode any value that varies between environments or users.

## Licensing

- Do not copy third-party skills or content without first checking and documenting the license.
- If a skill is derived from external work, note the source and license in the skill's Notes section.

## Registry

- Every approved skill must have an entry in `registry/approved-skills.md`.
- Every addition, modification, or removal must be recorded in `registry/skill-changelog.md`.

## Commit Style

- Commit messages should be clear and describe the skill or change being added.
- Use imperative mood: "Add retry-handler skill", "Update auth-flow template".
