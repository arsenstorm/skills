---
name: prd
description: Interview the user, then write a Product Requirements Document — problem, goals, non-goals, specific requirements, and success criteria. Use when the user says "PRD", "/prd", "write a product requirements doc", "spec this out", or wants the problem and requirements captured before anyone builds.
---

# PRD

A PRD is only as good as the understanding behind it. You interview the user until the problem is concrete, then write the document. Never write a PRD from a one-line request. You will fill the gaps with guesses, and a PRD built on guesses points the whole build in the wrong direction while looking authoritative.

## Interview first

Do not write a line of the PRD until you can answer each of these in one concrete sentence. Ask at most two questions per message — a five-question wall gets skimmed and answered shallowly; short rounds get real answers. If an answer is vague, ask one question to sharpen it, then continue. Never fill a gap with your own assumption. An assumption written into a PRD is indistinguishable from a fact the user gave you, so it never gets challenged.

1. **Problem** — what goes wrong today, in the user's words?
2. **Who and how often** — who hits this, and how frequently? Frequency separates a papercut from a fire and decides how much the fix is worth.
3. **Solved state** — what is true after this ships that is not true now?
4. **Boundary** — what is explicitly out of scope this time?
5. **Measure** — how will anyone know it worked? This feeds the Success section, which is the hardest part to get right.

Users pitch solutions, not problems ("add a retry button"). When you hear a solution, ask "what happens to you if we don't build it?" to reach the problem underneath. Build the PRD around that problem. The proposed solution is a requirement, never the goal.

## The PRD

Write it to `prd.md` in the working directory, or a path the user names. Five sections, in this order. Nothing else — a PRD is not a design doc or an implementation plan. Adding those invites the reader to argue about solutions before the problem is agreed.

### Problem
What is broken, who feels it, and the evidence it is real (a number, a quote, a support thread). No proposed solution here.

### Goals
The outcomes solving the problem achieves, stated as outcomes, not features. "Users recover from a failed webhook without contacting support" is a goal. "Add a retry button" is a requirement. Keep them separate: a feature listed as a goal never gets challenged on whether it actually serves an outcome.

### Non-goals
What you are deliberately not doing, each with one clause of why. Non-goals carry more weight than they look. Every scope boundary you leave unwritten, the builder fills with an assumption, and you find out at review when it is expensive to change. Writing "not doing X, because Y" converts a silent assumption into a decision the user can veto now.

### Requirements
Each requirement is a statement that is true or false about the finished system. "Fast" is not a requirement; "p95 response under 200ms" is. Number them so review and the success check can reference them. If you cannot state how you would verify a requirement, it is not specific enough yet — keep writing it.

### Success criteria
How anyone knows the work succeeded. For each criterion, walk this:

```
Can you verify it yourself by running something — a test, a query, a script?
├── Yes → write the exact check: the command, the test name, or the query. That IS the criterion.
└── No — it is a human or business outcome (support tickets, retention, adoption, revenue)
     └── Write three things:
         1. The true metric, with a number and a window ("80% fewer webhook support tickets within 90 days of launch")
         2. A proxy that ships with the feature and that you can instrument ("every webhook failure logs a reason code; an error-rate panel exists")
         3. Who checks the true metric and when — a named human or system at a named checkpoint, never the agent
```

Why the split: you can run a test suite, but you cannot watch a support queue. A success metric you cannot observe gives you no feedback loop, so you declare done on vibes and the real goal quietly disappears because no one could see it. The proxy gives the build a target it can hit today. The true metric with a named owner keeps the actual goal alive after you are gone.

## Rules

- Never invent a requirement, number, or constraint the user did not give. Ask. A made-up number becomes a real target no one chose.
- Any success criterion you cannot verify yourself ships with a proxy and a named owner, always.
