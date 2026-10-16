---
name: quiz-changes
description: Use when the user has uncommitted changes (staged, unstaged, or untracked) in a git repository and wants to be quizzed or tested on them to prove they fully understand what they changed and why — new packages, API/behavior changes, migrations, config, architecture decisions. User might say "quiz me on my changes", "test me on the diff", or "make sure I understand what I'm committing". Not a code-quality review; use code-review for that.
---

# Quiz Changes

Turn the uncommitted diff into a strict, end-to-end quiz so the developer proves they understand their own changes before committing. Format: hybrid multiple-choice + open-ended. Grading: strict pass/fail with a reteach loop and a READY-TO-COMMIT / NOT-READY verdict.

## When to Use
- User has uncommitted work and wants to verify understanding before committing/PR.
- User asks "quiz me on my changes", "test me on the diff", "make sure I know what I changed".
- NOT for code quality / style / correctness review — that's `code-review` / `code-review-and-quality`.
- NOT for planning new work — that's `brainstorming` / `grilling`.

## Step 1 — Inventory the changes (evidence, not vibes)
Run from the repo root:
- `git status --porcelain=v1` — staged + unstaged + untracked
- `git diff HEAD --stat` and `git diff HEAD` — full tracked diff
- `git diff --cached --stat` — separate staged view when useful
- **Untracked files don't appear in `git diff` — read them explicitly.**

Then read the hunks **in the current files** (read the file around each changed line), never quiz from the diff alone — the diff lacks context.

Scan change surfaces git diff hides:
- **Dependencies:** package.json/pom.xml/Cargo.toml/go.mod/pyproject.toml/requirements.txt + lockfiles — new packages vs version bumps, lockfile in sync?
- **Config/CI:** .env changes (redact secret values, quiz only on key names/defaults), *.config.*, compose files, workflows
- **Schema/migrations:** migration files, SQL, model/entity changes with NO migration (red flag)
- **Renames/deletions/mode changes**
- **Tests:** what's covered, what isn't, deleted tests, happy-path-only

If there are no uncommitted changes, say so and offer to quiz the last commit instead.

## Step 2 — Coverage map (rpiv-todo)
Create one `rpiv-todo` task per category that applies, storing evidence pointers (files + line ranges) in the description. Mark `in_progress` when quizzing, `completed` only when the user answers CORRECT (or explicitly skips).

Categories:
1. **Big picture** — 1-2 sentence summary; what problem does this solve?
2. **Dependencies** — new packages (why? what do they replace?), version bumps (breaking?), removed deps
3. **API/behavior changes** — signatures, endpoints, interfaces added/changed/removed; callers updated?
4. **New files/modules** — what each does, where it plugs in (routes, DI, entrypoints)
5. **Deleted/rewritten code** — why gone, what replaces it, feature flags
6. **Config/env/CI** — new vars (defaults, required?), .env.example updated, middleware/service changes
7. **Data/schema** — column changes, destructive ops, backfills, index changes, nullable changes
8. **Architecture decisions** — patterns introduced, trade-offs, alternatives rejected
9. **Tests** — covered vs untested, dubious test changes
10. **Risk/security** — auth/authorization, input validation at trust boundaries, exposed data, secrets, error paths
11. **Hygiene** — anything that shouldn't be there: build artifacts, node_modules, .env committed, debug prints, commented-out code, unrelated edits, hand-edited generated files

## Step 3 — Quiz (hybrid)
Order: big picture → factual (MC) → reasoning (open-ended) → hygiene → verdict.

**Multiple-choice via rpiv-ask-user-question** (group 2-4 related MC questions per call; never mix MC and open-ended in one call):
- Correct answer straight from the diff + 3 realistic distractors (near-miss values, other files, inverted conditions) — not jokes or absurd options.
- Covers: package names/versions, new endpoints/signatures, which file holds what, new config vars, new tables/columns.

**Open-ended in chat (no tool):**
- "Explain in your own words: why X over Y?"
- "What breaks if <this change> is reverted?" — the best question; tests real understanding
- "Walk through the flow when <new feature> first runs."
- "What would you check first if this change caused a production incident?"

**Grading:** CORRECT / PARTIAL / WRONG per answer. Grade against the actual diff, not assumptions — if the user's claim contradicts the diff, it's WRONG, show the line. A wrong reason for a dependency or a wrong understanding of a migration's data effect = WRONG even if hand-wavy. "I don't know" = WRONG (that's what the quiz exists to catch). Never pass a category whose reason the user can't explain and the diff doesn't obviously show.

## Step 4 — Reteach loop (strict)
For every PARTIAL/WRONG:
1. Show exact evidence (file:line + correct answer in 1-3 sentences).
2. Re-ask the same topic from a different angle, matching the original format.
3. Loop until CORRECT or the user says skip. Update the todo — reteach answers don't count toward the score.

## Step 5 — Score & verdict
- Score = first-try CORRECT / total questions.
- PASS ≥ 90% first-try CORRECT **and** every applicable category has ≥1 CORRECT answer (MC alone can't pass a category whose reasoning question was skipped).
- Otherwise NOT-READY: list each category the user couldn't explain or that hides hygiene problems, with file:line.

Verdict:
- **READY TO COMMIT** — all categories passed, no unexplained changes, no hygiene issues.
- **NOT READY** — any wrong understanding, unexplained change, or hygiene problem. List what must be understood or fixed first.

## Output
Close with a compact summary: score, categories passed/failed, anything the developer couldn't explain (with file:line), verdict. Never print secret values.

## Common Mistakes
- Quizzing from the diff without reading the current files — always read hunks in context
- Forgetting untracked files — they're invisible to `git diff`
- Distractors that are obviously wrong — quiz becomes trivia
- Letting "I don't know" pass — mark WRONG, reteach
- Accepting right words without consequences — "what breaks if reverted?" exposes this
- Leaking secrets from .env diffs — redact