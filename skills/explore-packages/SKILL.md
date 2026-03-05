---
name: explore-packages
description: Use when exploring a monorepo or multi-package codebase to generate natural-language summaries of each package's purpose, examples, and public APIs using parallel haiku subagents
---

# Explore Packages

## Overview

Dispatch one haiku subagent per package in parallel to produce concise, no-code, natural-language documentation. Run three passes (description, examples, APIs), then consolidate into a unified output.

## When to Use

- Onboarding to an unfamiliar monorepo
- Generating a high-level overview of a multi-package project
- Auditing what examples and public APIs each package offers
- Creating a single reference document for a codebase

## Execution

### Step 1 - Discover packages

Use Glob to find every `package.json` in the repo (excluding `node_modules`). Each directory containing a `package.json` is a package. Every single one is a target — do not skip, filter, or judge which packages are "meaningful." You explore all of them.

### Step 2 - Pass 1: Descriptions

For EACH package, launch a background haiku Agent with this prompt (substitute the path):

> Explore /absolute/path/to/packages/PKG_NAME/. Read the README.md and browse key source files to understand what this package does. Then write a brief, 1-page natural language description (no code examples) of:
> 1. What the package's intended purpose is
> 2. Why it is cool / what makes it interesting
>
> Write the description to /absolute/path/to/packages/PKG_NAME/DESCRIPTION.md. Keep it concise, engaging, and accessible. No code blocks.

Launch ALL agents simultaneously using `run_in_background: true`, then wait for all to complete.

### Step 3 - Pass 2: Examples

For EACH package, launch a background haiku Agent with this prompt:

> Explore /absolute/path/to/packages/PKG_NAME/ for any examples included in the package. Look in directories like examples/, test/, docs/, or any other location that contains example usage. Check the README.md for example references too. Search for directories named "example*", "demo*", "sample*", and also look at test files for usage patterns.
>
> Write a brief, no-code, natural language description of what examples exist, what each one demonstrates, and how they showcase the package's capabilities. Write it to /absolute/path/to/packages/PKG_NAME/EXAMPLES.md. No code blocks. Keep it concise and accessible.

Launch ALL agents simultaneously, then wait for all to complete.

### Step 4 - Pass 3: APIs

For EACH package, launch a background haiku Agent with this prompt:

> Explore /absolute/path/to/packages/PKG_NAME/ to find the most useful external/public APIs this package exports. Look at the main entry point (index.ts or similar), package.json "exports" field, and any barrel files. Identify the key exported functions, classes, types, and constants that consumers of this package would use.
>
> Write a brief, no-code, natural language description of the most useful exported APIs. For each one, mention its export name, what it does, and why it's useful. Do NOT include any code blocks or code examples. Only reference APIs by their export name.
>
> Write the result to /absolute/path/to/packages/PKG_NAME/APIS.md. Keep it concise and organized.

Launch ALL agents simultaneously, then wait for all to complete.

### Step 5 - Consolidation

After all three passes complete, run these bash commands:

1. Create the unified directory structure:

```bash
mkdir -p /path/to/repo/summaries/{pkg1,pkg2,...}
```

2. Copy all files into it:

```bash
for pkg in pkg1 pkg2 ...; do
  cp "/path/to/repo/packages/$pkg/DESCRIPTION.md" "/path/to/repo/summaries/$pkg/"
  cp "/path/to/repo/packages/$pkg/EXAMPLES.md" "/path/to/repo/summaries/$pkg/"
  cp "/path/to/repo/packages/$pkg/APIS.md" "/path/to/repo/summaries/$pkg/"
done
```

3. Concatenate into a single file:

```bash
output="/path/to/repo/summaries/summary.md"
echo "# Repo Name - Package Summaries" > "$output"
echo "" >> "$output"
for pkg in pkg1 pkg2 ...; do
  echo "---" >> "$output"
  echo "" >> "$output"
  echo "# Package: $pkg" >> "$output"
  echo "" >> "$output"
  for file in DESCRIPTION.md EXAMPLES.md APIS.md; do
    cat "/path/to/repo/summaries/$pkg/$file" >> "$output"
    echo "" >> "$output"
    echo "" >> "$output"
  done
done
```

4. Clean up the agent-written files from the package directories:

```bash
for pkg in pkg1 pkg2 ...; do
  rm "/path/to/repo/packages/$pkg/DESCRIPTION.md"
  rm "/path/to/repo/packages/$pkg/EXAMPLES.md"
  rm "/path/to/repo/packages/$pkg/APIS.md"
done
```

The summaries/ folder is the single source of truth. The files in the package directories were just an intermediate step.

## Agent Configuration

- **subagent_type:** general-purpose
- **model:** haiku
- **mode:** bypassPermissions (agents MUST be able to write files without prompts)
- **run_in_background:** true for all agents in a pass, then collect with TaskOutput
- **Output constraint:** no code blocks in agent output, only natural language

## Critical Rules

- **Agents write their own files.** Each agent uses the Write tool to create its output file directly. You do NOT write the files yourself from agent results.
- **Consolidation reads from disk.** Step 5 copies the files the agents already wrote. If an agent failed to write its file, re-dispatch that agent — do not write it yourself from memory or summaries.
- **Do not summarize agent output.** The consolidation step concatenates the raw files. You never rewrite, condense, or editorialize the agent output.
- **All passes can run in a single batch** if you prefer. There is no dependency between passes — you can launch all 3N agents at once (N per pass) instead of waiting for each pass to finish.

## Quick Reference

| Pass | Output File | Agent Task |
|------|-------------|------------|
| 1 | DESCRIPTION.md | Read README + source, describe purpose and appeal |
| 2 | EXAMPLES.md | Find examples/tests/demos, describe what each shows |
| 3 | APIS.md | Read entry points + exports, describe public API surface |

## Common Mistakes

- Running passes sequentially instead of launching all agents in parallel within each pass
- Using a heavier model than haiku for simple exploration tasks
- Including code blocks in output instead of natural-language descriptions
- Forgetting to wait for all agents to complete before starting consolidation
- Moving original files instead of copying them during consolidation
