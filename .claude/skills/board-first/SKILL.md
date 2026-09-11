---
name: board-first
description: Board-first GitHub workflow for this repo. Use when planning work, creating or updating issues, starting an issue ("start 67"), raising a PR, or finishing/verifying an issue or parent [...]
---

# Board-first workflow

The GitHub Project board is the only plan. No local PLAN/DESIGN docs.

## Board facts

| | |
|---|---|
| Repo | `ramjawade/my-farm` (push to remote `origin`) |
| Board | project **6** "farm", owner `ramjawade` — https://github.com/users/ramjawade/projects/6 |
| Project id | `PVT_kwHOAse-7M4Bi2uX` |
| Status field id | `PVTSSF_lAHOAse-7M4Bi2uXzhhtO_w` |
| Status options | **Todo** `f75ad846` · **In Progress** `47fc9ee4` · **Done** `98236657` |
| Labels | `bug` `enhancement` `documentation` (no backend/frontend labels) |

Status meaning: **Todo** = planned, awaiting approval or not started. **In Progress** = branch exists, including while its PR is in review (there is no "In Review" column). **Done** = set automatically when the issue closes.

## Commands

```bash
# Create an issue on the board (lands in Todo)
gh issue create --title "…" --label bug --project "farm" --body-file body.md

# Board item id for issue N
gh project item-list 6 --owner ramjawade --limit 200 --format json \
  --jq '.items[] | select(.content.number==N) | .id'

# Move status (use an option id from the table)
gh project item-edit --project-id PVT_kwHOAse-7M4Bi2uX \
  --field-id PVTSSF_lAHOAse-7M4Bi2uXzhhtO_w --id <ITEM_ID> \
  --single-select-option-id 47fc9ee4

# Link child issue C under parent P
gh api repos/ramjawade/my-farm/issues/P/sub_issues -F sub_issue_id=$(gh api repos/ramjawade/my-farm/issues/C --jq .id)
```

## Flow

1. **Plan** — write the issue body with the template below; big work = parent issue + one sub-issue per PR. Stop and wait for the user to approve. No code before approval.
2. **Start** ("start N") — read issue N (it is the plan), then:
   `git fetch origin && git checkout -b claude/feature-N-<slug> origin/main`, and move the item to **In Progress**.
3. **Build** — follow the plan exactly. If it must change, comment on the issue and ask; don't patch silently.
4. **Gates** — `npm run lint` and `npm run build` for frontend changes. Backend: `ruff`, `mypy`, `pytest` (Python may not be on PATH locally — then say so; CI runs them). Never claim a gate passes if you didn't run it.
5. **PR** — commit messages end with `(Fixes #N)` on the final commit; push `-u origin`; `gh pr create` with the PR template. Leave the item in In Progress.
6. **After merge** (user says merged) — `git checkout main && git pull origin main && git branch -D <branch>`. Check the item reached Done.
7. **Parent done?** — verify every sub-issue is closed *and* every requirement in each body exists in the code (grep for it). A closed issue is not proof the work was built.

Bugs found along the way go into a new issue in Todo, not into the current PR.

## Issue template

```markdown
## Overview
One or two sentences: what and why.

## Requirements
- …

## Implementation Notes
- Files/services to touch, risks, dependencies

## Related
- Depends on / Blocks: #…

## Acceptance Criteria
- [ ] …
- [ ] lint / build / tests pass; merged to main

## Status
⏳ Backlog (awaiting approval)
```

## PR template

```markdown
Fixes #N

## Plan (from issue)
…

## Changes
- file — what changed

## Testing
- what ran, what passed (only what actually ran)

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

## Rules

- Branches: `claude/feature-N-<slug>` or `claude/bugfix-N-<slug>`; one feature per PR.
- Never delete issues — close as "not planned" with a comment.
- Never force-push `main`; never skip hooks.
- Delete a branch right after its PR merges.
- Destructive or shared actions (deleting files/issues, closing issues, merging) need the user's go-ahead.
