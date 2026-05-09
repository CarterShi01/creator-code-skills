# Quality and Cost Harness for AI-Assisted Development

This document describes a reusable engineering approach for building software with AI coding agents under strict quality and token-cost constraints.

It was first developed while setting up CreatorMesh, an open-source personal agent operating system for independent creators.

## Original Problem

AI coding agents can be powerful, but they become expensive and risky as a codebase grows.

The initial concern was simple:

- Claude Code, Codex, Cursor, and similar tools may consume too many tokens.
- Developers and collaborators may not have enough budget for large model usage.
- As the codebase grows, agents may repeatedly read large amounts of source code just to rediscover the same architecture.
- Agents may modify files without understanding module boundaries.
- Agents may forget to update documentation when public interfaces change.
- Useful development experience may disappear after each session instead of becoming reusable knowledge.

This creates two kinds of debt:

1. Quality debt  
   The codebase becomes harder to maintain because agents make changes without respecting architecture, interfaces, tests, or human review.

2. Context debt  
   Agents repeatedly spend tokens rediscovering architecture, conventions, module boundaries, and decisions from raw source code.

The goal is to reduce both.

## Core Idea

The solution is to build a Quality Harness and a Cost Harness around AI-assisted development.

The harness is not one tool. It is a set of rules, documents, skills, and validation steps that guide agents before, during, and after code changes.

The key principle is:

> Interfaces before implementation. Plans before edits. Skills before repeated prompting.

Agents should understand the system through compact interface documents and context maps before reading implementation files.

## Quality Harness

The Quality Harness is designed to keep the codebase safe, understandable, and reviewable.

It includes:

- Clear architecture boundaries
- Directory-level README files
- Directory-level INTERFACE files
- Required planning before non-trivial edits
- Human approval before risky changes
- Interface maintenance after public contract changes
- Post-change review
- Skill harvesting after meaningful sessions
- Git diffs and explicit change summaries

The Quality Harness answers questions such as:

- What is this module responsible for?
- What does this module expose?
- What inputs does it accept?
- What outputs does it produce?
- What dependencies are allowed?
- What dependencies are forbidden?
- Did the change affect a public contract?
- Should an interface document be updated?
- Should this workflow become a reusable skill?

## Cost Harness

The Cost Harness is designed to reduce token usage and avoid repeated rediscovery.

It includes:

- `AGENTS.md` as the cross-agent instruction file
- `CLAUDE.md` as Claude Code-specific guidance
- `docs/context-map.md` as the compact codebase map
- `src/*/README.md` as directory purpose documentation
- `src/*/INTERFACE.md` as module contract documentation
- Project-level Claude Code skills
- A private reusable CodeSkill repository
- A strict context reading order
- Avoiding full-repository scans by default
- Turning repeated workflows into skills

The Cost Harness answers questions such as:

- What should the agent read first?
- Which directory is relevant to this task?
- What interface contract describes the target module?
- Can the agent solve the task without reading implementation files?
- Can repeated instructions be turned into a skill?
- Can expensive reasoning be converted into reusable documentation or scripts?

## Required Reading Order

Before making any non-trivial change, an agent should read context in this order:

1. `AGENTS.md`
2. `docs/context-map.md`
3. `docs/architecture.md`
4. Target directory `README.md`
5. Target directory `INTERFACE.md`
6. Only then, the specific implementation files needed for the task

The agent should not scan the whole repository unless explicitly asked.

## Interface-First Development

A software system can be understood as a tree of interfaces.

An interface is not only a TypeScript type or a function signature. It can exist at many levels:

- A repository has an interface.
- A source directory has an interface.
- A module has an interface.
- A class has an interface.
- A function has an interface.
- A workflow has an interface.
- A connector has an interface.
- A runner has an interface.

For AI coding agents, natural-language interface documents are especially useful.

An `INTERFACE.md` should describe:

- Purpose
- Public concepts
- Inputs
- Outputs
- Allowed dependencies
- Disallowed dependencies
- Invariants
- Main files
- Change rules for agents

This lets agents understand the contract before reading implementation details.

## DESIGN.md as Middle-Layer Context

During the CreatorMesh setup, another context gap became clear.

`README.md` is useful for explaining high-level module purpose and boundaries.

`INTERFACE.md` is useful for explaining public concepts, inputs, outputs, dependencies, invariants, and change rules.

However, there is a missing middle layer.

When working across Claude Code, ChatGPT, and human collaborators, a project often needs a compact design-context file that explains:

- What the current design is
- Why it is designed this way
- What tradeoffs were made
- What alternatives were considered
- What assumptions are currently active
- What open questions remain
- What another AI assistant needs to know to continue the design discussion

This file is called `DESIGN.md`.

### Why DESIGN.md Exists

`DESIGN.md` exists because AI-assisted development often moves across sessions and tools.

Claude Code may implement a change, but ChatGPT may be used later to discuss architecture, product direction, or design tradeoffs.

Without a middle-layer design document, the user has to repeatedly reconstruct context by pasting previous conversations, reading source code, or explaining the same design reasoning again.

That is expensive and fragile.

`DESIGN.md` turns design reasoning into reusable context.

### Position in the Context Stack

CreatorMesh uses the following context stack:

1. `AGENTS.md`  
   Cross-agent project rules and working discipline.

2. `docs/context-map.md`  
   Compact source map and reading guide.

3. `docs/architecture.md`  
   High-level system architecture.

4. `README.md`  
   Directory or module responsibility and boundaries.

5. `DESIGN.md`  
   Middle-layer design reasoning, tradeoffs, assumptions, alternatives, open questions, and ChatGPT handoff context.

6. `INTERFACE.md`  
   Public concepts, inputs, outputs, dependencies, invariants, and change rules.

7. Source code  
   Implementation details.

In short:

- `README.md` explains what this module is.
- `DESIGN.md` explains why it is designed this way.
- `INTERFACE.md` explains what contract it exposes.
- Source code explains how it is implemented.

### DESIGN.md and Cost Harness

`DESIGN.md` reduces token cost by preventing repeated rediscovery of design reasoning.

Instead of asking an AI assistant to infer design intent from source code or long conversation history, the assistant can read a compact design context file.

This is especially useful when switching between tools:

- Claude Code can summarize implementation and design reasoning into `DESIGN.md`.
- ChatGPT can read `DESIGN.md` to continue architecture or product discussion.
- Human collaborators can review `DESIGN.md` to understand why the current design exists.

This turns design reasoning into a reusable context artifact.

### DESIGN.md and Quality Harness

`DESIGN.md` also improves quality.

It helps preserve:

- Design goals
- Key decisions
- Tradeoffs
- Alternatives considered
- Current assumptions
- Open questions
- Future evolution
- ChatGPT handoff context

This makes architecture decisions easier to review, challenge, and improve over time.

It also prevents design drift, where implementation changes but the reasoning behind the system is forgotten.

### When to Create DESIGN.md

Create a `DESIGN.md` when a module, workflow, connector, runner, governance policy, or architecture area has meaningful design context that should be preserved.

Good candidates include:

- Non-trivial modules
- Multi-step workflows
- Connectors with integration tradeoffs
- Runners with execution tradeoffs
- Governance or approval policies
- Architecture decisions that may be revisited later
- Areas frequently discussed with ChatGPT or Claude

Do not create `DESIGN.md` files for trivial utilities unless they contain important design decisions.

### When to Update DESIGN.md

Update `DESIGN.md` when:

- A design decision changes
- A major tradeoff is discovered
- A new alternative is considered and rejected
- A module's responsibilities change
- A ChatGPT or Claude session produces useful design reasoning
- A feature is implemented in a way that future agents need to understand
- Current assumptions become outdated

### DESIGN.md Should Not Become

`DESIGN.md` should not become:

- A full implementation dump
- A changelog
- A duplicate README
- A duplicate INTERFACE file
- A transcript of every conversation
- A place for secrets or credentials

It should preserve design reasoning, not every detail.

### Design Context Maintainer Skill

To support this layer, a reusable skill can be created:

`creator-design-context-maintainer`

This skill should be used after:

- Architecture discussions
- Module design changes
- Complex implementation decisions
- Workflow design updates
- Connector or runner design decisions
- Governance or approval design decisions
- ChatGPT or Claude sessions that produce useful design reasoning

The skill should review whether a relevant `DESIGN.md` should be created or updated.

It should preserve useful design context without duplicating README, INTERFACE, or implementation details.

### Updated Reading Order

After adding `DESIGN.md`, the recommended reading order becomes:

1. `AGENTS.md`
2. `docs/context-map.md`
3. `docs/architecture.md`
4. Target directory `README.md`
5. Target directory `DESIGN.md`, if it exists
6. Target directory `INTERFACE.md`
7. Specific implementation files needed for the task

This preserves a clean progression:

Rules → Map → Architecture → Responsibility → Reasoning → Contract → Implementation

## Context Debt

Context debt is the hidden cost created when agents must repeatedly rediscover the same architecture, interfaces, conventions, and decisions from raw implementation code.

Context debt grows when:

- There is no context map.
- Directories do not have interface documents.
- Architecture decisions are only implicit in source code.
- Agent instructions are repeated manually in prompts.
- Reusable workflows are not converted into skills.
- Documentation drifts away from code.

Context debt is reduced by:

- Maintaining `AGENTS.md`
- Maintaining `docs/context-map.md`
- Maintaining `INTERFACE.md` files
- Using skills for repeated workflows
- Keeping summaries and decisions close to the code
- Checking interface drift after changes

## Project Skills

The first project-level skills created for CreatorMesh were:

1. `creator-context-navigator`  
   Guides the agent to read project rules, context maps, README files, and INTERFACE files before implementation code.

2. `creator-change-planner`  
   Produces a concise change plan before non-trivial edits.

3. `creator-interface-maintainer`  
   Checks whether `INTERFACE.md` files need updates after public contract changes.

4. `creator-skill-harvester`  
   Reviews a completed development session and identifies reusable workflows that may become future skills.

These skills are not just convenience prompts. They are part of the harness.

## Important Lesson About Skill Invocation

During validation, invoking a skill directly as:

`Skill(creator-context-navigator)`

returned:

`Unknown skill: creator-context-navigator`

However, Claude Code still followed the skill procedure when prompted in natural language.

This means the behavior goal passed even though formal skill registration or harness-level invocation was not available in that environment.

The practical workaround is to use natural language invocation:

`Use creator-context-navigator.`

or:

`Follow the creator-context-navigator procedure.`

Validation should focus on behavior, not just invocation syntax.

## Validation Strategy

Each harness component should be validated with a small test.

### Context Navigator Validation

Ask the agent to prepare for a future change in a target directory without editing files.

Expected behavior:

- It reads `AGENTS.md`.
- It reads `docs/context-map.md`.
- It reads `docs/architecture.md`.
- It reads the target directory `README.md`.
- It reads the target directory `INTERFACE.md`.
- It does not read unrelated implementation files.
- It does not edit files.

### Change Planner Validation

Ask the agent to plan a small future change without editing files.

Expected behavior:

- It identifies the target module.
- It reads the module README and INTERFACE.
- It produces a concise plan.
- It does not edit files.
- It asks for approval before broad changes.

### Interface Maintainer Validation

After a change, ask the agent whether any interface document needs an update.

Expected behavior:

- It identifies affected directories.
- It checks the relevant `INTERFACE.md` files.
- It updates only if public contracts changed.
- It explains if no update is needed.

### Skill Harvester Validation

After a meaningful session, ask whether a reusable skill was discovered.

Expected behavior:

- It identifies reusable patterns.
- It proposes skill candidates.
- It does not create skills automatically.
- It distinguishes between a skill, a document, a script, and a test.

## Recommended Development Flow

For a non-trivial change:

1. Use `creator-context-navigator`.
2. Use `creator-change-planner`.
3. Review and approve the plan.
4. Let the agent make the smallest necessary change.
5. Run relevant checks.
6. Use `creator-interface-maintainer`.
7. Use `creator-design-context-maintainer` when design reasoning should be preserved.
8. Use `creator-skill-harvester`.
9. Commit only after review.

This creates a controlled loop:

Context → Plan → Edit → Check → Interface Review → Design Context Review → Skill Harvest → Commit

## Private CodeSkill Repository

A private CodeSkill repository should store reusable skills and engineering playbooks that can be reused across projects.

Project-specific skills should stay inside the project repo until they become broadly reusable.

A private CodeSkill repository may include:

- `skills/`
- `templates/`
- `registry/`
- `docs/`

It should also include:

- Skill templates
- Interface templates
- Approved skill registry
- Skill changelog
- Third-party skill intake policy

## Third-Party Skill Intake Policy

When importing or learning from third-party skills:

1. Check the license first.
2. If reuse is allowed, preserve attribution.
3. If the license is unclear, do not copy the skill content.
4. Review any scripts before running them.
5. Do not import skills that require secrets or unsafe permissions.
6. Prefer small, focused skills with clear trigger conditions.
7. Record approved imported skills in a registry.

Some public skills are open source. Others may be source-available but not fully open source. Do not assume everything can be copied.

## Hooks Should Come Later

Hooks are useful, but they should not be added too early.

Hooks should only automate behavior that is already manually validated.

Early recommended hooks may include:

- Session-end reminder to run skill harvesting
- Edit tracking after file modifications

Avoid early hooks that:

- Automatically rewrite interface documents
- Automatically create skills
- Automatically commit changes
- Automatically push code
- Automatically make risky changes

Manual discipline should come before automation.

## Best-Practice Summary

The recommended order is:

1. Establish cross-agent rules.
2. Add a context map.
3. Add module interface documents.
4. Add project-level skills.
5. Manually validate the workflow.
6. Create a private reusable CodeSkill repo.
7. Harvest reusable skills after sessions.
8. Add hooks only after the manual process is stable.

## Core Principles

- Interfaces before implementation.
- Plans before edits.
- Skills before repeated prompting.
- Summaries before long context.
- Scripts before token-heavy reasoning.
- Human approval before expensive or risky actions.
- Every expensive session should produce reusable knowledge.
- Behavior validation matters more than invocation syntax.
- Do not automate a process before it is manually proven.
- Preserve design reasoning in DESIGN.md when it matters.
- Use DESIGN.md as the handoff bridge between Claude Code, ChatGPT, and human collaborators.

## One-Sentence Summary

The Quality and Cost Harness helps AI coding agents work like disciplined collaborators: they understand interfaces before reading code, plan before editing, preserve architecture quality, and turn expensive sessions into reusable knowledge.
