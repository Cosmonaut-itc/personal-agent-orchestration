# Issue Implementation Workflow

This workflow applies after an issue has entered the tracker and has been triaged.

Before delegating or choosing a reviewer, read
[`../orchestration.md`](../orchestration.md) and
[`../agent-routing.md`](../agent-routing.md). They govern roles, model choice,
reasoning effort, required review passes and the antibucle rule; this file
governs the issue sequence.

## Label Model

Before changing labels, read [`triage-labels.md`](triage-labels.md). A triaged
open issue has exactly one category and one state; implementation progress is
additive.

## Ready For Agent

Use this path when the issue is fully specified and can be implemented AFK.

Required workflow:

1. Read the issue, linked PRD/plan, domain glossary, and ADRs.
2. Create or use the requested feature branch.
3. Use TDD for behavior changes.
4. Use subagent-driven development for non-trivial work.
5. Run spec compliance review.
6. Run code quality review.
7. Add `implemented: ready for review` when both reviews pass.

## Ready For Human

Use this path when the issue requires human judgment, credentials, product
approval, manual QA, legal/fiscal review, or external vendor decisions.

Rules:

- Keep `ready-for-human` unless the maintainer explicitly reclassifies it.
- If an agent implements it anyway by maintainer request, document the human
  decision points and verification constraints.
- Add `implemented: ready for review` only after the human-dependent checks are
  satisfied or explicitly waived by the maintainer.

## TDD Expectations

For runtime behavior changes:

1. Write one failing behavior test.
2. Implement the smallest change to pass.
3. Repeat vertically.
4. Refactor only while green.

Tests should verify public behavior, not private implementation details.

## Subagent Expectations

For larger issues:

- Fresh implementer subagent per task.
- Spec compliance review before code quality review. These are reviewer roles,
  not model selections; use the global routing rubric for each pass.
- Fix and re-review until clean.
- Do not run multiple implementer subagents in one worktree at the same time.
- Parallel exploration/review is allowed when write scopes do not conflict.

### Model And Effort

Select both from [`../agent-routing.md`](../agent-routing.md). If a worker
reports `BLOCKED`, `NEEDS_CONTEXT`, or unexpected complexity, the orchestrator
may re-dispatch or escalate under that rubric.

## Completion

`implemented: ready for review` means implemented but not closed.

`implemented: complete` means accepted or merged.

Only close an issue after `implemented: complete`.

Use [`review-cycle.md`](./review-cycle.md) to review issues with
`implemented: ready for review` before marking them complete or closing them.
