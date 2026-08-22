---
name: comments
description: "Decide for each code comment whether to add it, keep it, or delete it. A comment states a why that the code cannot state: a reason, a constraint, a trade-off, a link, or a directive with its reason. Comments that narrate what the code does are deleted. Use while writing or editing code, or when the user asks to audit or clean up comments. Triggers on: comments, code comments, comment this, add comments, remove comments, too many comments, over-commented, commented-out code, TODO, FIXME, eslint-disable, biome-ignore, noqa, type: ignore."
---

# Comments

Every comment costs the reader one read and costs the author one update per edit. Code states the *what*. A comment exists only to state a *why* that the code cannot state. This skill applies to inline and block comments in all languages. Doc comments on public API (JSDoc, docstrings, rustdoc) are out of scope.

## Modes

| Mode | Trigger | Output |
|---|---|---|
| **Write** | You write or edit code. | Run the decision tree on every comment you add, keep, or touch. No separate report. |
| **Audit** | The user asks for a review or an opinion on the comments. | A list of violations: file, line, rule, the fix. No code changes. |
| **Cleanup** | The user asks to fix, clean, or reduce the comments. | The audit, then the edits applied. |

If the request is not clear, use audit mode and offer to apply the fixes. Why: an audit is easy to extend into edits, and a deleted comment the user wanted is not easy to notice.

## Decision tree

Walk this tree for every comment. Stop at the first rule that applies.

```
Is it commented-out code?
├── Yes → DELETE. Git has it.
└── No
    Is it a tool directive (lint ignore, type ignore, coverage ignore, pragma)?
    ├── Yes → KEEP. It must carry a reason on the same line or the line above.
    │         No reason → ADD one. You cannot find one → DELETE the directive and fix the code.
    └── No
        Is it a marker (TODO, FIXME, HACK, XXX)?
        ├── Yes → KEEP only if it names an owner, a ticket, a link, or a date.
        │         Bare marker → DELETE, or convert to a ticket and link it.
        └── No
            Is it a deliberate-limit note (`ponytail:`, "known ceiling")?
            ├── Yes → KEEP. It must name the limit and the upgrade path.
            └── No
                Does it restate what the next lines do?
                ├── Yes → DELETE. If the code needed the comment to be clear,
                │         rename the identifier or extract a function instead.
                └── No
                    Does it state a why: a reason, a constraint, a trade-off,
                    a non-obvious consequence, a link to a spec or a bug?
                    ├── Yes → KEEP. Make it one to three lines. Delete history
                    │         ("changed in v2"), names, and dates. Git has them.
                    └── No → DELETE.
```

## Rules

1. **Why, never what.** `// increment the counter` above `count++` is deleted. `// off by one on purpose: the API is 1-indexed` is kept. Why: the code already shows the what. A comment that repeats it goes stale on the first edit and then lies.

2. **A comment is the last resort.** Before you add a comment to explain code, try a better name, then a small extract. Add the comment only when neither works. Why: the compiler and every caller enforce a name. Nothing enforces a comment.

3. **No narration.** Never write step comments (`// loop over users`, `// handle errors`), section banners (`// ===== Helpers =====`), or change logs (`// added by X on 2024-03-01`). Why: these split a function into parts that the reader must scan, and they never carry a fact that the code does not.

4. **Directives carry a reason.** Every `eslint-disable`, `biome-ignore`, `noqa`, `@ts-ignore`, `@ts-expect-error`, `type: ignore`, `#pragma`, or `istanbul ignore` names the reason on the same line or the line above. Example: `// biome-ignore lint/performance/noBarrelFile: public package entry point`. Why: a directive without a reason tells the next reader "someone silenced this" and nothing more. They cannot tell if it is still needed.

5. **Markers have an owner.** `TODO`, `FIXME`, `HACK` keep only with an owner, a ticket, a link, or a date: `// TODO(arsen): remove after #412 ships`. Why: a bare TODO has no trigger to act on. It lives in the code forever.

6. **Deliberate limits are documented.** When you cut a real corner with a known ceiling, add a `ponytail:` comment. Name the ceiling and the upgrade path: `// ponytail: in-memory map, move to Redis when we run more than one instance`. Why: this is the one why that the next reader cannot recover from the code or from git.

7. **Commented-out code is deleted.** Delete it every time. Do not ask. Why: git keeps it, and a block of dead code makes the reader ask if it is still needed.

8. **Keep the why short.** One to three lines. If the why needs more, write it in a doc, an ADR, or a ticket, and link it. Why: readers skip long comments.

9. **A wrong comment is worse than none.** When you edit code under a comment, re-read the comment. If it is no longer true, fix it or delete it in the same change. Why: the reader trusts the comment more than the code, and a wrong comment sends them the wrong way.

## Good comments

Each of these states a fact the code cannot state:

```ts
// Intl.Segmenter is missing in Node 14, which the CLI still supports.
// Fallback: split on whitespace. Delete when Node 14 support ends.

// The order matters: the auth middleware reads req.user, set by session().

// 250ms matches the server-side debounce. Keep both in sync.

// Spec: https://www.rfc-editor.org/rfc/rfc7231#section-6.5.4

// Not a bug: the vendor API returns 200 with an error body on rate limit.

// biome-ignore lint/suspicious/noExplicitAny: the payload shape is set by a third-party webhook
```

## Bad comments

Each of these is deleted in cleanup mode:

```ts
// Get the user
const user = getUser(id);

// Check if the user is an admin
if (user.isAdmin) {

// ===== Utilities =====

// TODO: fix this

// eslint-disable-next-line

// const oldValue = compute(x);
// return oldValue * 2;

// Updated 2024-03-01 by J. to fix the null case
```

## Audit output

One line per violation, in this form:

```
src/auth.ts:42  restates code   → delete
src/auth.ts:58  bare TODO        → add owner and ticket, or delete
src/api.ts:12   directive, no reason → add the reason, or fix the code
src/api.ts:80   commented-out code → delete
```

Then the counts: kept, deleted, rewritten.

## Self-check

Before you deliver, read each comment you added or kept and answer two questions. Does the code already say this? Can a name or an extract say this instead? If the answer to either is yes, delete the comment.
