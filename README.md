# understand-your-changes

Three agent skills that make you prove you understand your own git changes before you ship them — in your own words, graded against the actual diff.

Install all three with the [skills CLI](https://skills.sh):

```bash
npx skills add tjtoo04/understand-your-changes
```

Or one at a time:

```bash
npx skills add tjtoo04/understand-your-changes --skill understanding-changes
npx skills add tjtoo04/understand-your-changes --skill clarify-changes
npx skills add tjtoo04/understand-your-changes --skill quiz-changes
```

## Skills

### understanding-changes

*Before everything else — while the change is still unfamiliar.*

The agent fixes the change set (uncommitted work first, then the branch), reads each file in context, and writes a browser report: what the change does, why each file is different, which architecture it touches, a feature flow that keeps the steps that already work beside the new ones, a data-flow diagram, and an impact table of the surfaces the change reaches. Every node and row is marked `+ added`, `~ changed`, `= already there` or `− deleted`. A sidebar carries the change set, jump links, the status key and the recommended next step. Chat gets the short version — scope, shape, the feature-flow line, the per-file table, the impact line, and the path to the report.

Triggers: "explain my changes", "walk me through this diff", "what did this branch actually do".

### clarify-changes

*Before committing or merging a branch.*

The agent summarizes what changed (what, never why), then asks you to write — in your own words — what the changes do, how they're implemented, and which architecture decisions you made and why. Answers are graded against a weighted rubric (behavior, mechanism, architecture, blast radius, risk). Vague or irrelevant answers get re-prompted; misunderstandings get corrected with file:line evidence. The loop repeats, one short question at a time from rotating angles, until you hit 90% or say "good enough."

Triggers: "clarify my changes", "explain what I changed", "walk me back through this branch", "make sure I understand what I built".

### quiz-changes

*Before committing uncommitted work.*

The agent inventories the full working tree — including what `git diff` hides: untracked files, lockfiles, config, migrations, deleted tests — builds a per-category coverage map, then quizzes you in a hybrid format: multiple-choice on facts (package names, endpoints, config vars, schema changes) with realistic distractors, plus open-ended reasoning questions ("what breaks if this is reverted?"). Wrong answers trigger a strict reteach loop. Verdict is READY TO COMMIT or NOT READY with the exact gaps.

Triggers: "quiz me on my changes", "test me on the diff", "make sure I understand what I'm committing".

## Which one to use

| Situation                                                     | Skill                   |
| ------------------------------------------------------------- | ----------------------- |
| The change is unfamiliar — first read, plain words, feature flow, data flow | `understanding-changes` |
| Branch vs main, prose explanations, architecture rationale    | `clarify-changes`       |
| Uncommitted working tree, interactive quiz, commit gate       | `quiz-changes`          |

None of them is a code review — they check *your* understanding, not the code's quality. Use `code-review` for that.

## Requirements

- Any agent supporting the [Agent Skills](https://agentskills.io) format (Claude Code, Codex, Cursor, OpenCode, pi, and 30+ more).
- `git` CLI in the target repository.
- `understanding-changes` writes a self-contained HTML file to the OS temp directory and opens it with `xdg-open` / `open` / `start`. No server and no network access.
- `quiz-changes` optionally uses structured-question and todo tools (`rpiv-ask-user-question`, `rpiv-todo` in [pi](https://pi.dev)); agents without them fall back to plain chat for the same flow.

## Repo layout

```
skills/
├── clarify-changes/
│   └── SKILL.md
├── quiz-changes/
│   └── SKILL.md
└── understanding-changes/
    ├── SKILL.md
    └── report-template.html
```

## License

MIT
