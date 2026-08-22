---
name: arsenstorm
description: "Work the way Arsen Shkrumelyak works: orient from evidence before any work, turn a new feature into a written PRD, fix the root cause in the smallest diff, commit one change at a time, ship through a PR with a watched CI run, and keep decisions and state in a store that outlives the chat. Routes to the other skills in this repo and names the external skills and tools it depends on, with a fallback for each one that is not installed. Use when the user says arsenstorm, /arsenstorm, work like me, work like Arsen, or wants the full shipping loop applied to a task."
---

# arsenstorm

This skill is a router. It decides which skill runs at each step of a task, and it holds the few rules that apply at every step. It does not repeat what the routed skills say. When a step names a skill, load that skill and obey it.

## Dependencies

Before the first step, find out which of these are available. Record the result once per session.

| Dependency | How to detect | If missing |
|---|---|---|
| Skills in this repo (`prd`, `delegate`, `commit`, `ship`, `where-are-we`, `review-pr`, `triage`, `decomplexify`, `comments`, `what-changed`) | Listed in the available skills | Tell the user to run `npx skills add arsenstorm/skills`. Until then, obey the fallback line that each loop node carries. |
| `simple-english` | Listed in the available skills | Install from https://github.com/AminBlg/SimpleEnglish. Until then: sentences under 20 words, one instruction per sentence, no "should". |
| `no-ai-slop` | Listed in the available skills | Install from https://github.com/petergyang/no-ai-slop. Until then: no em dashes, no "not X but Y", no summary endings. |
| `componentize` | Listed in the available skills | Part of a paid subscription at https://ui.sh. Until then, use `decomplexify` for the duplication and skip the component extraction. |
| Solo MCP | A tool named `mcp__solo__whoami` exists | Use the fallback store below. |

Why these two writing skills: `simple-english` makes every sentence survive one read, and `no-ai-slop` removes the patterns that make text read as generated. Every document, commit, PR, and report that this skill produces obeys both.

Why Solo: Solo (https://soloterm.com) holds scratchpads, todos, and dev processes for a project. A scratchpad outlives the chat, so a PRD or a decision written there is still there after a compact or a new session. Solo is a desktop app with a free tier and a paid one, and most users do not have it. The fallback store must work as well for them.

### The store

Every persistent artifact goes to the store. Walk this tree for each write:

```
Is the Solo MCP available?
├── Yes
│   ├── PRD, decision, interview notes, multi-session plan → scratchpad (one per topic, H1 as title)
│   ├── Deferred work, "later", "low priority" → todo, with a tag for the project
│   └── Dev server or build command → process (start_process), never a background shell
└── No
    ├── PRD, decision, interview notes, multi-session plan → a markdown file in the repo
    │   (`prd.md`, `docs/decisions/<slug>.md`)
    ├── Deferred work → `TODO.md` in the repo root, one line per item
    │   Never commit these files unless the user says to. Add them to `.git/info/exclude`.
    │   Why: a PRD or a todo list in the repo publishes the product thinking.
    └── Dev server → a background shell, and say so in one line
```

Never keep state only in chat. Why: the user returns after a compact or a day away, and `where-are-we` rebuilds state from sources it can read. Chat is not one of them.

## The loop

Every task walks this tree from the top. Say which branch you took in one line. Each leaf names a skill and, in parentheses, the one-line fallback for when that skill is not installed.

```
Did the user ask about state? Or did the session start or context compact
while work is in progress (dirty tree, unpushed branch, or open PR)?
├── Yes → /where-are-we. Stop after the report. Wait for the user.
│         (fallback: git status, branches, open PRs, last CI run. Then report done, in flight, one next action)
└── No
    Does the request name both the fault and the fix?
    ("add a checkbox to the header", "use TanStack Link for internal links")
    ├── Yes → build (below)
    └── No
        Is it a bug, or a complaint about how something feels?
        ("it feels weird", "the skeleton does not match the rows")
        ├── Yes → build (below). Read the code and find the cause. No PRD interview.
        │         One clarifying question is allowed when the request has two readings.
        └── No
            Is it a whole new feature?
            ├── Yes → /prd. Interview first, two questions per message. Write the PRD to the store.
            │         Then → build (below).
            │         (fallback: ask for the problem, who hits it, the solved state, and the
            │          out-of-scope list. Write them to the store before any code)
            └── No
                Is it hygiene (comments, complexity, duplication, structure)?
                ├── Yes → comments → /comments. Nesting, length, duplicated logic → /decomplexify.
                │         Duplicated UI → componentize. Audit mode first, then fixes after
                │         the user approves the audit.
                │         (fallback: list the violations with file and line, and stop)
                └── No
                    Is it a PR, or the repo inbox?
                    ├── Yes → /review-pr for one PR, /triage for the inbox. Draft only.
                    │         (fallback: read the diff, draft the verdict in chat, post nothing)
                    └── No → ask one question.
```

### Build

```
Read every file the change touches. Trace the flow end to end.
Does the change touch more than three files, and does a spec exist in the store?
├── Yes → /delegate. You plan, spec, and review. Agents type.
│         (fallback: write the spec to the store, then write the code yourself)
└── No → write it yourself. Say so in one line.
Then:
  1. Run the project's checks (lint, typecheck, tests).
  2. /commit. One change per commit.
     (fallback: one line, type(scope): message, imperative, under 72 characters)
  3. When the batch is done → /ship into a PR. Watch CI to green. Ask before any merge.
     (fallback: push the branch, open the PR, run gh pr checks --watch, report the result)
  4. If the user asks what changed → /what-changed.
```

Why three files and a spec: a spec for a three-line change costs more than the change. A large change with no spec is a design decision the user never saw.

## Standing rules

These rules hold in every step. Rules 2 and 3 are adapted from https://github.com/multica-ai/andrej-karpathy-skills.

1. **Done needs evidence.** Report "done" only with the check that passed: the command, the run, the URL, or the screen the user can open. If a check did not run, say so. Why: a claim without evidence is a guess that the user plans on.

2. **Before code, name the reading and the check.** If the request has two readings, list both and ask. If a simpler approach exists, say so. Then name the check that proves the task done: a command, an existing test, or a screen and the action the user takes on it. Add a new test only when the project tests that area already, or the user asks. For more than one step, state the plan as `step → check` lines. Why: a silent choice is found at review, when it is expensive. A task with a check loops on its own until it passes.

3. **Root cause, smallest diff, every line traced.** Before you edit a function, read every caller and fix the shared path once. Add no abstraction, dependency, config value, or error handling that the task did not ask for. Do not reformat, rename, or "improve" code next to the change. Remove the imports and functions that your change made unused. Leave dead code that was already there, and mention it in one line. Why: unrelated lines hide the real change from the reviewer. A line the user did not ask for breaks something the reviewer did not look at.

4. **Batches are closed.** When the user approves items 1 to 4 and holds 5 and 6, do 1 to 4 and stop. Report, then wait. When the user gives a verdict ("yep", "okay fine", "that sucks"), act on it. Do not re-argue. Why: the user sequences work on purpose. An agent that runs ahead takes a decision that was held back.

5. **Draft before anything public.** A PR review, a comment, an issue, a merge: show the draft and wait. Why: public text reaches another person at once and cannot be taken back.

6. **Remember on request.** When the user says "remember", write it to memory at once. When the user says "add a todo" or "later", write it to the store. Why: the user expects to find it in the next session, and chat is gone by then.

## Output shape

- One status line before each step. No paragraphs of plan.
- When you need a decision, give numbered options, one line each, with your pick first.
- Code first, then at most three lines: what was skipped, and when to add it.

## Self-check before you report

1. Did you walk the loop from the top, and say which branch you took?
2. Does every persistent artifact live in the store, not only in chat?
3. Is every commit one change, in the `commit` convention?
4. Did you name the evidence for each "done"?
5. Did you stop at the end of the approved batch?
