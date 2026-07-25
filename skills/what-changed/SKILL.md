---
name: what-changed
description: After finishing a unit of work, produce a self-contained HTML before/after report — each change shown with its reasoning, and cropped screenshots for anything visual. Use when the user says "what changed", "/what-changed", "before/after report", "show me what changed", "write up the changes", or asks for a visual summary of work done.
---

# What Changed

When you finish a unit of work, produce one self-contained HTML report showing each change as before/after with the reason behind it. The diff already shows *what* changed. This report exists to show *why*, and to make visual changes reviewable without re-running anything.

## When to make one

Make a report when the work produced changes a human will review and at least one change is visual or non-obvious. Skip it for a single trivial edit: a report longer than the change wastes the reader's time. When unsure, one entry is enough. Do not pad it with entries for changes the diff already makes obvious.

## Capture the "before" first — the one irreversible step

The moment you know a change is visual, screenshot the affected area **before you edit it**. You cannot screenshot a before-state after you have already changed it. If a screenshot tool is available (browser automation, the run skill, OS capture), take the before shot as your first action on that change, not at the end.

If you already made the change and have no before shot, write "before not captured" in that slot. Never reconstruct or fake a before-state — a fabricated before makes the whole report untrustworthy, and the reader has no way to tell which images are real.

## Each entry

One entry per meaningful change, ordered by impact with the most significant first — the reader may stop after the first entry, so make it the one that matters. Every entry has three parts:

- **Title** — the change in a few words.
- **Why** — the reasoning: why this approach and not the obvious alternative. This is the entire point of the report. An entry that states only *what* changed duplicates the diff and earns nothing.
- **Before / After** — pick the representation with this tree:

```
Is the change visible on screen (UI, layout, styling, rendered output)?
├── Yes → cropped screenshot of the specific affected area, before and after.
│         Crop to the change. A full-page shot buries the one thing that moved.
└── No
    ├── Behavior or logic change → before/after code snippet, or one line of old vs new behavior
    └── Config, data, or value change → before/after values in a code block
```

## The report file

Write one standalone HTML file (default `what-changed.html` in the working directory, or a path the user names). Put screenshots in `./assets/` next to it and reference them relatively. To share it as a link instead, publish the body through the Artifact tool.

Use the template below verbatim and fill the `{{slots}}`. Determinism comes from not redesigning the shell each time — you change the content, never the structure.

## Template

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>What Changed — {{TITLE}}</title>
<style>
  :root { --bg:#fff; --fg:#1a1a1a; --muted:#666; --border:#e2e2e2; --card:#fafafa; --accent:#2563eb; }
  @media (prefers-color-scheme: dark) {
    :root { --bg:#111; --fg:#eee; --muted:#999; --border:#2a2a2a; --card:#181818; --accent:#60a5fa; }
  }
  * { box-sizing: border-box; }
  body { margin:0; background:var(--bg); color:var(--fg); font:16px/1.6 system-ui, sans-serif; }
  .wrap { max-width:960px; margin:0 auto; padding:2rem 1.25rem; }
  header h1 { margin:0 0 .25rem; font-size:1.5rem; }
  header p { margin:0; color:var(--muted); }
  .entry { border:1px solid var(--border); border-radius:10px; background:var(--card); padding:1.25rem; margin:1.5rem 0; }
  .entry h2 { margin:0 0 .5rem; font-size:1.15rem; }
  .why { margin:0 0 1rem; }
  .why strong { color:var(--accent); }
  .ba { display:grid; grid-template-columns:1fr 1fr; gap:1rem; }
  @media (max-width:640px){ .ba { grid-template-columns:1fr; } }
  .ba figure { margin:0; }
  .ba figcaption { font-size:.8rem; text-transform:uppercase; letter-spacing:.05em; color:var(--muted); margin-bottom:.4rem; }
  .ba img { width:100%; border:1px solid var(--border); border-radius:6px; display:block; }
  pre { margin:0; padding:.75rem; background:var(--bg); border:1px solid var(--border); border-radius:6px; overflow-x:auto; font:13px/1.5 ui-monospace, monospace; }
</style>
</head>
<body>
<div class="wrap">
  <header>
    <h1>{{TITLE}}</h1>
    <p>{{ONE_LINE_SUMMARY}} · {{DATE}}</p>
  </header>

  <!-- One <section class="entry"> per change. Repeat the block below. -->
  <section class="entry">
    <h2>{{CHANGE_TITLE}}</h2>
    <p class="why"><strong>Why:</strong> {{why this approach, not the alternative}}</p>
    <div class="ba">
      <figure>
        <figcaption>Before</figcaption>
        <img src="assets/{{name}}-before.png" alt="Before: {{CHANGE_TITLE}}">
        <!-- non-visual change: replace <img> with <pre>{{old code or value}}</pre> -->
      </figure>
      <figure>
        <figcaption>After</figcaption>
        <img src="assets/{{name}}-after.png" alt="After: {{CHANGE_TITLE}}">
        <!-- non-visual change: replace <img> with <pre>{{new code or value}}</pre> -->
      </figure>
    </div>
  </section>

</div>
</body>
</html>
```

## Rules

- Never fabricate a before-state. If you did not capture it, write "before not captured." A faked before makes every image in the report suspect.
- Fill the template, never redesign it. Same shell every run keeps reports comparable.
