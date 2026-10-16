# understand-your-changes

Two agent skills that make you prove you understand your own git changes before you ship them — in your own words, graded against the actual diff.

Install both with the [skills CLI](https://skills.sh):

```bash
npx skills add tjtoo04/understand-your-changes
```

Or one at a time:

```bash
npx skills add tjtoo04/understand-your-changes --skill clarify-changes
npx skills add tjtoo04/understand-your-changes --skill quiz-changes
```

## Skills

### clarify-changes

*Before committing or merging a branch.*

The agent summarizes what changed (what, never why), then asks you to write — in your own words — what the changes do, how they're implemented, and which architecture decisions you made and why. Answers are graded against a weighted rubric (behavior, mechanism, architecture, blast radius, risk). Vague or irrelevant answers get re-prompted; misunderstandings get corrected with file:line evidence. The loop repeats, one short question at a time from rotating angles, until you hit 90% or say "good enough."

Triggers: "clarify my changes", "explain what I changed", "walk me back through this branch", "make sure I understand what I built".

### quiz-changes

*Before committing uncommitted work.*

The agent inventories the full working tree — including what `git diff` hides: untracked files, lockfiles, config, migrations, deleted tests — builds a per-category coverage map, then quizzes you in a hybrid format: multiple-choice on facts (package names, endpoints, config vars, schema changes) with realistic distractors, plus open-ended reasoning questions ("what breaks if this is reverted?"). Wrong answers trigger a strict reteach loop. Verdict is READY TO COMMIT or NOT READY with the exact gaps.

Triggers: "quiz me on my changes", "test me on the diff", "make sure I understand what I'm committing".

## Which one to use

| Situation | Skill |
|---|---|
| Branch vs main, prose explanations, architecture rationale | `clarify-changes` |
| Uncommitted working tree, interactive quiz, commit gate | `quiz-changes` |

Neither is a code review — they check *your* understanding, not the code's quality. Use `code-review` for that.

## Requirements

- Any agent supporting the [Agent Skills](https://agentskills.io) format (Claude Code, Codex, Cursor, OpenCode, pi, and 30+ more).
- `git` CLI in the target repository.
- `quiz-changes` optionally uses structured-question and todo tools (`rpiv-ask-user-question`, `rpiv-todo` in [pi](https://pi.dev)); agents without them fall back to plain chat for the same flow.

## Repo layout

```
skills/
├── clarify-changes/
│   └── SKILL.md
└── quiz-changes/
    └── SKILL.md
```

## License

MIT
