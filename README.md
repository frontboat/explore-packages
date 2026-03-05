# explore-packages

A Claude Code plugin that dispatches parallel haiku subagents to generate natural-language summaries of every package in a monorepo.

## What it does

When triggered, the skill runs three passes over every package (discovered via `package.json` files):

| Pass | Output | What the agent does |
|------|--------|-------------------|
| 1 | `DESCRIPTION.md` | Reads README and source to describe purpose and appeal |
| 2 | `EXAMPLES.md` | Finds examples, tests, and demos and describes what each shows |
| 3 | `APIS.md` | Examines entry points and exports to describe the public API surface |

All output is natural language with no code blocks. One haiku agent per package per pass, all running in parallel.

After all agents finish, the results are consolidated into a `summaries/` directory with a single `summary.md` containing everything.

## Install

```
claude plugin add frontboat/explore-packages
```

Or clone and install locally:

```
git clone https://github.com/frontboat/explore-packages.git ~/.claude/plugins/explore-packages
```

## When it activates

The skill triggers automatically when you're exploring a monorepo or multi-package codebase and need an overview of what each package does.

## License

MIT
