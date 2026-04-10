---
name: wiki-search
description: "Search the Wiki knowledge base for a topic and return a synthesized answer. Use when the user says '/wiki-search', 'search wiki', 'what do I have on...', 'check my wiki for...', or wants to query their knowledge base."
user-invocable: true
argument-hint: "[--vault <name>] <query>"
---

# wiki-search

Searches `Wiki/` for articles relevant to a query and returns a synthesized answer drawn from the user's own knowledge base.

**Arguments** — parse `$ARGUMENTS`:

- `--vault <name>` or `--vault=<name>` — explicit vault. Pass `.` to mean "the vault at cwd" (resolved against the pre-loaded list).
- Everything else in `$ARGUMENTS`, after removing the flag and its value, is the **query** — the topic, term, or question to search for. If empty, infer from the conversation context.

**Vault resolution** (only when `--vault` is absent):

- Read the pre-loaded vault list:
  - **Exactly one vault** → use it. Do not ask.
  - **Two or more** → ask via `AskUserQuestion`.
  - **None** → bail; Obsidian isn't running.

**`--vault .` handling**: resolve the current working directory against the pre-loaded vault list. If cwd matches a registered vault path, use that vault's name. Otherwise bail — cwd isn't a vault.

## Pre-loaded Context

Registered Obsidian vaults (`name<TAB>path` per line):

!`obsidian vaults verbose 2>/dev/null`

## Workflow

### 0. Select the target vault

Parse the pre-loaded vault list (`name<TAB>path` per line). Apply the **Vault** rule from the top of this file to pick one. Hold the chosen vault's `<name>` and absolute `<path>` — every subsequent step uses them.

If `<path>/Wiki/` or `<path>/Wiki/index.md` is missing, print `ℹ️ Vault not wiki-initialized. Run /wiki-init to bootstrap.` and stop.

### 1. Match against the index

Read the chosen vault's index:

```bash
cat <path>/Wiki/index.md
```

Scan it for entries whose title or hook phrase relate to the query. Cast a wide net — partial matches, synonyms, and related concepts all count.

### 2. Read matching articles

Read the full content of every matched article. If no index entry matches, fall back to:

```bash
obsidian vault=<name> search query="<query keywords>" path="Wiki" limit=5
```

Then read the results.

### 3. Answer from the content

Synthesize a clear, direct answer using **only** what the Wiki contains. Rules:

- **Cite sources** — reference articles with wikilinks: `[[Wiki/AI/agent-harness-engineering|Agent Harness Engineering]]`
- **Quote specifics** — use concrete details, numbers, and examples from the articles, not vague paraphrases
- **Flag gaps** — if the Wiki only partially covers the query, say what's missing
- **Don't invent** — if nothing relevant exists, say so plainly: "I have nothing in the Wiki about this."

### 4. Suggest next steps (optional)

If relevant, suggest:
- A Raw clipping to compile that might fill the gap
- A related Wiki article the user might not have connected
- Creating a new article if the topic deserves one
