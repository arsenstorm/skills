---
name: triage
description: "Work through a repo's inbox — open PRs, Dependabot, code-scanning, and secret-scanning alerts — breadth-first: sweep every source, give every item one verdict (merge, fix, dismiss, ask), then act only on approved verdicts. Use when the user wants open PRs and security findings reviewed, investigated, or resolved. Triggers on: triage, review open PRs, security issues, security findings, github findings, dependabot, codeql, code scanning, secret scanning, vulnerability alerts, what needs attention, check the alerts, review the findings."
---

# Triage

A repo inbox holds open PRs and security findings. Triage gives every item one verdict before anyone fixes anything. The output is a verdict table and an ordered action plan, not a code review.

## Activation

### Use For

- "review open PRs and security issues", "investigate the findings", "what needs attention"
- a scheduled or recurring inbox sweep

### Do Not Use For

- a deep review of one diff — use `code-review`
- the state of your own work in progress — use `where-are-we`
- the fix itself — that work starts after the user approves a verdict

## Rule 1: Breadth Before Depth

Work in two passes:

1. **Sweep.** Visit every source below. For each item, read only the surface facts: metadata, diff stat, CI result, alert severity. When these facts are enough, give the verdict. When they are not, put the item on the deep-look list and continue.
2. **Deep look.** After the sweep is complete, return to each listed item. Read the diff, trace the code path, or read the changelog. Then give the verdict.

Why two passes: the first interesting PR can absorb a whole session, and the other nine items stay unread. The sweep makes sure that you judge every item before you study any item.

## The Sweep

Visit the sources in this order. Skip a source only when the host does not offer it.

1. **Open PRs** — `gh pr list`, then per PR: author (bot or human), CI result, conflicts, age, review state.
2. **Dependabot alerts** — `gh api repos/{owner}/{repo}/dependabot/alerts --jq '.[] | select(.state=="open")'`
3. **Code-scanning alerts** — `gh api repos/{owner}/{repo}/code-scanning/alerts?state=open`
4. **Secret-scanning alerts** — `gh api repos/{owner}/{repo}/secret-scanning/alerts?state=open`
5. **Security advisories** — `gh api repos/{owner}/{repo}/security-advisories`
6. **Issues with a security label** — `gh issue list --label security`

Why the list is this long: `gh pr list` shows only the visible part of the inbox. The security surface sits behind four separate APIs, and no API shows the items of another.

### Not Checked Is Not Clean

If a source returns 403 or 404, report it as "not checked" with the cause. Never report a source that you did not read as clean. Why: "clean" and "not checked" look the same in a summary, and the user trusts the summary.

## Verdicts

Every non-draft item gets exactly one verdict. List a draft PR as "draft" and give it no verdict. Why: a draft asks for time, not a decision.

| Verdict | Meaning |
|---|---|
| **merge** | Safe to merge as-is. |
| **fix** | Needs work. Name the work in one line. |
| **dismiss** | Close the item, with a written reason. |
| **ask** | Only the user can decide. Name the decision. |

### Bot dependency PR

```
Does CI fail? → fix (name the failed check)
Patch or minor bump, CI green? → merge
Major bump? → deep look: read the changelog for breaking changes
├── None touch this project → merge
└── One does → ask (name the breaking change)
```

Why the changelog step: a green build proves that the code compiles, not that behavior survived a breaking change.

### Human PR

```
Does CI fail? → fix (name the failed check)
Does it conflict with the base branch? → fix (rebase)
Approved and green? → merge
No activity for more than 30 days? → ask (close it or revive it)
Otherwise → ask (name the missing step, usually a review)
```

### Secret-scanning alert

The verdict is always **fix**, never dismiss. The fix has a fixed order:

1. Treat the secret as public from the moment of the alert.
2. Revoke or rotate the credential first. When only the user holds that access, the verdict becomes **ask** and names the credential.
3. Then remove the secret from the code. When the secret entered a commit, remove it from the history too.
4. Then close the alert.

Why revoke first: removal without revocation closes the alert and keeps the hole open. A history rewrite does not delete forks or clones.

### Code-scanning alert

Trace the flagged path in the code before any verdict. Why: the scanner reports patterns, and a pattern is not proof.

- The path is reachable → **fix**, with the severity from the alert.
- The path is dead code, a test fixture, or sanitized upstream → **dismiss**, and write the reason into the dismissal.

Never dismiss on low severity alone. A real fault with a low score is still a fault. Why the written reason: an unexplained dismissal reads as "ignored", and the next scan reopens the debate.

### Dependabot alert

- Dependabot opened a PR for it → judge the PR. The alert follows its PR.
- No PR exists → **fix** (bump the dependency), or **ask** when the bump is major.

## The Disclosure Gate

Before any action on a confirmed vulnerability, run `gh repo view --json visibility`.

If the repo is public, a fix PR, a pushed branch, or a detailed comment also discloses the fault. Attackers diff public changes, and the fix reaches users later. On a public repo, the verdict for a confirmed vulnerability is **ask**. It stays **ask** until the user gives express permission in this conversation. Silence is not permission. Offer the private paths: a GitHub private security advisory with its private fork, or the process in `SECURITY.md`.

Never put exploit steps, a PoC, or a reachable-path walkthrough into a PR, an issue, or a commit message. Keep the detail in the triage report for the user, not in public text.

## Rule 2: Triage Is Read-Only

During the sweep and the deep look, run only commands that read. Do not merge, dismiss, comment, rebase, or push. Present the report, then act only on the verdicts that the user approves. Why: a merge, a dismissal, or a comment is public and hard to undo. The user's approval also stops a wrong verdict before it becomes an action.

## Report Format

One line per item: source, item, severity or age, verdict, reason. Group by source. End with the proposed actions in this order:

1. Secret rotations. Why first: a live secret leaks while everything else waits.
2. Vulnerability fixes.
3. Merges.
4. Dismissals and closures. Why last: a dismissal loses nothing by an hour of delay.

```
PRs
- #41 dependabot: bump hono 4.6.1 → 4.6.3 — merge (patch, CI green)
- #39 feat: add webhooks — ask (green, no reviewer assigned)
- #35 chore: node 20 → 22 — fix (build failed on node:test import)

Security
- secret-scanning #3: payment API key in scripts/seed.ts — fix (rotate first)
- code-scanning #12 (high): SQL injection, apps/api/search.ts:88 — fix (path reachable from /v1/search)

Not checked
- security advisories (403: token lacks the scope)

Proposed actions
1. Rotate the payment key (you hold it). Then I remove it from code and history.
2. Fix the SQL injection on a branch. The repo is public — I need your go-ahead before any PR.
3. Merge #41. Rebase #35 and repair the build.
```
