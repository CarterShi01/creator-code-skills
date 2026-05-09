# Interface: InterfaceName

## Purpose

What this interface represents — the concept or capability boundary it defines. Explain what callers can rely on and what the interface is responsible for enforcing.

## Public Concepts

List the core concepts, types, or entities exposed by this interface. These are the vocabulary callers must use.

- **ConceptName** — what it is and what it represents.
- **ConceptName** — what it is and what it represents.

## Inputs

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `param_name` | string | yes | What this parameter represents. |
| `optional_param` | number | no | What this optional parameter controls. Default: `0`. |

## Outputs

| Name | Type | Description |
|------|------|-------------|
| `result_name` | object | Shape and meaning of the returned value. |

Describe any error responses or failure modes the caller must handle.

## Allowed Dependencies

What this interface is permitted to depend on. Be specific — name the layers, modules, or external systems.

- Internal: `shared/types`, `shared/utils`
- External: none

## Disallowed Dependencies

What this interface must never depend on, and why.

- Must not import from `features/*` — would create circular dependencies.
- Must not call external network services directly — side effects belong at the application layer.

## Invariants

Conditions that must always be true when this interface is used correctly.

- Invariant: ...
- Invariant: ...

## Change Rules for Agents

Rules an agent must follow when modifying code that implements or consumes this interface.

- Do not remove or rename public fields without updating all callers.
- Inputs marked required may not be made optional without a deprecation step.
- Breaking changes require a version bump and entry in the changelog.
- When in doubt, extend rather than modify.
