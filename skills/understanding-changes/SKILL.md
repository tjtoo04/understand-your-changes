---
name: understanding-changes
description: Use when the user wants to understand changes before reviewing, committing, or merging them — "explain my changes", "walk me through this diff", "what did this branch actually do", "help me understand this PR", "I forgot what I built". Produces a browser report with a sidebar, a plain-language behaviour summary before and after, per-file explanation, architecture notes, a feature flow, a data-flow diagram, and an impact table, each marked to show what already existed, what was added, and what changed. Then it offers to run quiz-changes or clarify-changes. Not a code-quality review, and not a test of the developer's recall.
---

# Understanding Changes

Turn a diff into a mental model that the developer can hold. The skill answers five questions, in order. What the change does. Why each file is different. Which path the user walks now, and where the change sits on that path. How the data moves. What the change reaches outside the diff.

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

For the impact table, also read the callers of each changed symbol and the consumers of each changed contract. A caller outside the diff is the evidence for a row.

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

The other fields of the report:

| Field          | Rule                                                                        |
| -------------- | --------------------------------------------------------------------------- |
| `does`         | The behaviour now, in the user's words. See "What it does" below.           |
| `how`          | The mechanism, in order. See "How it works" below.                          |
| `architecture` | Rows of `area`, `before`, `after`, `consequence`. The structure that moved. |
| `featureFlow`  | The path that the user walks. See "Feature flow" below.                     |
| `flows`        | The path that the data walks. See "Data flow" below.                        |
| `impact`       | The surfaces that the change reaches. See "Impact" below.                   |
| `watch`        | 3–5 heads-ups. Not a review.                                                |

### What it does

`does` states the behaviour. It answers what the caller or the user gets now that the change has landed.

| Key          | Rule                                                                                         |
| ------------ | -------------------------------------------------------------------------------------------- |
| `summary`    | One to three sentences. The behaviour before, then the behaviour after: "Before, X. Now, Y." |
| `behaviours` | Two to five short bullets. One behaviour in each bullet.                                     |

Rules:
- Use the words that the user of the product uses. Do not put a file name or a symbol in `summary`.
- State what now happens, what now fails, or what now costs. Do not walk the code line by line.
- Prove each bullet with the diff hunk, the file around the hunk, or a test that pins the behaviour. Do not guess.
- If the change alters no behaviour, write `refactor: no behaviour change` in `summary`, and one bullet that names what must stay the same.
- Do not repeat `shape`. `shape` names the theme and the core files. `does` names the behaviour.
- Do not repeat `architecture`. `architecture` names the structure that moved inside. `does` names what the caller sees.

### How it works

`how` states the mechanism. It answers how the code turns the input into the new behaviour, in order.

Give three to six steps as an ordered list. One step for one decision or one action.

Rules:
- Use the names of the code: the function, the class, the table, the key. `how` may name symbols, `does` may not.
- State the order. A later step depends on an earlier step, and the reader must see that.
- Name the branch where the code decides: the condition, the fallback, and the result.
- Prove each step from the file around the hunk. Do not describe a step that the diff does not reach.
- Do not repeat `does`. `does` gives the behaviour, `how` gives the route to it.
- Do not repeat `architecture`. `architecture` gives the before and after of a structure, `how` gives the order of the work inside it.
- Do not repeat `flows`. `flows` gives the values that hop between symbols, `how` explains the decision in each hop.
- If the change is data or config only, give the order of the effects.

### Feature flow
The feature flow is the path that the user walks to get value. The data flow is the path that the values take through the code. Keep the two apart. The feature flow uses the words of the user. The data flow uses the names of symbols.

Each step has a `label` and a `status`:

| Key            | Rule                                                             |
| -------------- | ---------------------------------------------------------------- |
| `label`        | One short sentence. What happens here, in the words of the user. |
| `status`       | One of `added`, `changed`, `same`, `deleted`.                    |
| `file`, `sym`  | Optional. The one place in the code that carries this step.      |
| `kind`, `edge` | The same values as the data flow below.                          |

Rules:
- Start the flow at what the product already does. Every step that the diff does not touch is `same`, and it stays in the flow. The reader needs the old path to understand the new step.
- Mark the changed steps `added` or `changed`. Mark the step that loses behaviour `deleted`.
- End the flow at the value that the user gets, or fails to get.
- If the diff changes no behaviour, draw one flow with every step in `same`, and write `refactor: no user-visible change` in the title.
- Keep each flow at 8 steps or fewer. Use a second flow for a second feature path.

### Impact
The impact table states what the change reaches outside the lines that it edits. Write one row for each surface.

| Key        | Rule                                                                 |
| ---------- | -------------------------------------------------------------------- |
| `area`     | The surface: an endpoint, a table, a config key, a caller, or a job. |
| `status`   | One of `added`, `changed`, `same`, `deleted`.                        |
| `reach`    | Who or what feels the change. Name them.                             |
| `effect`   | What happens to them now. State the failure, the cost, or the gain.  |
| `evidence` | The file and the symbol that proves the row.                         |

Rules:
- Order the rows from the widest reach to the narrowest.
- Read these surfaces before you write the table: the public API and its callers, the stored data and the migrations, the config and the environment, the permissions, the performance and the quota, the tests and the CI, and the deploy and the rollback path.
- Leave out a surface that the change does not reach.
- The impact table states what the change reaches. The watch list states what the reader must check. Do not write the same item in both.

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
Both flows use these keys. A flow is a list of steps, drawn top to bottom. Start at the entry point. End at the side effect: a response, a write, a message, or a file.

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
Keep it short. Eight parts, in this order:
1. Scope line — the change set, the file count, +X/−Y.
2. What it does — the `summary` field, one or two sentences. The behaviour before, then the behaviour after.
3. Shape of the change — the bullets from Step 2.
4. Feature flow — one line. What the user could already do, and where the change lands on that path.
5. Per-file table — the same four columns as the report.
6. Impact — one line. The surface with the widest reach, and what it now gets.
7. The absolute path of the report, on its own line.
8. The handoff question from Step 6.

The how steps, the behaviour bullets, the architecture table, the two flows, the impact table, and the watch list stay in the report. Do not print them again in chat. Do not paste the HTML or the JSON.

## Step 6 — Hand off
Ask one question as the last action: does the user want `/quiz-changes` or `/clarify-changes`?

- `/quiz-changes` — asks questions about the working tree. Use it when the work is not committed.
- `/clarify-changes` — asks the user to explain the branch in their own words. Use it when the branch is committed.

Recommend `quiz-changes` when the change set is uncommitted. Else recommend `clarify-changes`. Put the same recommendation in the `next` field of the report, so the sidebar and the chat agree.

Use the structured-question tool when it exists (`rpiv-ask-user-question` in pi). Else ask in plain chat. Offer three options: quiz-changes, clarify-changes, neither. If the user picks neither, stop. Do not ask twice.

## Common Mistakes
- `how` walks the diff line by line — give the order of the decisions, in three to six steps.
- `how` repeats the data flow — show the route to the behaviour, not the value hops.
- `does` repeats the file table or the architecture table — state behaviour, one bullet for one behaviour.
- `does` names files or symbols — speak in the words that the user uses.
- Explaining from the diff alone — read the file around each hunk.
- Rewriting the template CSS — the template is the design. Change only the JSON.
- Pasting HTML or JSON into chat — print the path.
- Unmarked nodes in the flow — the reader cannot tell the new code from the old code.
- A feature flow that starts at the new code — lead with the steps that already work, and mark them `same`.
- One flow for two jobs — the feature flow speaks in the words of the user, the data flow speaks in code symbols.
- An impact table that repeats the file list — state what those files now reach.
- The same item in the impact table and the watch list — impact states the reach, watch states the check.
- A reason built from a plausible story — mark it `inferred`, or write `unknown`.
- Explaining every file at the same depth — order by blast radius and go deeper on the core.
- Listing the edits — state what the change does, not which lines moved.
- Starting an HTTP server, or retrying the opener with another tool.
- Skipping the handoff, or asking for it twice.
- Forgetting untracked files, lockfiles, and migrations.
