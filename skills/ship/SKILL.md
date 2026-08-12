---
name: ship
description: "Get finished work live in one supervised chain: commit, push, watch CI, correct failures, and prove the result with evidence. Use when the user wants changes committed and deployed, pushed and watched to green, or merged and live. Triggers on: ship, ship it, push it, push and deploy, commit and deploy, deploy this, watch ci, watch the build, get this live, get this merged, release this."
---

# Ship

To ship is one chain: commit, push, watch CI, correct failures, and prove the result. "Shipped" means that the furthest automated stage of the project completed with success on the pushed commit. For a project with deploy automation, that is the deploy. For a project with tests only, that is a green run. For a project with no CI, that is a clean push after local checks. The chain is not complete until you show that evidence.

## Activation

### Use For

- "ship it", "push it", "commit and deploy", "get this live", "watch CI"
- follow-through after a change is complete and approved

### Do Not Use For

- a commit message alone — use `commit`
- creation of a CI pipeline — that is a build task. When no pipeline exists, report it and offer to build one.
- a review of code quality — that belongs before this chain starts

## Workflow

Give the user a one-line status at the start of each stage.

### Stage 0: Inspect

Gather these facts before any action. Why: every later decision depends on them, and a wrong guess here costs a failed push or a history rewrite.

- `git status`, the current branch, and commits that are not pushed
- how the project ships: CI workflows, deploy config, package scripts
- how changes reach the default branch: read the last 20 commits. Merge commits or PR references mean that the project merges through PRs.
- whether commits are signed: read the git config for signature settings and the signature on the last commit

Then walk this tree:

```
Is the current branch the default branch?
├── No → push the branch, then open or update its PR
└── Yes
    ├── History shows PR merges → stop and ask the user
    │   (a direct push to a reviewed branch skips review and is hard to undo)
    └── History shows direct pushes → push directly
```

### Stage 1: Commit

1. If the project has a commit convention or a commit skill, obey it. If not, match the style of the last 20 messages in `git log`.
2. Before the commit, search the diff for secrets: keys, tokens, passwords, `.env` values. Why: after a push, a secret is public. Revocation is the only repair.
3. If the git config or the host requires signatures, make sure that the new commit is signed. Why: an unsigned commit on the remote forces a history rewrite.

### Stage 2: Local Checks

Run the same checks that CI runs, fastest first: lint, typecheck, build, tests. Why: a CI round trip costs minutes. The local check costs seconds and catches most failures.

If a check fails, correct the code and return to Stage 1.

### Stage 3: Push

- Never push with `--force` to a shared branch. Use `--force-with-lease`, and only after the user approves the rewrite. Why: a plain force push can delete remote commits that you did not fetch. `--force-with-lease` stops the push when the remote changed.
- Never use `--no-verify` or `[skip ci]`. Why: hooks and CI are the project's own guard rails. A skipped guard rail fails later, in public.

### Stage 4: Watch CI

Watch the run to its end with `gh run watch` or `gh pr checks --watch`. Do not end the turn while the run is active. Why: an unwatched run is not shipped. The confirmation is the product of this skill.

On failure, read the log of the failed step. Then classify the cause:

| Cause | Action |
|---|---|
| Code fault | Correct the code. Return to Stage 1. |
| Transient fault (network, runner, rate limit) | Rerun once. A second identical failure is a real fault — investigate it. |
| Missing permission or secret | Report the exact missing item and stop. Why: only the user can grant credentials. |

After three failed fix cycles, stop and report. Why: repeated blind fixes burn CI minutes and hide a wrong assumption about the cause.

### Stage 5: Prove the Result

Report the strongest evidence available. The levels, strongest first:

1. The deployed app serves the new version — a version endpoint, a health endpoint, or a visible change.
2. The deploy job completed with success.
3. CI is green on the pushed commit.
4. The push completed and the project has no CI. State this level plainly and offer to add CI. Why: a clean push proves delivery of code, not a working product.

Never report a lower level as a higher one.

## Report Format

```
Shipped: 3 commits on feat/pass-preview → PR #12 (merged)
CI: run 8123456789 green (lint, test, deploy) — 4m12s
Deploy: https://app.example.com/version returns 1.4.2 (new)
Left behind: uncommitted changes in docs/ (out of scope)
```

When anything stays unshipped, the "Left behind" line is required. Why: silence reads as "all of it shipped", and the user plans on top of that.

## Guard Rails

- Do not rewrite pushed history without an instruction from the user.
- One rerun per flaky job. No exceptions.
- Do not present a clean push as a deploy. Name the evidence level you reached.
