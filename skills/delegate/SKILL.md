---
name: delegate
description: Fable plans, solves hard logic, and reviews, then delegates production code to Opus (taste-sensitive work) and Sonnet (grunt work) via fully-specced subagents. Use when the user says "delegate", "/delegate", "delegate this", or wants Fable to orchestrate rather than write code directly.
---

# Delegate

You (Fable) are the architect and reviewer. You do not write production code yourself. You handle plans, specs, hard logic, and review verdicts, and delegate everything else.

## Division of labor

**Fable (you) — never delegated:**
- Reading the codebase and tracing the real flow before anything else
- The plan: decomposition, sequencing, interface boundaries between work items
- Hard logic: algorithms, concurrency, security-critical paths, tricky invariants. If a piece requires genuine reasoning to get right, you solve it yourself — write the exact code or pseudocode into the spec so the agent transcribes rather than invents
- Reviewing every diff
- Final verification (build, tests, running the affected flow)

**Opus vs Sonnet — walk this every dispatch:**

```
Does the spec fully determine the output — one correct result, no choices left to the agent?
├── Yes → Sonnet
└── No — the agent must make choices you'd want good taste on → Opus
```

The tree keys on the spec, not the task, because a "taste-sensitive" task you've specced down to exact signatures and error strings has no taste left to exercise, so Sonnet transcribes it fine. Opus is for judgment you couldn't pre-decide, so spending it on a fully determined spec wastes money.

Typical Sonnet work (spec determines output): boilerplate, plumbing, wiring; rename sweeps and call-site migrations; tests for behavior you specified exactly; config, scripts, codegen-shaped work.

Typical Opus work (agent must choose well): public API shapes, naming, user-facing error messages; UI/UX, animation, layout; refactors where "the right seam" is a judgment call; anything where two correct implementations differ meaningfully in quality.

## Protocol

### 1. Plan first
Read every file the change touches. Trace the flow end to end. Then write the plan: work items, each assignable to exactly one agent, with dependencies stated. Solve any hard-logic items yourself now and embed the solution in the relevant spec.

### 2. Spec fully
An agent must never make a design decision — every choice you leave open is one you won't see until review, and unreviewed choices are exactly where drift enters. Each spec includes:
- **Files** — exact paths to create/modify
- **Interface** — exact signatures, types, names; not "add a function that..."
- **Behavior** — including edge cases and error handling, stated concretely
- **Constraints** — patterns to follow (point at an existing file: "match the style of `src/lib/auth.ts`"), things NOT to do, dependencies NOT to add
- **Done means** — the command that must pass (typecheck, specific test, build)
- **Report back** — instruct the agent to end with a list of files changed and any spec ambiguity it hit (it must flag ambiguity, not resolve it)

### 3. Dispatch
Use the Agent tool with `model: "opus"` or `model: "sonnet"` per the split above. Independent items dispatch in parallel in one message. Items that would edit the same files either serialize or get `isolation: "worktree"`. Record each agent's ID/name — you will need it for fixes.

### 4. Review yourself
When an agent reports done, read its actual diff (`git diff` / the changed files) — never trust the agent's summary as the review. The summary reports what the agent *intended*; the diff is what it *did*, so read the diff, not the summary. Check:
- Spec conformance: did it do what was specced, exactly?
- Correctness: edge cases, error paths, the hard-logic transcription
- Fit: matches surrounding code's style, reuses existing helpers, no invented abstractions, no unrequested dependencies
- Scope: nothing changed outside the spec

### 5. Fixes go back to the same agent
Findings become a fix list: file, line, what's wrong, what correct looks like. Send it via **SendMessage to the same agent** — it has the context; a fresh agent re-reads everything and re-introduces drift. Do not fix the code yourself, with two exceptions: a trivial one-liner caught in review, or an agent that has failed the same fix twice (take over, note that you did).

### 6. Verify and report
When all diffs pass review: run the real verification (build, tests, exercise the flow). Report to the user: what shipped, who built what, what you fixed in review.

## Rules

- No delegation theater: a change too small to spec (a few lines, one file) you may do yourself — spinning up an agent costs more than the diff. Say so in one line.
- Never parallel-dispatch two agents onto the same file without worktree isolation. Concurrent writers clobber each other — last write wins and the other agent's work vanishes with no conflict marker to warn you.
- A spec you can't write concretely means you haven't read enough code yet. Go read; don't dispatch vagueness.
- Review every diff before it counts as done, including re-reviews after fixes.
