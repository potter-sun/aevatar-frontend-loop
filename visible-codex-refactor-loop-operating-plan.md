# Visible Codex Refactor Loop Operating Plan

Date: 2026-06-01
Owner: visible Codex supervisor
Scope: user-level runbook for Codex automation loops

## Purpose

This plan records the stable operating model for visible Codex automation loops after the Aevatar trial. It is a user-level runbook, not a project artifact. Host-specific rules belong in the heartbeat prompt or host overlay sections, while the loop mechanics should remain reusable across repositories.

The loop exists to discover real product and UI problems from a user perspective, reach consensus through the local `codex-refactor-loop` / `consensus-rnd` control framework, implement only agreed slices, open PRs against the configured integration branch, and stop before merge for user review.

## Stable Entry

- Reload the local skill on every round.
- Default skill path for this machine: `/Users/pottersun/.codex/skills/codex-refactor-loop/SKILL.md`.
- Do not use, copy, patch, or repair repo-local `.claude/skills/codex-refactor-loop/` unless the user explicitly authorizes it.
- Read the host repository rules file on every round.
- Verify `https://github.com/ChronoAIProject/consensus-rnd` is accessible on every round.
- State explicitly in the transcript that the stable entry skill is local `codex-refactor-loop`.

## Control Framework

Use this sequence as the default loop:

1. Product/UI issue phase from a real user perspective.
2. Three solver views: minimal, structural, delete.
3. Meta-judge consensus, converge, or reflector routing.
4. Split issue into narrow, reviewable implementation slices.
5. Implement in an isolated worktree and branch.
6. Create or update PR only when the file threshold is met.
7. Run reviewer x3.
8. Reflect state to GitHub through labels and status cards.
9. Stop at waiting for user review/merge.

The loop may use visible right-side subagents for auxiliary work, but the supervisor transcript stays in the visible main thread.

## PR Threshold

Effective from 2026-06-01:

- Do not create a PR until the automation batch has at least 10 meaningful tracked files.
- Exclude user dirty files.
- Exclude host-forbidden paths.
- Do not add files or churn code merely to hit the threshold.
- If fewer than 10 meaningful files are present, continue discovery, consensus, or implementation within the same theme.
- Exception: if the user explicitly requests a narrow fix or asks to open a PR for a smaller change, the loop may open a PR below the 10-file threshold. The status card must state the file count and explain that the threshold exception was user-authorized.

## Branch And PR Rules

- PR base must be the host-configured integration branch, never an unrelated default branch.
- For Aevatar, the integration branch is `auto-frontend-dev`.
- Before executing each implementation task, synchronize the remote `dev` branch into the configured integration branch.
- The sync must use remote facts: fetch `origin dev auto-frontend-dev`, update local `auto-frontend-dev` from `origin/auto-frontend-dev`, merge `origin/dev` into it, and push `auto-frontend-dev` back to `origin` only after the merge succeeds.
- If the sync has conflicts, requires destructive git operations, or cannot push cleanly, stop and report the blocker instead of continuing the task.
- Branch naming must follow the host repository rules.
- For Aevatar, branch naming is `<type>/YYYY-MM-DD_<purpose>`.
- Commit messages should be imperative.
- Never push directly to the integration branch except for the required `origin/dev` -> integration branch synchronization described above, unless the user explicitly authorizes another integration-branch push.
- Never run `gh pr merge`.
- Never enable auto-merge.
- Never simulate or click a GitHub merge action.
- Even with green CI and approvals, stop at `waiting for user review/merge`.

## GitHub Visibility

GitHub is the public status surface for the automation. Every PR created by this loop must immediately get labels and a status card.

### PR Status Transaction

Opening or updating an automation PR is not complete until GitHub state has been written and read back. Treat PR creation as a transaction with these required steps:

1. Create or update the PR against the configured integration branch.
2. Apply the required labels.
3. Post the first `## 📊 状态卡片` comment in the same turn.
4. Read the PR back through GitHub and verify both labels and the status card are present.
5. Only then report the PR as opened or ready for review.

If the readback cannot find a status card containing `⟦AI:AUTO-LOOP⟧`, the PR is not considered fully created. Retry the status-card write or report the failure explicitly in the visible supervisor transcript. Do not rely on PR labels, the PR body, Codecov comments, or local logs as a substitute for the status card.

Minimum readback check:

```bash
gh pr view "$PR_NUMBER" --repo "$GH_REPO_SLUG" --json labels,comments,statusCheckRollup,mergeStateStatus
```

The readback must confirm:

- `auto-loop` is present.
- The human/automation ownership label is present.
- The current phase label is present.
- At least one non-bot status card comment starts with `## 📊 状态卡片`.
- The status card contains `⟦AI:AUTO-LOOP⟧`.
- The status card states base branch, head branch, file count, validation, backend boundary, merge policy, and next action.

GitHub PR list pages show labels and checks but do not show status-card comments. Labels are an index into state, not the state explanation. When checking whether cards are complete, inspect PR comments directly.

Required labels for a new Aevatar automation PR:

- `auto-loop`
- `🤖 human:auto-推进`
- `🚀 phase:pr-open`

Add phase labels as work progresses:

- `👀 phase:reviewing` when reviewer x3 is active or pending.
- `⚙️ phase:ci-running` while checks are still running, if useful.
- `🔧 phase:fixing` when review or CI fix work is active.
- `⏸️ phase:blocked` only when genuinely blocked.

Status cards should say the issue/PR, phase, base branch, head branch, worktree, file count, validation, next action, and whether human action is needed.

### Phase Cards

Every phase transition needs a card, not only a label.

Required phase-card moments:

- PR opened: lifecycle summary card.
- Reviewer x3 dispatched: reviewer dispatch card with roles and expected next action.
- Reviewer x3 completed: reviewer result card with approve/comment/reject summary.
- CI running: CI tracking card or explicit note in the lifecycle card if checks are already queued.
- CI green: CI result card naming the required successful checks.
- CI failed or reviewer rejected: fixing card naming the blocker and the owner.
- Conflict resolved: conflict card naming files and validation.
- Waiting for user review/merge: final waiting card that states no merge or auto-merge was attempted.

Labels may move ahead of comments only inside the same short transaction. A turn must not end with phase labels changed but no matching status card, unless GitHub write failed and the visible transcript reports that failure.

### Ownership Guard

Before writing comments, labels, commits, pushes, or conflict resolutions on a PR, confirm the PR belongs to the user or that the user explicitly authorized work on that PR.

For Aevatar, if a PR is not opened by `potter-sun`, default to read-only inspection. Do not comment, edit labels, resolve conflicts, push, or otherwise mutate the PR unless the user explicitly asks to act on that specific non-user PR.

## Host Forbidden Paths

Each host repository may define paths the loop can inspect but not modify.

For Aevatar, do not modify:

- backend
- proto
- API
- actor
- projection
- workflow
- runtime
- database
- backend tests
- backend config
- architecture docs

If an Aevatar backend error appears, only locate, reproduce, record, attribute, and risk-label. Write exactly:

`后端错误已记录，按用户要求未修改`

## Worktree Safety

Every round must:

- Record host repo `git status --short --branch`.
- Avoid `git reset --hard`, destructive checkout, or reverting user changes.
- Implement only in an isolated worktree/branch.
- Never use user dirty files to meet the PR file threshold.

## Round Transcript

Every heartbeat round should output a complete Chinese transcript with these sections:

1. 标题
2. 目标
3. skill reload
4. command log with cwd, command, exit, output
5. git branch status
6. concurrency status
7. PR/issue status
8. decision chain
9. backend error record
10. cumulative file count
11. verification
12. Human: challenge
13. next step
14. shortest conclusion

If content is too long, split into Part 1/N, Part 2/N, and keep going.

## Verification

Every round must run at least:

```bash
git diff --check
```

Paste the result in the transcript. If there is no output, state `<no output>`.

If tests are modified, run the host-required test stability guard.

For Aevatar:

```bash
bash tools/ci/test_stability_guards.sh
```

For frontend changes, run focused tests and type checks appropriate to the host app. Use the in-app Browser for visible UI verification when a dev server or local page is available.

## Concurrency And No-Gap Rule

Each round should check active work and active codex processes related to this loop.

- Active phase issue/PR plus zero loop workers is a no-gap bug unless all work is explicitly waiting for the user.
- If the local skill scripts are usable, prefer their peek/concurrency semantics.
- If repo-local scripts would require touching `.claude/skills/codex-refactor-loop/`, do not touch them. Use conservative `ps`/`pgrep`, GitHub issue/PR state, and logs.
- After spawning workers, confirm GitHub has a status card and labels.

## Aevatar Host Overlay

- Control repo: `/Users/pottersun/Desktop/sbt_projects/aevatar-frontend-loop`
- FKST wrapper: `/Users/pottersun/Desktop/sbt_projects/aevatar-frontend-loop/scripts/fkst-aevatar`
- FKST task creator: `/Users/pottersun/Desktop/sbt_projects/aevatar-frontend-loop/scripts/add-aevatar-task`
- Integration sync script: `/Users/pottersun/Desktop/sbt_projects/aevatar-frontend-loop/scripts/sync-aevatar-integration`
- FKST wrapper config: `/Users/pottersun/Desktop/sbt_projects/aevatar-frontend-loop/fkst-aevatar.env`
- Repo: `/Users/pottersun/Desktop/sbt_projects/aevatar`
- Remote: `git@github.com:aevatarAI/aevatar.git`
- Rules file: `/Users/pottersun/Desktop/sbt_projects/aevatar/AGENTS.md`
- Stable entry skill: `/Users/pottersun/.codex/skills/codex-refactor-loop/SKILL.md`
- Current local branch at last verification: `dev`
- Integration branch: `auto-frontend-dev`
- Remote integration branch: `origin/auto-frontend-dev` exists; the local branch may need `git fetch` and checkout before use.
- Required pre-task sync: merge remote `origin/dev` into `auto-frontend-dev` and push the updated integration branch before each implementation task.
- Recommended pre-task sync command from the control repo: `./scripts/sync-aevatar-integration`
- Run FKST from the control repo with `./scripts/fkst-aevatar <command>` so the current shell stays in the control repo while FKST targets the Aevatar host repo.
- Verified FKST commands from the control repo:
  - `./scripts/fkst-aevatar status`
  - `./scripts/fkst-aevatar doctor`
  - `./scripts/fkst-aevatar start --print`
- FKST runtime root: `/Users/pottersun/.local/state/fkst/runtime/Users-pottersun-Desktop-sbt_projects-aevatar`
- Create new FKST tasks from the control repo with `./scripts/add-aevatar-task "Task title" [slug]`. The task creator injects this runbook path, Aevatar host rules path, integration branch, forbidden backend boundary, PR threshold reminder, verification requirements, and the `⟦AI:FKST⟧` sentinel into the inbox task.
- Use `./scripts/add-aevatar-task --dry-run "Task title" [slug]` to preview the generated inbox body without writing a task.
- Keep #1642 untouched unless the user explicitly asks.
- Do not put thread/bootstrap/supervisor-carrier debugging content into unrelated GitHub issue cards.
- New product/UI themes should get new issues; do not attach all future work to #1636.
- Localization and English/Chinese consistency are out of scope for this loop because the user plans a separate PR.

## Quick Wakeup Checklist

1. Reload local skill and host rules.
2. Run `./scripts/fkst-aevatar doctor` from the control repo.
3. Use `./scripts/add-aevatar-task --dry-run "Task title"` before adding any new FKST inbox task.
4. Synchronize `origin/dev` into `auto-frontend-dev`; stop and report conflicts or push failures.
5. Verify `consensus-rnd`.
6. Record main worktree status.
7. Inspect open automation PRs and labels.
8. Run `git diff --check`.
9. Check whether active work has no worker or missing status labels.
10. Continue the consensus/implementation/review route.
11. Enforce the 10-file PR threshold.
12. Post visible transcript and GitHub status as needed.
13. Stop before merge.
