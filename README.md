# Skills

A collection of skills I use to work effectively with agents.

## How I work with agents

Agents rarely write bad code. They confidently build the wrong thing, make undocumented decisions along the way, and hand you a summary that hides what actually changed. These skills close those three gaps, in order:

1. **Understand before building** — `/prd`. Before an agent writes anything, it interviews me until the problem is concrete, then writes down goals, non-goals, and how we will know it worked. This is where wrong-thing bugs get caught, while they still cost nothing.
2. **Delegate the build, keep the judgment** — `/delegate`. One agent plans, solves the hard logic, and reviews every diff, then hands the mechanical work to cheaper subagents against a full spec. The decisions I care about stay with the model I trust; the typing goes to whoever is cheapest.
3. **See what actually changed** — `/what-changed`. When the work is done, the agent produces a before/after report with the reasoning behind each change and screenshots of anything visual, so I review the real diff instead of a flattering summary.

Each skill turns a guess into a written decision I can check: the requirement, the spec, the diff.

## Installation

Install every skill in this repo with [skills.sh](https://www.skills.sh/):

```
npx skills add arsenstorm/skills
```

## What's in the collection

| Skill | Description | Link |
|-------|-------------|------|
| commit | Writes commit messages and PRs in one strict convention: single-line conventional subjects (type(scope): message, no body or footers) and short typed PR descriptions. | [skills/commit/SKILL.md](skills/commit/SKILL.md) |
| decomplexify | Audits code in any language against numeric complexity limits (nesting depth, function length, parameters, duplication), then refactors in a fixed order: delete, flatten, name, extract, split. | [skills/decomplexify/SKILL.md](skills/decomplexify/SKILL.md) |
| delegate | Plans, solves the hard logic, and reviews every diff, then delegates production code to Opus/Sonnet subagents against a full spec. | [skills/delegate/SKILL.md](skills/delegate/SKILL.md) |
| prd | Interviews the user, then writes a Product Requirements Document: problem, goals, non-goals, testable requirements, and success criteria (with a proxy + owner for metrics the agent can't verify). | [skills/prd/SKILL.md](skills/prd/SKILL.md) |
| ship | Gets finished work live in one supervised chain: commit, push, watch CI to green, correct failures, and prove the result with the strongest evidence available. | [skills/ship/SKILL.md](skills/ship/SKILL.md) |
| what-changed | After finishing work, writes a self-contained HTML before/after report: each change with its reasoning, cropped screenshots for visual changes, from a fixed template. | [skills/what-changed/SKILL.md](skills/what-changed/SKILL.md) |

## Specification

The Agent Skills specification can be found at [agentskills.io/specification](https://agentskills.io/specification).

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

<sub>Copyright © 2026 Arsen Shkrumelyak. All rights reserved.</sub>
