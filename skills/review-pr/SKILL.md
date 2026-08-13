---
name: review-pr
description: "Review a pull request as the reviewer of record: draft a verdict (approve, comment, or request changes) with the fewest comments that justify it, show the draft, and post only when the user approves. Only blocking faults and real questions become comments — the skill drops style and preference notes. Use when the user wants a PR reviewed, approved, rejected, or commented on. Triggers on: review this PR, review pr, approve the pr, request changes, reject the pr, leave review comments, pr review, review #."
---

# Review PR

A review is a verdict plus the fewest comments that justify it. Ten small comments bury the one that matters. This skill drafts the verdict and its comments, shows the draft, and posts nothing until the user approves.

## Activation

### Use For

- "review this PR", "approve #12", "request changes on #12", a review verdict of any kind
- the follow-up to a triage verdict of "ask (needs a review)"

### Do Not Use For

- findings on a local diff with no PR — use `code-review`
- a sweep across many PRs and alerts — use `triage`

## Rule 1: Draft First, Post on Approval

Present the draft review in chat and stop. Post only after the user approves the draft. Post without a preview only when the user's request in this conversation says to post ("review and submit it"). Why: a posted review is public and reaches the author at once. A wrong draft costs nothing.

## Read the PR

1. `gh pr view <n>` — title, description, author, base branch, CI result.
2. `gh pr diff <n>` — the full diff.
3. For every file with a candidate comment, read the full file, not the hunk. Why: the hunk hides the context that often explains the suspect line.
4. Judge changed behavior, not changed style.

## The Comment Bar

Classify every candidate finding. The class decides its fate:

| Class | Definition | Fate |
|---|---|---|
| **Blocking** | wrong behavior, data loss, a security fault, a breaking change to a public API | inline comment |
| **Question** | a part of the diff you cannot judge without an answer | inline comment, phrased as the question |
| **Everything else** | style, naming, structure, preference, minor cleanup | drop it |

If a finding does not change the verdict and does not ask a question that the verdict needs, delete it. Why: each extra comment dilutes the blocking one, and a wall of nits reads as hostility, not care.

When one nit pattern repeats across the diff, one sentence in the review body can name the pattern. The pattern gets no inline comments.

### Before a Blocking Comment

1. Read the full file and trace the path. Why: one wrong blocking comment discredits the whole review.
2. Write the concrete failure: this input or state → this wrong outcome.
3. Anchor the comment to a file and line.

"Consider handling errors here" is banned. A blocking comment names the failure or it is not blocking.

### Security Faults on Public Repos

An inline comment that explains a vulnerability also discloses it to everyone who reads the PR. On a public repo, keep the detail in the chat draft and ask the user how to deliver it. Offer a private channel: a security advisory, or a direct message to the author.

## The Verdict

```
One or more blocking findings → request changes
No blocking findings, one or more questions → comment
Neither → approve
```

An approval body is empty or one sentence. Never pad it. Why: padding teaches authors that your approvals carry filler, and they stop reading your reviews.

If the PR author is the user, GitHub rejects an approval of their own PR. Map "approve" to a comment that says the review found nothing blocking.

## Draft Format

Show the draft in chat before any posting:

```
Verdict: request changes

Comments
1. apps/api/auth.ts:88 — blocking — refresh accepts an expired token,
   so an expired session stays alive forever
2. apps/api/auth.ts:130 — question — is the 24h window intended for
   service tokens too?

Dropped: 5 non-blocking observations (naming, import order)
```

Report the count of dropped observations. Why: the count shows that the filter ran, and the user can ask for the list.

## Posting

After approval, post the verdict and all inline comments as one review:

- approve: `gh pr review <n> --approve`
- comment: `gh pr review <n> --comment --body "..."`
- request changes: `gh api repos/{owner}/{repo}/pulls/<n>/reviews` with `event: REQUEST_CHANGES` and the comments array (`path`, `line`, `body`)

Never post the comments one by one. Why: each single comment sends its own notification, and no verdict binds them.
