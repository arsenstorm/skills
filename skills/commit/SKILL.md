---
name: commit
description: Write git commit messages and pull requests in one strict convention: a one-line message (type(scope): message, no body, no footers) and a short typed PR description. Use whenever committing, amending, rewording, writing a commit message, opening a pull request, or writing a PR title or description. Triggers on: commit, commit message, git commit, amend, PR, pull request, PR description, PR title, open a PR.
---

# Commit

## Commit messages

Every commit message is exactly one line: `type(scope): message` or `type: message`.

Never add a body, a blank line, a footer, or a trailer. No `Co-Authored-By`, no "Generated with Claude Code" line. Why: the diff is the body. An explanation that needs more space belongs in the PR description, where reviewers read it. In the log it is only noise between messages.

### Pick the type

Walk this list top to bottom. Take the first match:

1. Fixes a vulnerability or hardens security → `security`
2. Reverts an earlier commit → `revert`
3. Repairs broken behavior → `fix`
4. Adds or changes behavior or capability → `feat`
5. Restructures code with identical behavior → `refactor`
6. Improves only speed or resource use → `perf`
7. Touches only documentation → `docs`
8. Touches only tests → `test`
9. Everything else (deps, config, tooling, releases) → `chore`

Why first-match: one change often qualifies for two types (a security patch is also a fix). First-match means the same change always gets the same label.

If a change needs two types, it is two commits. Split it.

### Pick the scope

1. Run `git log --oneline -20` and collect the scopes already in use.
2. If the change is confined to one app, package, or subsystem → use that name as the scope. Reuse an existing scope spelled exactly as before, never a synonym (`web`, not `frontend`, if the log says `web`).
3. If the change spans the repo or has no single home → omit the scope.

Why exact reuse: `git log --grep 'fix(web)'` finds every web fix only if the scope has one spelling.

### Write the message

- Imperative present: `add`, `remove`, `stop`. Never `added`, `adds`, or `adding`. Test: the message must complete "this commit will …".
- Lowercase after the colon. No period at the end.
- The full message stays under 72 characters. A longer message means the commit does two things or explains itself. Split the commit, or move the explanation to the PR.
- Say what changed, not that something changed: `fix(web): stop double submit on login form`. Banned: `fix bug`, `update files`, `improve code`, `address feedback`.

## Pull requests

Write the title with the commit-message rules above. A one-commit PR takes that commit's message as its title.

For the body, pick the template for the PR type and fill it. Add nothing else. Keep the body under 10 lines and each sentence under 20 words. Why: reviewers read the body once, then move to the diff. Reviewers do not read a longer body. No emoji, no boilerplate checklists, no "This PR…" opener. The reader knows it is a PR.

**Bug fix**

```
**Broken:** what the user saw.
**Cause:** the defect, one sentence.
**Fix:** what changed.
```

**Feature**

```
**What:** the new capability, from the user's side.
**Why:** the problem it solves.
**How:** only when the approach is non-obvious. Otherwise omit the line.
```

**Enhancement / refactor / chore**

```
**Change:** what is different.
**Why:** what it improves, with the number when one exists (before → after).
```

**Security**

Before you write a security PR, walk this check. A public PR that fixes a vulnerability also discloses it. Attackers diff PRs, and the fix reaches users only later.

1. Check the repo visibility: `gh repo view --json visibility`.
2. Private repo → use the template below.
3. Public repo → stop. Ask the user for express permission to disclose, and name the risk when you ask. Silence is not permission. An old "open PRs for me" instruction is not permission. Until permission arrives in this conversation, do not open the PR. Do not push the branch.
4. If the user refuses, offer the private paths: a GitHub private security advisory with its private fork, or the process in the project's `SECURITY.md`.

With permission, or in a private repo, fill the template:

```
**Issue:** the vulnerability class. Never include exploit steps, a PoC, or affected-version details beyond what the fix reveals.
**Fix:** what changed.
```

Apply the same restraint to the commit message: `security: validate session tokens on refresh` names the fix and reveals nothing about the attack.

Every claim must be concrete: a file, a number, an error message. "Improves performance" is banned. "Cuts cold start from 2.1 s to 0.3 s" is the bar. Why: a vague body forces the reviewer to reverse-engineer the diff, and that is the work the body exists to save.

Do not list the commits in the body. The PR page already shows them.
