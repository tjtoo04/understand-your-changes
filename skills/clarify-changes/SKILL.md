---
name: clarify-changes
description: Use when a developer needs to articulate or defend their own changes in their own words before committing or merging a branch — "clarify my changes", "explain what I changed", "walk me back through this branch", "make sure I understand what I built", "justify these architecture decisions". Not a code-quality review (use code-review) and not a multiple-choice recall quiz (use quiz-changes).
---

# Clarify Changes

Establish whether the developer can **explain their own changes unaided** — what changed, how it is implemented, and which architecture decisions were made and why. The developer writes prose; you grade it against a weighted rubric and correct specific misconceptions until 90% or an explicit stop.

## When to Use
- User asks to clarify, articulate, or be walked back through their changes.
- A branch the user wrote is about to be committed or merged.
- NOT code quality/style/correctness review → `code-review`.
- NOT "quiz me" / multiple-choice recall → `quiz-changes`.

## Step 1 — Establish the changes
Default scope: everything on this branch since the default branch.

- Base ref, first that resolves: the argument the user passed (`/skill:clarify-changes HEAD~5`, `/skill:clarify-changes feat/foo`) → `git symbolic-ref --short refs/remotes/origin/HEAD` (gives `origin/main`) → `main` → `master`. If none resolve, ask which base to use.
- `git merge-base <base> HEAD`, then `git diff --stat <merge-base>...HEAD` and `git diff <merge-base>...HEAD`.
- Include uncommitted work only when the user asks: `git status --porcelain=v1`, `git diff HEAD`.
- **Read the resulting files, not just the hunks.** The diff has no surrounding context, and grading requires it.
- Scan what `git diff` hides: lockfiles/dependencies, config/env, migrations, renames and deletions, deleted or weakened tests, generated files.

## Step 2 — Bulleted summary (what, never why)
Group by area and cover the change surfaces above, not just the code diff.

State **what** changed — file, symbol, behavior. Do **not** state the rationale, the trade-offs, or the rejected alternatives: that is what is being assessed, and writing it here hands over the answer you are about to grade.

## Step 3 — Build the weighted rubric (internal)
Score each item 0-1 on correctness × coverage. Weights are a baseline; re-normalise over the categories that apply (no migrations → drop that weight and rebase the rest).

| Category | Baseline weight |
|---|---|
| Behavior — what actually changed | 0.25 |
| Mechanism — how it is implemented, the flow | 0.20 |
| Architecture — decisions made, alternatives rejected, why | 0.25 |
| Blast radius — what breaks if this is reverted | 0.20 |
| Risk — migration semantics, failure modes, rollback | 0.10 |

Show the user the **topic headings only** — never the expected answers, never a hint of them.

## Step 4 — Elicit
One plain chat message, not the question tool, so prose is not squeezed into a picker.

Keep the prompt **two sentences or fewer: no preamble, no restating the summary, no stacked follow-ups.** If you have more than one gap, ask about the weakest rubric item now and the rest in later rounds.

First round — verbatim:

> In your own words: what do these changes do, how are they implemented, and what architecture decisions did you make and why?

Use `rpiv-ask-user-question` for mode decisions only — "keep drilling / good enough / tell me the gaps".

## Step 5 — Respond to the answer

| Answer | Do |
|---|---|
| **Empty or non-responsive** ("what", "I don't care", "just tell me") | Do not score, do not advance. Acknowledge flatly in one line with no lecture, restate the ask in one sentence, say why it matters — this is about to ship — then re-prompt. |
| **Wrong or partly wrong** | Name the **specific** misconception, quote the user's own words back, show `file:line` evidence, state the correct picture, then have them re-state that one point. |
| **Right words, wrong consequences** | Treat as wrong. Ask "what breaks if you revert this?" — fluency is not understanding. |
| **Adequate** (≥90% weighted) | Close out. |

Never accept "I don't know" or "got it" as a pass. A correction counts only once the user articulates it in their own words.

## Step 6 — Loop
Re-ask **only** the items scoring under 1.0, **one short question per round** (same two-sentence cap), from a **different angle** each round — rotate between "what changed", "how does it work", and "what breaks if you revert it". Stop when the weighted score reaches 90%, or immediately and without nagging on an explicit "good enough" / "stop".

## Step 7 — Close
Compact summary: score, categories demonstrated, what stayed unverified, verdict. Report gaps plainly without moralizing. Print no secret values from env/config diffs.

Do not commit. This skill changes no files.

## Common Mistakes
- Revealing rationale in the summary, then grading the user on recall of what you just told them
- Lengthy elicitation prompts — preamble, restating the summary, or stacking several asks into one pile
- Grading from the diff instead of the current files
- Letting "I don't know" or "got it" pass
- Scoring keyword overlap rather than causal understanding
- Stopping after one round instead of looping
- Lecturing when the user is wrong; nagging after "good enough"
- Missing untracked files and lockfiles — easy to overlook, and often where the real risk sits
