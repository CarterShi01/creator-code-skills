# creator-code-skills

A private repository for reusable coding-agent skills, templates, and engineering playbooks.

## Purpose

This repository stores Agent Skills that can be reused across CreatorMesh and future projects. Skills here are designed to:

- Reduce repeated prompting and context cost across projects
- Encode proven engineering patterns in a consistent, reusable format
- Provide a shared foundation for AI-assisted development workflows

## Structure

```
creator-code-skills/
  skills/          # Individual reusable agent skills
  templates/       # Starter templates for skills and interfaces
  registry/        # Approved skills index and changelog
```

## Principles

- **Generic over specific.** Skills should be broad enough to apply across projects. Project-specific skills belong in the project repo unless they become broadly reusable.
- **Small and focused.** Each skill should do one thing well.
- **Context-efficient.** Good skills reduce repeated prompting and keep agent context lean.
- **Documented.** Every skill must clearly define when to use it, what it expects, and what it produces.

## Adding a Skill

1. Copy `templates/SKILL_TEMPLATE.md` into `skills/<skill-name>/SKILL.md`.
2. Fill in all sections.
3. Add an entry to `registry/approved-skills.md`.
4. Record the addition in `registry/skill-changelog.md`.

## Templates

- `templates/SKILL_TEMPLATE.md` — standard skill definition format
- `templates/INTERFACE_TEMPLATE.md` — module or service interface contract format

## Methodology

- [Quality and Cost Harness](docs/quality-and-cost-harness-20260509.md) — Engineering lessons from CreatorMesh: how to run AI coding agents under strict quality and token-cost constraints.
