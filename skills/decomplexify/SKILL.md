---
name: decomplexify
description: "Reduce the cognitive complexity of code in any language and make it easier to maintain. Audit functions against numeric limits (nesting depth, length, parameters, boolean operators, duplication), then refactor in a fixed order: delete, flatten, name, extract, split. Use when the user wants code simplified, refactored for readability, audited for complexity, or untangled. Triggers on: decomplexify, cognitive complexity, complexity, simplify code, refactor, readability, maintainability, deeply nested, nesting, spaghetti code, long function, god function, clean up code, code smell, hard to read, hard to maintain."
---

# Decomplexify

Cognitive complexity is the quantity of state that a reader must hold in mind to follow the code. This skill lowers that load with numeric limits and a fixed fix order. It applies to code in all languages.

## Activation

### Use For

- an audit of code for complexity or maintainability
- a refactor of nested, long, or duplicated logic
- a split of a large function or file into focused units

### Do Not Use For

- extraction of UI into components — use `componentize`
- bug fixes or behavior changes
- performance work
- format or style changes only

## Modes

| Mode | Trigger | Output |
|---|---|---|
| **Audit** | The user asks for a review, a report, or an opinion. | A list of violations. No code changes. |
| **Fix** | The user asks you to refactor, simplify, or clean the code. | The audit, then the corrected code. |

If the request is not clear, use audit mode and offer to apply the fixes. Why: an audit is easy to extend into fixes, and an unwanted rewrite is not easy to reverse.

## Limits

Measure each function against these limits. Each value above its limit is one violation.

| Property | Limit |
|---|---|
| Nesting depth | 2 levels |
| Function length | 40 lines |
| Parameters | 4 |
| Boolean operators in one condition | 2 |
| Boolean parameters that select behavior | 0 |
| Copies of one structure that differ only in data | 1 |
| File length | 400 lines |

Why these limits:

- **Nesting depth**: each level adds a condition that the reader must hold for every line below it. Two levels keep that stack small.
- **Function length**: a function above 40 lines almost always does more than one task.
- **Parameters**: five or more parameters usually travel together. A named object shows that they belong together.
- **Boolean operators**: a named predicate states the intent of a condition. A chain of operators does not.
- **Boolean flag parameters**: at the call site, a bare `true` gives the reader no information. The flag also joins two functions into one body.
- **Copies**: at the second copy, extract one parameterized function. Then one fix corrects all call sites.
- **File length**: a reader finds code by file name. A file above 400 lines covers more than one topic.

## Fix Order

Apply the transformations in this order. Why this order: each step makes the later steps smaller, and deletion is the cheapest fix.

1. Delete dead code, unused branches, and comments that repeat the code.
2. Invert nested conditions into guard clauses that return early.
3. Extract each condition with three or more boolean operators into a named predicate.
4. Extract each duplicated structure into one function with parameters for the differences.
5. Split each function that does more than one task. Name each part for its effect, not its position.
6. Split each function with a boolean flag parameter into two named functions.
7. Examine the new functions for duplicated structure. If you find a copy, go back to step 4.
8. If a file is above 400 lines after these steps, split it into files with one topic each.

## Guard Rails

These rules stop a refactor that adds complexity:

- Extract a function with one caller only when the extraction removes a nesting level or replaces a comment. Why: each extraction adds a jump for the reader, and a jump is also load.
- Do not add a wrapper, an interface, or a design pattern for a requirement that does not exist. Why: indirection is complexity with a different name.
- Do not rename or reformat code outside the violations. Why: noise in the diff hides the real changes from the reviewer.
- If a fix changes a public API, do not apply the fix. Report the fix and ask the user. Why: callers outside the project break without a warning.
- If the conventions of the project disagree with this skill, obey the project. Why: one consistent codebase reads better than a mixed one.

## Workflow

Give the user a one-line status before each step.

1. Inspect the project for its conventions, helpers, and test commands.
2. Measure the target code against the limits. Record each violation as: location, property, value, limit.
3. If the mode is audit, report each violation with one proposed fix. Stop.
4. Run the tests. If the tests fail before any change, report the failures and stop.
5. Apply the fixes in the fix order. After each transformation, make sure that the code compiles.
6. Run the tests again. If a test fails, reverse the last transformation and report it.

## Report Format

Report each violation on one line: location, property, value against limit, proposed fix.

```
src/billing.ts:120 — nesting depth 4 (limit 2) — invert the null checks into guard clauses
src/billing.ts:184 — 6 parameters (limit 4) — group the address fields into one object
```

## Verify

- Run format, lint, typecheck, and tests when the project has them.
- Make sure that the code keeps the same behavior and the same public API.
- Make sure that each new function name states what the function does.
- Measure the changed functions against the limits one more time.
