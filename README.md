# Personal Agent Orchestration

Reusable orchestration documentation for Codex and Claude Code.

## Contents

- `docs/agent-routing.md`: canonical model-selection rubric.
- `docs/orchestration.md`: shared delegation contract.
- `configs/codex/`: global Codex guidance and the Codex-to-Claude worker bridge.
- `configs/claude/`: global Claude guidance and the Claude-to-Codex bridge.
- `docs/workflows/`: reusable issue, implementation, review, feedback, domain,
  and triage workflows.

The routing and delegation contract are platform-neutral. Each bridge owns only
the mechanics for its direction of travel, so model policy is not duplicated.
