# Code Review Kit

Review, fix, change and diagnose existing code with IBM Bob in a repeatable, checkable way. One click installs the **code-review** skill and the rule that activates it.

## What it installs

| Asset | Type | Installs to | What it does |
|---|---|---|---|
| **code-review** | Skill | `.bob/skills/code-review/` | The method and the scripts: five tasks, a scan for 41 kinds of defect, and a gate at the end of every task |
| **rules-code-review** | Rule | `.bob/rules/rules-code-review/` | Has Bob use the skill for every review, fix or change of code, run its commands from the workspace root, and check `status` before every reply |

Keep the marketplace's **Install Location** on **project** (the default).

## After you install

1. Put the code in its own folder, with the documents that came with it (a requirement, coding standards, lists of names) saved as `.md` or `.txt`.
2. Open Bob in **Agent** mode.
3. Send a prompt in three parts:

```text
Review billing/src/billing.js. Write billing/reports/review.md. Do not change the code.
```

Bob runs the skill's review command, fills the notes file it writes, runs the command again, and replies once the command prints `Gate: passed`. The report is in `billing/reports/review.md`.

## The five tasks

| The prompt starts with | Task |
|---|---|
| `Review <code>` | Review code: a defect register |
| `Fix` | Fix findings: a corrected copy and a change log |
| `Clean up`, `Standardize`, `Rename`, `Split`, `Add` | Change existing code: a changed copy and a report |
| `Review <changed> against <baseline>` | Review a change: every difference judged |
| `Diagnose <code>: <symptom>` | Diagnose a failure: from the symptom to the line |

The full guide, with an example prompt and the result of each task, is the README of the **code-review** skill.

## Requirements

IBM Bob in Agent mode with skills enabled, and Python 3.9 or later (no other packages).
