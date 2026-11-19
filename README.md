# fix-my-life

An agent skill that finds the toil in a repository and proposes how to remove it.

It reads the Git history, the configuration and the CI records of a repository. It writes
a ranked report: each cause of toil with its evidence, its cost, what the project already
decided about it, and a repair. It does not change the repository that it reads.

## Install

```bash
npx skills add <owner>/<this repository> --skill fix-my-life
```

Or copy `skills/fix-my-life/` into the skills directory of your agent (for Claude Code:
`.claude/skills/` in a project, or `~/.claude/skills/`).

## Use

Ask the agent, for example:

- "Merges in this repository are painful. Where does our time go?"
- "Do a toil audit of this repository."
- `/fix-my-life`

## What is in the skill

| File | Purpose |
| --- | --- |
| `SKILL.md` | The six steps: get the history, collect the signals, turn signals into findings, check what the project decided, propose repairs, write the report. |
| `scripts/churn.py` | Prints the signals. Read-only. Python 3.9 or later and git, no other package. |
| `references/patterns.md` | Thirteen patterns of toil, each with its signal, its cause and its repairs. |
| `references/example-opendustpan.md` | The steps applied to OpenDustpan/OpenDustpan, with the errors of the first draft. |

You can run the script without an agent:

```bash
python3 skills/fix-my-life/scripts/churn.py --repo <path> --ref origin/main --since "3 months ago"
```
