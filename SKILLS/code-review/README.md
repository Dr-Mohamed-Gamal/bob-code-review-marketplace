# Code Review — Agent Skill

Review, fix, change and diagnose existing code with IBM Bob in a way you can **repeat and check**. The skill's scripts scan the code and write every report and every changed file. Bob writes its judgement in a short notes file. Every task ends with a gate that must print `Gate: passed`.

| | |
|---|---|
| **Tasks** | Review code · Fix findings · Change existing code (clean up, standardize, rename, split, add) · Review a change · Diagnose a failure |
| **Works on** | Any language. The scripts read code as text, including the scripts and event attributes inside a web page |
| **Needs** | IBM Bob in **Agent** mode with skills enabled · Python 3.9 or later, no other packages |
| **Installs to** | `.bob/skills/code-review/` (the skill) and `.bob/rules/rules-code-review/` (the rule that activates it) |
| **Version** | 1.0.0 |
| **Author** | Dr. Mohamed Gamal |

**Contents:** [Why use it](#why-use-it) · [Install](#install) · [Quick start](#quick-start-your-first-review) · [How it works](#how-it-works) · [The five tasks](#the-five-tasks) · [The closed loop](#the-closed-loop) · [Writing prompts](#writing-prompts) · [Preparing the workspace](#preparing-the-workspace) · [Reading the results](#reading-the-results) · [What the scan finds](#what-the-scan-finds) · [Troubleshooting](#troubleshooting) · [What a passed gate means](#what-a-passed-gate-means) · [Reference](#reference)

---

## Why use it

Work on existing code has to be checkable: a reviewer must be able to see what was found, why, and what changed. The skill splits that work between the part that needs judgement and the part that can be scripted:

| Who | Does what |
|---|---|
| **The scripts** | Scan the code for 41 kinds of defect, write every report and every changed copy from Bob's notes, compare the new version with the original line by line, and keep the gate closed while anything is unexplained |
| **Bob** | Judges: which scan hit is a real defect, which correction is right, what to copy and under which names, which difference is intended, what only the owner of the code can decide |
| **The gate** | Prints `Gate: passed` only when every item is decided and every claim is checked against the files |

What that gives you:

- **The same notes give the same result.** Corrections, removals, renames and copies are applied by code, so a run can be repeated and audited.
- **Every finding quotes its line.** The script checks that the quoted words are on the line the finding cites.
- **The original never changes.** Every change is made in a copy you name; the original stays as the baseline.
- **Nothing changes unnoticed.** A new call, operator, string or non-ASCII character, a file left with half a block, or a requirement left unaccounted for keeps the gate closed until it is explained or corrected.
- **The owner decides what is the owner's.** Choices that the code and the documents do not settle become specific questions in the report.

---

## Install

### Option 1 · From the CE Bob Marketplace (recommended)

1. Open the **CE Bob Marketplace** sidebar in IBM Bob.
2. Search for `code review`.
3. Install the **Code Review Kit** collection. One click installs both parts:
   - the skill, to `.bob/skills/code-review/`
   - the rule, to `.bob/rules/rules-code-review/`
4. Keep the marketplace's **Install Location** setting on **project** (the default). The rule runs the scripts from `.bob/skills/code-review/` inside the workspace.

You can also install the **Code Review** skill and the **rules-code-review** rule one by one. Install both: the skill holds the method, and the rule makes Bob use it and check its progress before every reply.

### Option 2 · Manual install

From the root of your workspace:

```bash
git clone https://github.com/Dr-Mohamed-Gamal/bob-code-review-marketplace.git /tmp/code-review-kit
mkdir -p .bob/skills .bob/rules
cp -R /tmp/code-review-kit/SKILLS/code-review .bob/skills/code-review
cp -R /tmp/code-review-kit/RULES/rules-code-review .bob/rules/rules-code-review
```

### Check the install

```bash
python3 .bob/skills/code-review/scripts/run.py status
```

In a new workspace this prints `No task has been started in this workspace.` and `Gate: passed, nothing is open.`

---

## Quick start: your first review

**1. Put the code in its own folder, with the documents that came with it.**

```
billing/
├── src/billing.js
└── docs/requirement.md      what the code must do, coding standards, lists of names
```

**2. Open Bob in Agent mode** and keep one chat for the whole piece of work.

**3. Send a prompt in three parts:** what to do and where, the report to write, and what not to do.

```text
Review billing/src/billing.js. Write billing/reports/review.md. Do not change the code.
```

**4. Watch the run.** Bob says which task it is doing (*B · Review code*) and runs the review command. The first run writes a notes file and stops, as designed:

```
Review report: billing/reports/review.md (written by this command; do not edit it)
  The scan points at 4 places: 3 findings, 0 for the owner to confirm, 0 explained as not defects, 1 to decide.
Your notes: billing/reports/review.notes.md (written now: it lists the rows to decide and the form of each part)
  Decisions:              TO DO     0 of 1 made; still open: S-02
  Findings from reading:  TO DO     not written: add them, or the word "none"
  Coverage of the intent: TO DO     no rows yet: one line for each requirement of requirement.md
  Lines read:             TO DO     not stated

Gate: not passed yet. This is expected on the first run: write your part in billing/reports/review.notes.md. Then run this command again.
```

Bob then decides each open scan hit, reads the code once for what a scan cannot see, fills the notes file and runs the command again:

```
  Decisions:              done      1 of 1 made
  Findings from reading:  done      2
  Coverage of the intent: done      3 row(s)
  Lines read:             done      1-13
The report to hand over is billing/reports/review.md. Do not write or copy another report file.

Gate: passed
```

**5. Open `billing/reports/review.md`.** It holds the verdict, the findings with their quoted lines, the coverage of the requirement, what was not checked, and the questions for the owner. Bob's reply gives the verdict and the counts, and names the report.

---

## How it works

Every task follows the same four steps:

```
  your prompt
      │
      ▼
  1. First run      the script scans the code and writes the notes file:
                    the facts to start from and the form to fill        →  Gate: not passed yet
      │
      ▼
  2. Bob's notes    Bob writes its decisions, findings, corrections or plan
                    in the notes file (never in the report or the code)
      │
      ▼
  3. Next run       the script applies the notes, writes the report and any
                    changed copy, compares the copy with the original
      │
      ▼
  4. Gate           something to correct  →  Bob corrects the notes, back to 3
                    everything settled    →  Gate: passed, Bob replies
```

A task normally takes two runs of its command, three at most. The rule also has Bob run `status` before every reply, so a request that asks for several reports is finished in full before Bob answers.

---

## The five tasks

The first word of the prompt picks the task.

| The prompt starts with | Task | What you get |
|---|---|---|
| `Review <code>` | **B · Review code** | A defect register |
| `Fix` | **D · Fix findings** | A corrected copy and a change log |
| `Clean up`, `Standardize`, `Rename`, `Split`, `Add` | **E · Change existing code** | A changed copy, a report and, if asked, a map of where every line went |
| `Review <changed> against <baseline>` | **A · Review a change** | A review of every difference |
| `Diagnose <code>: <symptom>` | **C · Diagnose a failure** | The cause, from the symptom to the line |

When one request asks for more than one task, they run as separate tasks with separate reports, in the order C, D, E, A.

### B · Review code

For code with no earlier version to compare with.

```text
Review src/billing.js. Write reports/review.md. Do not change the code.
```

| | |
|---|---|
| **Bob runs** | `run.py review <code> --out <report>` |
| **Bob decides** | Each scan hit the script cannot settle: *finding*, *not a defect* (with what in the workspace shows it), or *a question for the owner*. Then Bob reads the whole file once for what a scan cannot see (wrong logic, a value set on only some paths, a check that runs before its values exist) and writes those as findings `R-01`, `R-02`… |
| **You get** | Verdict · findings with severity, quoted line, what it does, where it lands, the fix and the confidence · findings from reading · scan hits that are not defects, with the reason · coverage of the requirement, one line per requirement · not checked · questions for the owner |
| **The gate passes when** | Every scan hit is decided, every finding quotes words that are on its line, and every requirement of the intent document has a coverage line |

To have the register speak the language of the business, name the outcome in the prompt: *"…with the effect of each defect on the invoice."* The report then gets an **Effect** column.

### D · Fix findings

```text
Fix the findings of reports/review.md in a copy of src under fixed/. Write reports/change-log.md. Do not decide what is the owner's to decide.
```

| | |
|---|---|
| **Bob runs** | `run.py fix <code> --out <folder> --log <change log> --register <review report>` |
| **The script** | Makes every correction that has only one possible form, in the copy |
| **Bob decides** | For each other High finding: correct it (the new line, with a worked example: one input, what the line gave before, what it gives now), or leave it to the owner with a specific question |
| **You get** | The corrected copy (the original is untouched) · the change log · proposals and questions for the owner |
| **The gate passes when** | Every High finding is corrected or has a question for the owner, no correction adds a new scan hit, and every new function, operator or syntax is explained |

A correction that only the owner can confirm is listed as a **proposal** and not applied.

### E · Change existing code

The code works today; the task improves it without changing what it does. Bob writes a plan in the notes file and the script makes the change, so no line is re-typed by hand.

```text
Clean up a copy of src under cleaned/: remove the commented-out code as section 3 of the requirement asks. Write reports/clean-up.md. Do not change what the code outputs.
```

```text
Standardize a copy of src under standardized/: rename the variables to camelCase as section 3 of the requirement asks. Write reports/standards.md. Do not rename anything another component reads.
```

```text
Add the item NEW, modelled on the item OLD, to a copy of src/page.html under changed/. Write reports/add-item.md. Do not change src.
```

| | |
|---|---|
| **Bob runs** | `run.py edit <code> --out <folder> --report <report> --map <map file>` (`--map` for a traceability map or a split) |
| **The script** | Lists the facts to start from: repeated calls, calls inside loops, logging calls, commented-out code, unused names, duplicate branches, and names that are not camelCase with a proposed form for each |
| **Bob plans** | A **rule** (one pattern for every line of a kind) · a **correction** (new lines for a line or a range) · **renames** (`old -> new`) · a **copy** (a new item made exactly like an existing one, wherever the model appears) · **files** (for a split: which line ranges go to which file) |
| **You get** | The changed copy · the report, with one coverage line per requirement (done, in part, or not done with the reason) · the map, if asked |
| **The gate passes when** | Every difference between the two versions is corrected or explained against the requirement · a split keeps whole blocks, the order in which lines run, and every file · a copy holds no name of its model, in any letter case, and no name the code already has |

### A · Review a change

For a change whose earlier version exists, such as the output of a fix or a clean-up.

```text
Review changed/ against src/page.html. Write reports/code-review.md. Do not fix anything.
```

| | |
|---|---|
| **Bob runs** | `run.py change <baseline> <changed> --out <review>` |
| **Bob decides** | Each difference the comparison lists: *no behaviour change*, *intended* (with the requirement it serves) or *unintended*. Then Bob reads each changed file in full against the baseline and checks every claim of the earlier reports |
| **You get** | Both versions in numbers · scan counts before and after · every difference, judged · findings · each claim marked *holds* or *does not hold*, with evidence · coverage of the intent · questions for the owner |
| **The gate passes when** | Every difference is judged, every check is answered, and every claim is checked |

### C · Diagnose a failure

```text
Diagnose src/page.html: the NEW tiles open the alerts of OLD. Write reports/diagnosis.md. Do not fix anything.
```

| | |
|---|---|
| **Bob runs** | `run.py diagnose <code> --out <report>` |
| **Bob writes** | The symptom in the reporter's words, the names and values it involves, and the evidence that exists besides the code (error text, logs, the input that triggers it) |
| **The script** | Lists every line that names what the symptom involves, with the scan hits on those lines |
| **You get** | The causes, most likely first, each with its line, how it produces the symptom, and its status: *shown*, *ruled out*, or *open* with the test that decides it · a proposed fix for the owner. The code is not changed |
| **The gate passes when** | One cause is shown, or every cause that is not ruled out names the test that decides it, and every quote is on its line |

---

## The closed loop

A change runs best as a loop, with one prompt per step, all in the same chat:

```
  change  →  review the change  →  fix  →  review the fix  →  …until the review is clean
```

When the review is clean, the fix step passes at once without changing anything.

A full worked example, five prompts on production code (defect register → fix → clean-up → standardize → review against the original), is in the Netcool pilot: [use case 5](https://github.com/Dr-Mohamed-Gamal/bob-netcool-code-review/tree/main/use-case-5-code-review), with each prompt and what it produces.

---

## Writing prompts

Every prompt has three parts:

| Part | Example | Why |
|---|---|---|
| 1 · The verb, what and where | `Review billing/src/billing.js` | The verb picks the task |
| 2 · `Write <report>.` | `Write billing/reports/review.md.` | The command writes the report under exactly that name |
| 3 · `Do not ...` | `Do not change the code.` | States the limit of the step; the comparison checks it |

Tips that keep runs short and results precise:

- **Name the file, not its folder,** unless the code is spread over the files of the folder. Other files next to it (lookup tables, includes, data) are read as context.
- **One concern per prompt.** Fix, clean up and standardize are separate steps, each working on the copy the step before wrote.
- **Point at the document.** *"…as section 3 of the requirement asks"* ties the change to the owner's own words, and the report covers that item.
- **Be exact about scope.** Give the exact text to replace, and say what must stay, for example *"or any folder or file name after the path prefix"*.
- **Several bodies of code:** write *"one after the other"* and give each path with its folder.
- **Stay in one chat, in Agent mode,** for the whole loop. Bob uses `status` to see what is finished.

---

## Preparing the workspace

Put each body of code in its own folder, with the documents that came with it. The scripts find those documents by themselves and use them:

| Document | Used for |
|---|---|
| A requirement or brief (the intent) | One coverage line per requirement in every report |
| Coding standards | The items a clean-up or a standardization must meet |
| Lists of names, with their types | The *"not in the list of names"* check of the scan |

```
workspace/
├── .bob/                         the skill and the rule
├── impact-policy/
│   ├── inputs/policy.ipl
│   └── docs/requirement.md
└── probe-rules/
    ├── inputs/device.rules
    └── docs/code-standards.md
```

Good to know:

- **Documents must be text:** `.md`, `.txt`, `.rst`, `.csv`, `.tsv` or `.adoc`, under 2 MB. Save a Word or PDF document as Markdown or plain text first.
- Files named `README…`, files in folders that start with a dot, the reports the scripts wrote, and notes files are not read as documents.
- When the workspace holds several bodies of code, each one uses the documents of its own folder only; the command prints the ones it left out.

---

## Reading the results

| File | Written by | What it holds |
|---|---|---|
| The report you named (`reports/review.md`) | The script | The result of the task. Written again on every run: do not edit it |
| The notes file (`reports/review.notes.md`) | Bob | Bob's decisions, findings, corrections or plan. This is where any change of judgement goes |
| The copy (`fixed/`, `cleaned/`, `changed/`) | The script | The changed code. The original stays as it was |
| The change log or map | The script | Every correction with its reason, or where every line went |
| `.code-review-tasks.json` | The script | The task list that `status` reads |

**IDs.** `S-` scan rows · `C-` the script's corrections · `E-` changes of a plan · `R-` Bob's findings from reading · `H-` Bob's corrections · `V-` findings of a review of a change.

**Severity.**

| | |
|---|---|
| **High** | Wrong output, lost data, a rule that no longer applies where it should, a security exposure, a file that cannot be loaded on its own, a step every case used to reach that only some cases reach now |
| **Medium** | May fail when run, or leaves a requirement partly done |
| **Low** | Unused code, an unsupported statement in a report, naming, tidiness |

**Check progress at any time:**

```bash
python3 .bob/skills/code-review/scripts/run.py status
```

```
Tasks started in this workspace, in the order they were started:
  NOT PASSED  review   billing/reports/review.md
...
Not finished. Do this one next: run it again and settle what it lists.
  python3 .bob/skills/code-review/scripts/run.py review billing/src/billing.js --out billing/reports/review.md
```

---

## What the scan finds

The scan looks for 41 kinds of defect, in any language. It finds the same things every time, and for each hit it also says where the line lands: what it sets, where that value goes, and which blocks it is inside.

| Group | Kinds |
|---|---|
| **Names** | Read and never assigned · assigned and never read · misspelt · not in the list of names a document gives · written in two letter cases · read before they are set · overwritten at once · replaced before anything reads them |
| **Conditions** | A single `=` where a comparison is meant · a `;` straight after the condition · the same condition twice in one chain · comparisons that contradict each other · `&&` and `\|\|` mixed without brackets · a value compared with itself · a comparison whose result is not used |
| **Structure** | Brackets that do not balance · an `else` with no `if` · an empty block · branches with the same statements · the same label twice in a switch · a function defined twice or never called · code after a `return` · a loop that nothing ends · a loop bound that includes the size · a result indexed before its size is checked · an error handler that only logs, or whose message names another error · two `;` in a row |
| **Text in strings** | An entity or a tag not closed · a string not closed · placeholder text · an environment name or a credential · a query or a command joined with a value |
| **Data** | A value given to the code that the code changes · a replacement limited to a count · a replacement made twice on the same value |
| **Lines as written** | A log that says the code does something (discard, drop, skip…) while no statement after it does it · in a page with rows of one kind, a row that names another item, such as a tile whose filter selects another device |

**Beyond the scan.** A review always has a second pass: Bob reads the whole file once for what only reading shows, such as a branch that can never run, a field built from a variable that nothing sets, or a value set and then always overwritten. Those findings go under *Findings from reading*, and the gate checks that each one quotes its line.

---

## Troubleshooting

| You see | What it means | What to do |
|---|---|---|
| `Gate: not passed yet` after the first run | The notes file was written; this is by design | Nothing. Bob fills the notes and runs the command again |
| Exit code `3` | Not passed yet: a result, not an error | Bob settles what the command lists and runs it again |
| The command lists items to correct | The gate found something unsettled in the notes | Bob corrects the notes and runs again, in the same turn |
| A command ends without a `Gate:` line, or says a script failed | A fault in a script | Bob names the command and its output, does that step by hand, and says so in its reply. Please open an issue with the output |
| Bob replies before every report of the request exists | The request asked for more than one report | Send: `Run status and finish what is open.` |
| The skill is not used | Bob is not in Agent mode, skills are off, or a file is missing | Check Agent mode, that skills are enabled, and that `.bob/skills/code-review/SKILL.md` and `.bob/rules/rules-code-review/review-and-change-code.md` exist. Start the prompt with the task verb |
| You installed with **Install Location: global** | The rule runs the scripts from `.bob/skills/code-review/` in the workspace | Install again with the location set to **project** |
| Another skill named `code-review` is in `~/.bob/skills` | Bob uses the project's skill first | Keep this skill installed in the project's `.bob/skills/` |
| A change typed into a report disappears | Reports are written again on every run | Put the change in the notes file and run the command again |

---

## What a passed gate means

- `Gate: passed` means the work of the task is **complete and accounted for**: every scan hit is decided, every quoted line is on its line, every difference is explained, every requirement has a coverage line.
- It does **not** mean the code is right where it runs. The scripts read the code as text and do not run it, so every report ends with **Not checked**, and what a finding does on the target platform is marked *needs a test*.
- Findings from reading are Bob's judgement, as a human reviewer's would be. A person should read them before they go to the owner.
- Defects outside the code, such as a value in a lookup table or a difference from the live system, are out of reach of any code review. The report lists them as questions for the owner.
- After a review or a diagnosis, nothing is changed until you say which findings to fix.

---

## Reference

### Commands

Bob runs these from the workspace root; you rarely need to type them yourself.

| Command | What it does |
|---|---|
| `run.py review <code> --out <report>` | Review code (task B) |
| `run.py fix <code> --out <folder> --log <change log> --register <review report>` | Fix findings in a copy (task D) |
| `run.py edit <code> --out <folder> --report <report> --map <map file>` | Change existing code in a copy (task E) |
| `run.py change <before> <after> --out <review>` | Review a change (task A) |
| `run.py diagnose <code> --out <report>` | Diagnose a failure (task C) |
| `run.py look <code>` | Count what is in the code: repeated calls, calls inside loops, logging, unused names, commented-out code, line endings |
| `run.py check <report> <code> --intent <requirement>` | Check a review report on its own |
| `run.py status` | List every task started in the workspace, whether its gate passed, and the command to finish each open one |

The full path is `python3 .bob/skills/code-review/scripts/run.py`.

### Files in this skill

```
code-review/
├── SKILL.md                 the skill: tasks, rules and gates, loaded by Bob
├── README.md                this guide
├── scripts/                 the work; run.py is the single entry point
│   ├── run.py               review · fix · edit · change · diagnose · status · look · check
│   ├── scan_code.py         the scan: 41 kinds of defect, any language
│   ├── write_register.py    review report from the scan and Bob's notes
│   ├── fix_code.py          corrected copy and change log
│   ├── edit_code.py         changes: corrections, rules, renames, splits, copies
│   ├── write_review.py      review of a change, line by line
│   ├── compare_code.py      the gate's comparison of two versions
│   ├── diagnose.py          from a symptom to the line
│   └── ...                  helpers: notes, inventory, trace, report rows, strings
└── tests/
    └── run_tests.py         712 tests on small synthetic samples
```

### Requirements

- IBM Bob in **Agent** mode, with skills enabled.
- Python 3.9 or later. Standard library only: nothing to install.

### Tests

```bash
python3 .bob/skills/code-review/tests/run_tests.py
```

Expected last line: `712 passed, 0 failed, 0 known fault(s)`.

### Version history

| Version | Date | Changes |
|---|---|---|
| 1.0.0 | 2026-10-04 | First marketplace release: five tasks, 41 kinds of defect, `status`, 712 tests |

### Guiding principle

**Bob brings the judgement; the scripts make every result repeatable and auditable.** Script what can be scripted, leave to Bob what needs reading and deciding, and let nothing count as done until a gate has checked it.
