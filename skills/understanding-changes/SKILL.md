---
name: understanding-changes
description: Use when the user wants to understand changes before reviewing, committing, or merging them — "explain my changes", "walk me through this diff", "what did this branch actually do", "help me understand this PR", "I forgot what I built". Produces a browser report with a sidebar, a plain-language per-file explanation, architecture notes, and a data-flow diagram marked to show what already existed, what was added, and what changed, then offers to run quiz-changes or clarify-changes. Not a code-quality review, and not a test of the developer's recall.
---

# Understanding Changes

Turn a diff into a mental model that the developer can hold. The skill answers three questions, in order. What the change does. Why each file is different. How data moves now.

The output is an HTML report that opens in the browser. Chat carries a short summary and the handoff question.

This skill **tells** the developer what the change is. `clarify-changes` and `quiz-changes` **test** the developer. Run this skill first when the change is still unfamiliar.

## When to Use
- The user asks for an explanation of their changes, a branch, or a PR.
- The user is about to review, commit, or merge work. They did not write it, or they wrote it some time ago.
- The diff is large and the user needs the shape of it before reading line by line.
- NOT for quality, style, or correctness — use `code-review`.
- NOT for testing recall — use `clarify-changes` (committed branch) or `quiz-changes` (working tree).

## Step 0 — Fix the change set
Use the uncommitted work when it exists. Else use the branch.

```bash
git status --porcelain=v1
git diff HEAD --stat
```

- If the working tree is dirty, the change set is `git diff HEAD` plus every untracked file.
- If the working tree is clean, find the base: `git merge-base origin/HEAD HEAD`, then `git diff --stat <base>...HEAD`.
- Untracked files never appear in `git diff`. Read each one.
- List the surfaces that git hides well: lockfiles, migrations, config, CI, deleted tests.
- If neither side changed, say so and stop.

## Step 1 — Read the hunks in context
Read the current file around each changed hunk. Never explain a change from the diff alone.

Look for the reason in this order. First the commit message, then the PR body. Then a test that pins the new behaviour. Then a caller that needed it. Then a comment.

## Step 2 — Shape of the change
Write 2–4 bullets that state the theme. Is this a feature, a fix, a refactor, a migration, or wiring? Name the one or two files that carry the core of it.

## Step 3 — Fill the report data
The per-file table is the spine of the report. Write one entry for each file. Order the entries by blast radius: entry points and core logic first, lockfiles and generated files last.

Each entry has four fields:

| Field    | Rule                                                                |
| -------- | ------------------------------------------------------------------- |
| `file`   | The path from the repo root. Plain text, no backticks, no markdown. |
| `status` | One of `added`, `changed`, `same`, `deleted`.                       |
| `what`   | One sentence in plain words. Do not recite the diff.                |
| `why`    | The reason, with the evidence that proves it.                       |

For `why`:
- If the reason is likely but not provable, write `inferred: <reason>`.
- If no reason is plausible, write `unknown — the diff does not show why`.
- Never invent a reason. A wrong reason is worse than `unknown`.

Do the same for `architecture` (area, before, after, consequence), `flows`, and `watch` (3–5 heads-ups, not a review).

## Step 4 — Build the HTML report
`report-template.html` sits next to this file. It holds all the CSS and the rendering script. Do not rewrite it.

1. Read `report-template.html`.
2. Replace the JSON object inside `<script type="application/json" id="report">` with your data. The sample object in the template is the schema: keep its keys and its nesting. Change no other byte of the file.
3. Write the result to the OS temp directory:
   - `${TMPDIR:-/tmp}/understand-changes-<repo>-<ref>.html`
   - Build the name from the repo folder and the branch, in lower case. Spaces become `-`. Keep `[a-z0-9._-]` only. Cut the name at 60 characters.
4. Open the file with the OS opener. Use `xdg-open <path>` on Linux, `open <path>` on macOS, and `start "" <path>` on Windows. Do not start an HTTP server.
5. If the opener fails, keep the file and print the path. Do not retry with another tool.
6. Check the page. A red banner means the JSON did not parse. Fix the object and open the file again.

### Flow steps
A flow is a list of steps, drawn top to bottom. Start at the entry point. End at the side effect: a response, a write, a message, or a file.

| Key           | Rule                                                                                |
| ------------- | ----------------------------------------------------------------------------------- |
| `kind`        | `actor` for a person or a client. `node` for code. `branch` for a conditional path. |
| `file`, `sym` | The code location. Every node must be checkable in the code.                        |
| `status`      | The same four values. Mark what the diff did to this node.                          |
| `edge`        | The label on the arrow that enters this node.                                       |

Rules:
- A `branch` step attaches to the step above it.
- Mark every node and every branch. An unmarked node hides the new code.
- Keep each flow at 8 steps or fewer. Use a second flow for a second path.
- Show the nodes that the change touches, plus the neighbours that give them meaning.

## Step 5 — Reply in chat
Keep it short. Five parts, in this order:
1. Scope line — the change set, the file count, +X/−Y.
2. Shape of the change — the bullets from Step 2.
3. Per-file table — the same four columns as the report.
4. The absolute path of the report, on its own line.
5. The handoff question from Step 6.

The architecture table, the diagram, and the watch list stay in the report. Do not print them again in chat. Do not paste the HTML or the JSON.

## Step 6 — Hand off
Ask one question as the last action: does the user want `/quiz-changes` or `/clarify-changes`?

- `/quiz-changes` — asks questions about the working tree. Use it when the work is not committed.
- `/clarify-changes` — asks the user to explain the branch in their own words. Use it when the branch is committed.

Recommend `quiz-changes` when the change set is uncommitted. Else recommend `clarify-changes`. Put the same recommendation in the `next` field of the report, so the sidebar and the chat agree.

Use the structured-question tool when it exists (`rpiv-ask-user-question` in pi). Else ask in plain chat. Offer three options: quiz-changes, clarify-changes, neither. If the user picks neither, stop. Do not ask twice.

## Common Mistakes
- Explaining from the diff alone — read the file around each hunk.
- Rewriting the template CSS — the template is the design. Change only the JSON.
- Pasting HTML or JSON into chat — print the path.
- Unmarked nodes in the flow — the reader cannot tell the new code from the old code.
- A reason built from a plausible story — mark it `inferred`, or write `unknown`.
- Explaining every file at the same depth — order by blast radius and go deeper on the core.
- Listing the edits — state what the change does, not which lines moved.
- Starting an HTTP server, or retrying the opener with another tool.
- Skipping the handoff, or asking for it twice.
- Forgetting untracked files, lockfiles, and migrations.
