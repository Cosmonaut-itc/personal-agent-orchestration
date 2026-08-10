# Triage Labels

The skills speak in terms of five canonical triage roles. This file maps those
roles to the label strings expected in a project issue tracker.

| Canonical role | Default label | Meaning |
| -------------- | ------------- | ------- |
| `needs-triage` | `needs-triage` | Maintainer needs to evaluate this issue. |
| `needs-info` | `needs-info` | Waiting on reporter for more information. |
| `ready-for-agent` | `ready-for-agent` | Fully specified, ready for an AFK agent. |
| `ready-for-human` | `ready-for-human` | Requires human implementation. |
| `wontfix` | `wontfix` | Will not be actioned. |

When a skill mentions a role, use the corresponding label string from this table.

Project repos may override the default label strings in their local docs.

## Implementation Progress Labels

These labels are additive progress labels. They do not replace the canonical
category/state labels used by triage.

| Progress label | Meaning |
| -------------- | ------- |
| `implemented: ready for review` | Work exists in a branch/PR, relevant tests passed, and the issue is ready for maintainer review. |
| `implemented: complete` | The maintainer accepted the implementation or the PR was merged; the issue can now be closed. |

Rules:

- Every triaged issue still needs exactly one category label: `bug` or `enhancement`.
- Every open issue still needs exactly one state label: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, or `wontfix`.
- `implemented: ready for review` can be added to `ready-for-agent` or `ready-for-human` work once the implementation is ready to review.
- Do not close an issue only because it has `implemented: ready for review`.
- Add `implemented: complete` only after maintainer acceptance or PR merge.
- Close issues only after `implemented: complete` is present.
