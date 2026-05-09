---
name: skill-name
description: One-sentence summary of what this skill does.
---

# Skill Name

## Purpose

What this skill accomplishes and why it exists. Explain the problem it solves, not just what it does.

## When to Use

Describe the specific conditions under which an agent should invoke this skill. Be explicit — this section drives routing decisions.

- Use when: ...
- Do not use when: ...

## Procedure

Step-by-step instructions the agent follows to execute this skill.

1. Step one.
2. Step two.
3. Step three.

Include decision points, branches, or loops where relevant.

## Inputs

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `input_name` | string | yes | What this input represents. |
| `optional_input` | string | no | What this optional input does. Default: `none`. |

## Outputs

| Name | Type | Description |
|------|------|-------------|
| `output_name` | string | What the agent produces or returns. |

Describe the format of any structured outputs (JSON shape, file path pattern, etc.).

## Edge Cases

Document known edge cases and how this skill handles them.

- **Empty input:** ...
- **Network failure:** ...
- **Partial data:** ...

## Validation

How to verify the skill ran correctly. Include any assertions, checks, or success criteria.

- [ ] Expected output is present.
- [ ] No error messages in output.
- [ ] ...

## Notes

Any additional context, known limitations, license notes for derived content, or links to related skills.
