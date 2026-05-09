# Project Memory and Context Brief

This document describes reusable practices for preserving long-term project memory and handing context to another LLM or collaborator efficiently.

These practices were first developed during CreatorMesh development, where AI-assisted sessions accumulate decisions, direction changes, and progress across weeks and multiple tools.

## The Problem

AI coding sessions do not persist memory across conversations.

After a session ends, the decisions made, the architecture discussed, and the progress achieved are lost unless they are written down somewhere the next LLM can read.

This creates two problems:

1. **Direction drift.** Without a persistent project goal document, the project can slowly drift away from its original strategic intent.

2. **Progress illusion.** Without a persistent project progress document, it is easy to overstate what has been built, what has been validated, and what remains.

The solution is to keep versioned documents as project memory.

## Practice 1: Versioned Project Goal Document

A project goal document captures the strategic direction of the project at a point in time.

It is named with a date:

```
docs/project-goal-YYYYMMDD.md
```

A project goal document should include:

- Mission
- Target users
- Near-term focus
- Core primitives or concepts
- First concrete integration direction
- Important constraints
- Long-term positioning

When direction changes significantly, create a new dated version rather than editing in place.

The previous versions serve as a history of how the project evolved.

### What a Project Goal Document Is Not

A project goal document is not a roadmap.

It is not a task list.

It is not a specification.

It is a compact strategic record of what the project is trying to do and why.

## Practice 2: Versioned Project Progress Document

A project progress document captures the current state of the project at a point in time.

It is named with a date:

```
docs/project-progress-YYYYMMDD.md
```

A project progress document should include:

- Current phase
- Completed work
- Current focus
- Next work
- Known risks
- Progress summary

It should distinguish clearly between what is done, what is planned, what is partially complete, and what is missing.

When meaningful progress is made, create a new dated version or update the latest version.

### What a Project Progress Document Is Not

A project progress document is not a commit log.

It is not a list of every file changed.

It is a compact progress record that allows the next agent or collaborator to understand where the project currently stands without reading git history or long conversation logs.

## Practice 3: Context Brief for LLM Handoff

A context brief is a compressed version of project context designed for export to another LLM.

It is useful when:

- Switching from Claude Code to ChatGPT or another tool
- Starting a new session with no prior context
- Handing a task to a collaborator who was not part of recent sessions

A context brief should include:

- Project mission summary
- Current architecture overview
- Relevant module context
- Design decisions relevant to the task
- Interface contracts the next LLM must respect
- Constraints the next LLM must not violate
- What the next LLM should help with

A context brief is controlled by a compression parameter between 0 and 1.

- 0 means more detailed.
- 0.5 means balanced.
- 1 means highly compressed.

The compression parameter allows the brief to be tuned based on how much context the destination tool can receive.

A context brief should not include secrets, raw diffs, or full source files.

## Practice 4: Evidence-Based Goal and Progress Updating

Versioned project goal and progress documents should not be static artifacts.

They should be updated as the project direction, repository state, architecture, and completed work evolve.

The project goal document records the strategic direction.

The project progress document records the evidence-based current state.

This creates a durable project memory that does not depend on chat history.

### Project Goal Update

The project goal document should be updated when:

- the mission becomes clearer
- target users are clarified
- the near-term focus changes
- the first concrete integration direction changes
- important constraints are added
- the long-term positioning evolves

For example, during CreatorMesh development, the project goal became clearer around:

- independent creators as the target audience
- Thoughts and Messages as the two core input primitives
- Notion as the first knowledge workspace direction
- Claude Code as the first development execution environment
- quality and cost harness as a first-class engineering principle
- human-in-the-loop control as a default design principle

### Project Progress Update

The project progress document should be updated based on repository evidence.

It should inspect what actually exists before claiming completion.

Examples of repository evidence include:

- README.md
- AGENTS.md
- CLAUDE.md
- docs/context-map.md
- docs/context-architecture.md
- docs/cost-control.md
- docs/design-context.md
- docs/context-brief.md
- docs/project-goal-*.md
- docs/project-progress-*.md
- src/*/README.md
- src/*/INTERFACE.md
- templates/*
- .claude/skills/*/SKILL.md
- actual implementation files

The progress document should distinguish:

- Present
- Missing
- Planned
- Partially present
- Present but unvalidated

This distinction is important.

A file or skill can be present but not validated.

A feature can be planned but not implemented.

A concept can be documented but not yet wired into product logic.

### Why Evidence Matters

AI assistants can easily overstate progress if they rely only on conversation history.

Evidence-based progress updates reduce this risk.

They force the agent to check the repository before updating project memory.

This improves quality by keeping status honest.

It also reduces context cost because future agents can read the latest progress document instead of reconstructing the project state from chat history.

## Practice 5: Progress Maintainer Skill

A project can use a dedicated progress-maintenance skill to keep progress documents current.

This skill is different from a design-context maintainer or interface maintainer.

Its job is to update project progress based on actual evidence.

A good progress maintainer skill should be used after meaningful development sessions, such as:

- code feature implementation
- documentation layer changes
- new skill creation
- interface changes
- design context updates
- harness validation
- project management document updates
- meaningful project status changes

It should not be used for trivial typo fixes unless the progress document would otherwise become misleading.

### Responsibilities

A progress maintainer should:

- find the latest project-progress-*.md file
- read the latest project-goal-*.md file
- inspect repository evidence relevant to the current session
- identify what changed
- classify changes honestly
- update current phase, completed work, current focus, next work, known risks, and progress summary when needed

### Classification

Progress should be classified using clear statuses:

- completed
- partially completed
- planned
- missing
- present but unvalidated

This prevents false progress.

For example:

A skill file may exist, but if invocation has not been tested, it should be marked as present but unvalidated.

A connector may be planned, but if there is no implementation, it should be marked as planned or missing, not completed.

### Rules

A progress maintainer skill should follow these rules:

- Do not claim completion without repository evidence.
- Distinguish present from validated.
- Distinguish planned from implemented.
- Preserve useful existing progress history.
- Avoid rewriting the whole document unless necessary.
- Keep progress practical and current.
- Do not include secrets.
- Do not include long raw diffs.
- Do not make commits automatically.

### Relationship to the Harness

The progress maintainer skill is part of the post-change harness.

It complements:

- interface review
- design context review
- skill harvesting

A practical post-change flow becomes:

1. Review interface impact.
2. Review design context impact.
3. Review project progress impact.
4. Review reusable skill candidates.
5. Commit only after review.

## Practice 6: Context Brief as Next-Step Recommender

The context brief skill originally focused on exporting project context for another LLM.

A newer refinement is to make it useful even when the user does not provide a target goal.

The context brief can support two modes:

1. Empty goal mode
2. Goal-focused mode

### Empty Goal Mode

If goal is empty, the context brief should:

- read the latest project-goal-*.md file
- read the latest project-progress-*.md file
- summarize the whole project
- identify current focus
- recommend several possible next tasks
- ground recommendations in actual project progress
- avoid inventing missing capabilities

This mode turns the context brief into a project navigation aid.

It helps the user decide what to do next.

### Goal-Focused Mode

If goal is not empty, the context brief should treat goal as the target requirement.

It should:

- identify modules relevant to that goal
- preserve goal-relevant design context
- compress unrelated background more aggressively
- include constraints the next LLM must respect
- explain what the next LLM should help with

For example:

goal: Add a minimal Message domain primitive

The context brief should focus on:

- the two input primitives: Thought and Message
- the core module
- existing Thought implementation if present
- relevant README, DESIGN, and INTERFACE files
- constraints against adding Notion, storage, workflows, connectors, or UI too early

### Compression Behavior

The compression parameter remains a value between 0 and 1.

- 0 means more detailed.
- 0.5 means balanced.
- 1 means highly compressed.

If goal is provided, goal-relevant content should be compressed less aggressively than unrelated background.

If goal is empty, compression controls the overall brief length, but next-task recommendations should remain actionable.

### Recommended Output Change

A context brief should include a section such as:

Recommended Next Tasks or Next LLM Help

If goal is empty, this section should recommend practical next tasks.

Each recommended task can include:

- task name
- why it is next
- expected scope
- risk level
- whether it is suitable for harness validation

If goal is not empty, this section should explain what the next LLM should help with for that target requirement.

## Why Context Briefs Matter

Without a context brief, handing off to another LLM requires:

- Pasting raw conversation history
- Re-explaining architecture that was already decided
- Risking misunderstanding about what constraints exist
- Repeating costly reasoning that was already done

A context brief compresses the right information into a form the next LLM can use immediately.

It turns multi-session project knowledge into a portable handoff artifact.

## Recommended Workflow

1. Maintain project-goal-YYYYMMDD.md.
2. Maintain project-progress-YYYYMMDD.md.
3. Use Claude Code to inspect the repo and update progress honestly.
4. Use a progress maintainer skill after meaningful project changes.
5. Use a context brief skill when handing context to ChatGPT or another LLM.
6. If no target goal is provided, use the context brief to recommend next possible tasks.
7. If a target goal is provided, use the context brief to generate goal-focused context.
8. Ask the next LLM to reason from the brief, not from vague memory.
9. When design changes, update DESIGN.md.
10. When interface contracts change, update INTERFACE.md.
11. When project direction or progress changes meaningfully, update or create new dated goal and progress documents.

## Important Rules

- Keep goal and progress documents dated and versioned.
- Do not edit old versions in place; create new dated versions when direction changes significantly.
- Context briefs should compress intelligently, not truncate randomly.
- Context briefs should never include secrets, credentials, or long raw diffs.
- A context brief is not a full README. It is a handoff artifact.
- Update progress from repository evidence, not memory alone.
- Mark present-but-unvalidated capabilities honestly.
- When goal is empty, context briefs may recommend next tasks based on latest goal and progress.
- When goal is not empty, context briefs should preserve goal-relevant context more than unrelated background.
- Progress maintenance is part of the post-change harness.

## Reusable Skill Ideas

These skill patterns emerged from CreatorMesh development and may be useful in other projects.

1. project-goal-updater
   Updates or creates a new dated project goal document when strategic direction changes.

2. project-progress-updater
   Updates or creates a new dated project progress document based on repository evidence and session changes.

3. context-brief-generator
   Generates a compressed context brief for export to another LLM, with a compression parameter between 0 and 1.

4. creator-design-context-maintainer
   Reviews whether a DESIGN.md should be created or updated after architecture discussions or design decisions.

5. project-progress-maintainer
   Updates the latest project progress document after meaningful development sessions based on repository evidence.

6. next-task-context-brief-generator
   Generates a whole-project context brief and recommends next possible tasks when no target goal is provided.

7. goal-focused-context-brief-generator
   Generates a compressed context brief for a specific target requirement.

## One-Sentence Summary

Versioned project goal and progress documents preserve long-term project memory; progress maintainer skills keep that memory evidence-based; and context briefs compress the right context or next-task recommendations for the next LLM or collaborator.
