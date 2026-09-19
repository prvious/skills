---
name: plan-issue
description: Plan a GitHub issue or feature into an implementation-ready plan in one pass. Use this skill whenever the user asks to plan an issue or feature, work from a GitHub issue number, break work into tasks or commits, run a plan-first flow, or pastes an issue reference expecting discovery before code. Produces the complete plan and stops; it never implements.
---

# Plan a GitHub Issue

You are producing an implementation-ready plan for a single GitHub issue or feature. This skill runs in Plan mode only. Produce the complete plan in one pass and stop. Never implement.

Follow all repository instructions and required skills.

## Discovery — read-only

Do not edit or generate files, install dependencies, change GitHub state, create/switch branches, stage, commit, push, or open PRs.

1. Read the issue, comments, linked parent/child issues, PRD, architecture notes, and other referenced material.
2. Trace the current implementation and compare it with the requested behavior and acceptance criteria. Treat existing code as an interim baseline, not proof that the issue is complete.
3. Resolve every question answerable from the repository or linked material. For the remainder, make a reasonable assumption, record it under assumptions, and continue. Ask the user a question only when missing information makes a meaningful plan impossible — never for routine preferences or decisions inferable from context.
4. Identify conflicts, missing enablers, risks, and scope boundaries in the same pass. Keep optional refactors and improvements out of the required plan; list them separately.

## Output — always use this template

```markdown
# Implementation Plan

## Objective

## Current Behavior

## Desired Behavior

## Scope

### Required

### Optional / Out of Scope

## Acceptance Criteria

## Technical Decisions and Assumptions

## Implementation Tasks

### Task 1 — <name>

- Purpose
- Changes
- Relevant files
- Tests
- Verification
- Commit boundary

### Task 2 — <name>

...

## Final Verification

## Review Loop

## Risks and Edge Cases

## Open Questions
```

Each task must be independently reviewable with exactly one commit boundary, meaningful regression tests, and concrete verification. Name the formatter, linter, static checks, code-simplifier pass, and broader affected test/build suite the Build-mode execution must run — as plan content, not as actions you perform. The plan must also specify the review loop below as required Build-mode content: after implementation and verification, and before opening a PR, (1) hand the diff to a reviewer, fix-worthy findings only, loading `pr-review-toolkit` and `ponytail-review` first; (2) pass the findings to a notetaker, which decides FIX or DECLINE per finding with a one-line rationale; (3) implement the FIX findings, re-verify, and repeat from step 1 until no new fix-worthy findings remain.

## Stop condition

Present the plan and stop. If the user gives corrections, update the plan and stop again. Do not invite implementation approval, do not offer to implement, branch, commit, push, open a PR, or check CI, and do not describe phases. The user controls the switch to Build mode and requests implementation separately.
