---
name: wiki-init
description: "Bootstrap an Obsidian vault as a wiki-enabled workspace: validate the target is a vault, create Raw/ and Wiki/ structure, and inline the Wiki contract into CLAUDE.md. Use when the user says 'wiki-init', 'init wiki', or wants to add a personal wiki to an Obsidian vault."
user-invocable: true
argument-hint: "[target-dir]"
---

# wiki-init

Bootstraps an Obsidian vault into a wiki-enabled workspace: creates the folder structure and inlines the Wiki contract into `CLAUDE.md` (creating it if absent). After running, the sibling skills (`wiki-compile`, `wiki-lint`, `wiki-save`, `wiki-search`) work with zero further setup.

The contract lives inside `CLAUDE.md`, so Claude Code picks it up as standard project context.

**Target** (`$0`): If provided, use as the target directory (absolute or relative to cwd). If not, default to the current working directory. The target must be an Obsidian vault — see step 1 for the redirect flow when it isn't.

## Pre-loaded Context

Registered Obsidian vaults (`name<TAB>path` per line):

!`obsidian vaults verbose 2>/dev/null`

## Workflow

### 1. Resolve target and validate it's a vault

- Resolve `$0` (or cwd if empty) to an absolute path.
- Parse the pre-loaded vault list. If it's empty, bail: Obsidian isn't running or the CLI isn't installed.
- Check whether the resolved path matches one of the vault paths.
  - **Match** → use it. Print the path and continue.
  - **No match** → use `AskUserQuestion` to let the user pick one of the listed vaults. The chosen vault's path becomes the target. Print it and continue.

### 2. Check existing files

Read-only check on the chosen target:

- `CLAUDE.md`, `Raw/`, `Wiki/`, `Wiki/index.md`, `Raw/log.md` — note which exist. Each step below decides what to do based on this.

### 3. Create folder structure

Idempotent. Only create what's missing. Never overwrite.

- `<target>/Raw/` — create if absent.
- `<target>/Wiki/` — create if absent.
- `<target>/Wiki/index.md` — create with `# Wiki Index\n` if absent. Leave untouched if present.
- `<target>/Raw/log.md` — create with `# Wiki Log\n` if absent. Leave untouched if present.

### 4. Inline the contract into CLAUDE.md

Source: `${CLAUDE_SKILL_DIR}/assets/contract-template.md`. Destination: a `## Wiki` section appended to `<target>/CLAUDE.md`.

Read the asset file and transform it into a properly nested section:

1. **Strip the top-level `#` title** (e.g. `# Wiki.md`). That title becomes the `## Wiki` heading.
2. **Demote every remaining markdown heading by one level.** `##` → `###`, `###` → `####`, `####` → `#####`, etc. The asset's `##` sections must nest under `## Wiki` as children.
3. **Fence-aware.** Do not rewrite `#` characters that appear inside fenced code blocks (```...``` or ~~~...~~~), inline code, or YAML frontmatter. Only transform headings at column 0 that are outside any code fence.

The final section to write looks like:

```markdown
## Wiki

<asset contents, top `#` line removed, all other headings demoted one level>
```

Then handle `CLAUDE.md` in three cases:

- **`CLAUDE.md` absent** → create it with a minimal header followed by the Wiki section:
  ```
  # CLAUDE.md

  This file provides guidance to Claude Code when working in this directory.

  <Wiki section>
  ```
- **`CLAUDE.md` present, no `## Wiki` section** → append the Wiki section (with a leading blank line) to the end of the file.
- **`CLAUDE.md` present, already has a `## Wiki` section** → do not touch it. Show a short diff summary against the shipped contract and ask the user (via `AskUserQuestion`) to choose:
  - Replace the existing `## Wiki` section with the shipped version.
  - Skip (keep what's there).
  - Write the shipped section as `<target>/CLAUDE.md.wiki.new` for manual merge.

Detect "already has a Wiki section" by grepping for `^## Wiki` at the start of a line. When replacing, replace from that line up to (but not including) the next `^## ` line, or end of file.

Never silently overwrite a user-modified Wiki section.

### 5. Report

Print a structured summary. The user should be able to read it once and know the exact final state without re-running. Suggested format:

```
wiki-init → <absolute vault path>

Created:
  - Raw/
  - Wiki/
  - Wiki/index.md
  - Raw/log.md
  - CLAUDE.md        (created/updated with inlined Wiki contract)

Already in place (left untouched):
  - <anything that existed>

Next steps:
  - Clip something into Raw/ (via Obsidian Web Clipper or manually)
  - Run /wiki-compile to synthesize into Wiki/
```

Adapt the sections based on what actually happened. If nothing was created because everything already existed, say so plainly.

## Rules

- **Only touch the target directory.** Never write outside `<target>/`, and never run git, package managers, or anything with side effects beyond file creation.
- **Idempotent.** Running `/wiki-init` twice on the same target must be safe and must not overwrite anything.
- **Never overwrite silently.** If a file exists with different content from what we'd write, ask.
- **Do not install sibling skills.** `wiki-init` does structure + contract + CLAUDE.md wiring only. Skill installation is a separate concern.
- **Do not seed content.** No sample articles, no example clippings. The user fills `Raw/` themselves.
