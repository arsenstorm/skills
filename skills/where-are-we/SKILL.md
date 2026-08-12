---
name: where-are-we
description: "Rebuild the state of a project from evidence — working tree, branches, PRs, CI, tasks, deploys — and report what is done, what is in flight, and one next action. Use at session start, after a compact, or on any status question. Triggers on: where are we, where were we, status, project status, what's next, whats next, what's left, continue from where you left off, catch me up, current state, review where we're at."
---

# Where Are We

After a break, a compact, or a hand-off, the conversation record is stale. The repo, the remote, and the tracker are current. This skill rebuilds project state from those sources and answers with short lists and one next action.

## Activation

### Use For

- "where are we", "status", "what's next", "catch me up"
- "continue from where you left off" — run the sweep first, then continue
- session start on a project with work in progress

### Do Not Use For

- a review of code quality — use a review skill
- a plan for a new feature — use `prd` or a plan
- questions about one specific commit or file — read it directly

## Rule 1: Evidence, Not Memory

Never answer a status question from the conversation summary alone. Why: a summary records what you intended. The repo records what happened, and the two drift apart. Make sure that a source in the sweep supports each claim in your report. Mark a claim with no source as "unverified".

## The Sweep

Run `git fetch` first, so remote facts are current. Then walk the sources in this fixed order. Skip a source only when the project does not have it. Why a fixed order: the same walk every session produces reports that the user can compare. The order also runs from the most fragile state to the most durable.

1. **Working tree** — `git status`. Uncommitted work is the most fragile state. It exists on one machine only.
2. **Branches** — the current branch, its commits ahead of the default branch, and other branches that are ahead.
3. **History** — the last 10 commits on the default branch.
4. **Pull requests** — open PRs with check status and review state, plus PRs merged since the last activity.
5. **CI** — the last 5 runs on the default branch.
6. **Tasks** — the tracker that the project uses: TODO or PLAN files, issue lists, or a connected task tool.
7. **Deploy** — the last deploy run, or a version endpoint when one exists.

"The last activity" means the end of the previous session. When that time is unknown, use the last 7 days.

## Rule 2: Read Only

Run only commands that read or fetch. Do not commit, push, rerun CI, or edit files. Why: the user asked for orientation. A change is a different task and needs their go-ahead.

## Rule 3: Surface Conflicts

When two sources disagree, report the conflict as its own line. Example: the task list marks a feature done, but the code does not contain it. Why: a silent choice between sources hides the drift that caused the status question.

## Report Format

Maximum 15 lines before any detail section. Each line names its evidence: a PR number, a commit, a run, a file.

- **Done** — work merged or deployed since the last activity.
- **In flight** — open PRs with their state, unpushed branches, uncommitted changes, failed runs.
- **Conflicts** — disagreements between sources. Omit this section when it is empty.
- **Next** — one action with a one-sentence reason.

Give exactly one next action. Why: a menu of options returns the decision to the user who asked to be oriented. Offer a second option only when two actions have an equal claim, and say why they are equal.

```
Done
- PR #12 (pass preview) merged — deploy run 81234 green

In flight
- feat/editor-canvas: 4 commits ahead of main, not pushed
- PR #13: checks green, no review requested
- Working tree: 2 modified files in apps/api, uncommitted

Next
- Push feat/editor-canvas and open its PR — PR #13 rebases on it.
```

Stop after the report. Do not start the next action until the user agrees. Why: the report is the deliverable. An action on top of it turns orientation into unrequested work.
