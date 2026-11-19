---
name: understanding-changes
description: Use when the user wants to understand changes before reviewing, committing, or merging them — "explain my changes", "walk me through this diff", "what did this branch actually do", "help me understand this PR", "I forgot what I built". Produces a plain-language per-file explanation with the reason for each change, architecture notes, and a data-flow diagram marked to show what already existed, what was added, and what changed, then offers to run quiz-changes or clarify-changes. Not a code-quality review, and not a test of the developer's recall.
---

# Understanding Changes

Turn a diff into a mental model that the developer can hold. The skill answers three questions, in order. What the change does. Why each file is different. How data moves now.

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

Look for the reason in this order. First the commit message, then the PR body. Then a test that pins the new behaviour. Then a caller that needed it. Then a comment. Stop at the first confirmed source.

## Step 2 — Shape of the change
Before the tables, write 2–4 bullets that state the theme. Is this a feature, a fix, a refactor, a migration, or wiring? Name the one or two files that carry the core of it. Give the size: N files, +X/−Y lines.

## Step 3 — Per-file table
Write one row for each file. Order the rows by blast radius: entry points and core logic first, lockfiles and generated files last.

| File           | Status                              | What changed                 | Why (source)                                  |
| -------------- | ----------------------------------- | ---------------------------- | --------------------------------------------- |
| `path/file.py` | added / changed / deleted / renamed | One sentence in plain words. | The reason, with the evidence that proves it. |

Rules:
- If there are more than 8 files, add a bold sub-heading for each group. For example: **Core logic**, **Wiring**, **Tests**, **Generated**.
- Write one sentence in each cell. Do not recite the diff.
- If the reason is likely but not provable, write `inferred: <reason>`.
- If no reason is plausible, write `unknown — the diff does not show why`.
- Never invent a reason. A wrong reason is worse than `unknown`.

## Step 4 — Architecture table
Cover only the architecture that this change touches.

| Area | Before | After | Consequence |
| ---- | ------ | ----- | ----------- |

Watch for these areas: a new or removed layer. A dependency that changed direction. A new seam or interface. A pattern introduced. A dependency added or removed. A new contract between two modules. If the change adds no architecture, write `none`. Do not drop the table.

## Step 5 — Data-flow diagram
Draw the flow that the diff touches. Mark every node and every edge, so the reader sees the new code at a glance.

Print this legend before each diagram:

```text
LEGEND   [+] added     [~] changed     [=] already there
         --> existing flow            ==> new flow          -.-> changed flow
```

Rules:
- Draw top-down. Start at the entry point. End at the side effect: a response, a write, a message, or a file.
- Put the file or symbol in each node label. Every node must be checkable in the code.
- Keep each diagram at 15 nodes or fewer. Split a large change into one diagram for each flow.
- Show the nodes that the change touches, plus the neighbours that give them meaning. Leave out unrelated code.
- Draw a second diagram when the change crosses a boundary, for example client → service, or service → queue → worker.

Example:

```text
LEGEND   [+] added     [~] changed     [=] already there

client
  │ POST /v1/reply
  ▼
[~] api/router.py  handle_reply
  │
  ├──► [+] api/ratelimit.py  check_bucket() ──► 429 when over limit
  ▼
[=] ai_server/gateway.py  reply()
  ▼
[~] ai_server/streaming.py  flush()   was fire-and-forget, now awaited
```

Then write one short paragraph. Read the diagram from the top and state what changed in the flow, in two to four sentences.

## Step 6 — Watch list
Write three to five bullets. State what the reader must know before they ship. Examples: migration order, a flag, a rollback problem, an untested path, a missing caller update. This is a heads-up, not a review.

## Step 7 — Hand off
Ask one question as the last action: does the user want `/quiz-changes` or `/clarify-changes`?

- `/quiz-changes` — asks questions about the working tree. Use it when the work is not committed.
- `/clarify-changes` — asks the user to explain the branch in their own words. Use it when the branch is committed.

Recommend `quiz-changes` when the change set is uncommitted. Else recommend `clarify-changes`. State the reason in one short clause.

Use the structured-question tool when it exists (`rpiv-ask-user-question` in pi). Else ask in plain chat. Offer three options: quiz-changes, clarify-changes, neither. If the user picks neither, stop. Do not ask twice.

## Output contract
The reply contains these parts, in this order:

1. Scope line — the change set, the file count, +X/−Y.
2. Shape of the change — bullets.
3. Per-file table.
4. Architecture table.
5. Data-flow diagram with the legend, then the read-out paragraph.
6. Watch list.
7. The handoff question.

If a part has no content, write `none`. Never drop a part. Keep the whole reply near 120 lines. Do not repeat the diff.

## Common Mistakes
- Explaining from the diff alone — read the file around each hunk.
- Prose walls — use the tables and the bullets.
- Unmarked diagrams — the reader cannot tell the new code from the old code.
- A reason built from a plausible story — mark it `inferred`, or write `unknown`.
- Explaining every file at the same depth — order by blast radius and go deeper on the core.
- Listing the edits — state what the change does, not which lines moved.
- Skipping the handoff, or asking for it twice.
- Forgetting untracked files, lockfiles, and migrations.
