# Agent Workflow Reference

Imported and adapted from
[`Cosmonaut-itc/personal-skills/docs/agents`](https://github.com/Cosmonaut-itc/personal-skills/tree/aa09c88491103633602a75653fcbd5096a679105/docs/agents)
at commit `aa09c88491103633602a75653fcbd5096a679105` (2026-08-04).

This directory is a disclosed reference for reusable project workflows. The
global contract in [`../orchestration.md`](../orchestration.md) and agent
selection in [`../agent-routing.md`](../agent-routing.md) remain canonical.

## Purpose

Use these modules when a project's instructions point to them:

- issue tracker conventions
- triage labels
- implementation workflow
- review cycle workflow
- review feedback workflow
- domain documentation expectations

Keep project-specific details in the project's `AGENTS.md`, `CONTEXT.md`, ADRs,
or local docs. Those files decide the tracker, commands and gates for that
project.

## What Belongs Here

- Generic issue state and implementation flow rules.
- Generic review and feedback loops.
- Generic guidance for using project domain docs.
- References to skills by name, not by local absolute path.

## What Does Not Belong Here

- Project names, product-specific domain terms, or release-specific plans.
- `.env*`, secrets, credentials, URLs with tokens, or vendor account details.
- Local machine paths or other absolute workstation paths.
- Stack-specific commands unless they are clearly examples and marked as
  replaceable by project commands.

## Activation

These files do not replace project documentation automatically. Before using a
module, confirm that the project's instructions select the same tracker and
workflow. A project can point to the relevant file here or maintain an adapted
local copy.

The GitHub tracker commands in [`issue-tracker.md`](issue-tracker.md),
[`review-cycle.md`](review-cycle.md), and
[`review-feedback-workflow.md`](review-feedback-workflow.md) apply only when
the project explicitly uses GitHub issues. Otherwise use the project's tracker
guide while preserving the workflow states and review boundaries that remain
compatible.
