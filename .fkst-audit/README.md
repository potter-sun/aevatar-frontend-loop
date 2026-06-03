# FKST Local Patch Audit

Date: 2026-06-03
Scope: local FKST package patches outside this control repo

This directory records the local FKST package delta needed for the Aevatar
branch-plus-PR trial. It is intentionally separate from the Aevatar host repo.

## Current Local FKST Package Delta

Package root:

```text
/opt/homebrew/lib/fkst/current/share/fkst
```

The current retained FKST package delta is captured in:

```text
.fkst-audit/fkst-pr-mode-local-patch-20260603.diff
```

The retained package delta is split into two groups.

## Keep For PR-Only Mode

These changes are the important behavioral guardrails for Aevatar:

- `departments/evolve/candidate_implement.lua`
  - Create a normal PR after a successful candidate branch push, even when the task was not sourced from a GitHub issue.
  - Mark the PR body as `mode=branch-pr-only`.
  - State that FKST must not auto-merge the candidate into the integration branch.

- `departments/auto_promote/main.lua`
  - When `github_pending_pr_branch` is present, skip raising `promote_request`.
  - This prevents an approved candidate from being automatically merged into the integration branch.

## Review Before Reverting

These changes mainly allow review/evidence logic to read runtime pipeline
artifacts directly instead of requiring every artifact to exist as a branch
blob:

- `departments/promoter/main.lua`
- `departments/evolve/branch_refs.lua`
- `departments/evolve/failure_archive.lua`
- `departments/review/consensus_flow.lua`
- `departments/review/context_helpers.lua`
- `departments/review/worktree_artifacts.lua`
- `scripts/review_evidence_gate.lua`

They are not GitHub decoration logic. Reverting them may make FKST review or
evidence gates fail when artifacts are stored in the runtime pipeline tree.

## Already Reverted

The mistaken FKST-native GitHub decoration patch has been reverted. These files
must stay identical to their `*.bak-fkst-pr-decoration-20260603` backups:

- `departments/github_publisher/main.lua`
- `departments/evolve/candidate_implement.lua` relative to the decoration backup

`github_publisher` must remain a low-level GitHub executor. It must not know
about `auto-loop`, phase labels, status cards, or the visible runbook.

## Correct Decoration Path

FKST-created PRs are decorated by the local codex-refactor-loop controller
contract:

```text
/Users/pottersun/.codex/skills/codex-refactor-loop/scripts/decorate_existing_pr.sh
```

The control repo adapter only forwards FKST `github_publisher` ack facts:

```text
scripts/decorate-fkst-pr-with-codex-loop
```

## Checks

Run this before continuing FKST automation:

```bash
scripts/check-fkst-local-patches
```

Expected result:

- PR-only local package deltas are present.
- Mistaken FKST-native decoration deltas are absent.
- FKST Lua syntax checks pass.
- The control adapter and local codex-refactor-loop helper parse successfully.
