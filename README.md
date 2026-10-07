# Code Review Kit for IBM Bob

A skill, its rule and a collection for IBM Bob, in the folder layout of the **CE Bob Marketplace**. With them, Bob reviews, fixes, changes and diagnoses existing code in a way you can repeat and check: the scripts do the mechanical work, Bob writes the judgement, and every task ends with a gate that must print `Gate: passed`.

## The assets

| Asset | Type | Folder | Installs to |
|---|---|---|---|
| **Code Review Kit** | Collection | [`Collections/code-review-kit/`](Collections/code-review-kit/) | Installs the two assets below in one click |
| **code-review** | Skill | [`SKILLS/code-review/`](SKILLS/code-review/) | `.bob/skills/code-review/` |
| **rules-code-review** | Rule | [`RULES/rules-code-review/`](RULES/rules-code-review/) | `.bob/rules/rules-code-review/` |

**Read the guide:** [SKILLS/code-review/README.md](SKILLS/code-review/README.md) covers installation, a five-minute first review, the five tasks with an example prompt for each, how to write prompts, how to prepare the workspace, how to read the results, and troubleshooting.

## Repository layout

```
.
├── SKILLS/
│   └── code-review/
│       ├── SKILL.md              the skill, loaded by Bob (name, description, author in the front matter)
│       ├── README.md             the guide, shown on the asset's page in the marketplace
│       ├── scripts/              run.py and the scripts it calls (Python standard library only)
│       └── tests/run_tests.py    712 tests
├── RULES/
│   └── rules-code-review/
│       └── review-and-change-code.md
└── Collections/
    └── code-review-kit/
        ├── collection.yaml       name, description, author, and the assets it installs
        └── README.md
```

The marketplace finds each asset by its folder: `SKILLS/<name>/` for a skill, `RULES/<name>/` for a rule, and `Collections/<name>/collection.yaml` for a collection. The card shows the first heading and paragraph of `SKILL.md` and the `author` of its front matter; the asset's page shows its `README.md`.

## Use it

**From the CE Bob Marketplace:** open the **CE Bob Marketplace** sidebar, search for `code review`, and install the **Code Review Kit** collection. Keep **Install Location** on **project**.

**Preview from this repository** before the assets are in the CE catalog: the marketplace extension can read this public repository as an additional source.

1. In IBM Bob, open **Settings** and search for `bob.marketplace.repo2`.
2. Set **Additional repository 2 — URL** to `https://github.com/Dr-Mohamed-Gamal/bob-code-review-marketplace` and give it a display name, for example `Code Review Kit`. Leave the branch empty (it defaults to `main`) and the token empty (the repository is public).
3. Run **Bob Marketplace: Refresh Catalog** from the Command Palette. The kit appears under **Collections**, **Skills** and **Rules**.

**Manual install,** from the root of your workspace:

```bash
git clone https://github.com/Dr-Mohamed-Gamal/bob-code-review-marketplace.git /tmp/code-review-kit
mkdir -p .bob/skills .bob/rules
cp -R /tmp/code-review-kit/SKILLS/code-review .bob/skills/code-review
cp -R /tmp/code-review-kit/RULES/rules-code-review .bob/rules/rules-code-review
```

## Add it to the CE Bob Marketplace

For the marketplace maintainers: copy the three asset folders into the same top-level folders of the marketplace repository.

| From this repository | To the marketplace repository |
|---|---|
| `SKILLS/code-review/` | `SKILLS/code-review/` |
| `RULES/rules-code-review/` | `RULES/rules-code-review/` |
| `Collections/code-review-kit/` | `Collections/code-review-kit/` |

No index file needs editing: the catalog lists every folder it finds. To show a vote count on the cards, open a tracking issue for each asset and map it in `marketplace.votes.json`, for example `"skill:code-review": <issue number>`.

## Requirements

IBM Bob in **Agent** mode with skills enabled, and Python 3.9 or later. The scripts use the Python standard library only.

## Author

Dr. Mohamed Gamal. Questions and feedback: open an issue in this repository.
