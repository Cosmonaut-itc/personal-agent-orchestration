# Issue Tracker: GitHub

Use this module only when the project's instructions explicitly place issues
and PRDs in GitHub. If the project selects another tracker, use its local guide
instead; the GitHub commands below are not portable.

For a GitHub-backed project, use the `gh` CLI for all operations.

## Conventions

- **Create an issue**: `gh issue create --title "..." --body "..."`. Use a heredoc for multi-line bodies.
- **Read an issue**: `gh issue view <number> --comments`, filtering comments by `jq` and also fetching labels.
- **List issues**: `gh issue list --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'` with appropriate `--label` and `--state` filters.
- **Comment on an issue**: `gh issue comment <number> --body "..."`
- **Apply / remove labels**: `gh issue edit <number> --add-label "..."` / `--remove-label "..."`
- **Close**: `gh issue close <number> --comment "..."`

Infer the repo from `git remote -v` - `gh` does this automatically when run inside a clone.

## When a skill says "publish to the issue tracker"

Create a GitHub issue.

## When a skill says "fetch the relevant ticket"

Run `gh issue view <number> --comments`.

## Workflow Routing

This file owns GitHub transport only. After triage, use
[`implementation-workflow.md`](implementation-workflow.md).

Issues that have `implemented: ready for review` should follow
[`review-cycle.md`](review-cycle.md) before they are marked
`implemented: complete` and closed.

Issues with review feedback or requested changes should follow
[`review-feedback-workflow.md`](review-feedback-workflow.md) before requesting
review again.
