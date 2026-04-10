---
name: wiki-lint
description: "Audit the Wiki knowledge base for contradictions, unexplained topics, unsourced claims, and content gaps. Use when the user says '/wiki-lint', 'lint wiki', 'audit wiki', or wants a quality review of their knowledge base."
user-invocable: true
argument-hint: "[--vault <name>]"
---

# wiki-lint

Reviews the entire `Wiki/` directory and reports issues with consistency, coverage, and sourcing.

**Arguments** — parse `$ARGUMENTS`:

- `--vault <name>` or `--vault=<name>` — explicit vault. Pass `.` to mean "the vault at cwd" (resolved against the pre-loaded list).
- No other arguments.

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

Then load the working set from that vault:

```bash
cat <path>/Wiki/index.md
find <path>/Wiki -type f -name "*.md" | sort
find <path>/Raw -type f -name "*.md" | sort
```

Review the entire `<path>/Wiki/` directory and produce a lint report with these sections:

### 1. Contradictions

Flag any pair of articles that make conflicting claims about the same topic. Quote both sides and cite with wikilinks.

### 2. Unexplained topics

List concepts, terms, or names mentioned across articles but never defined in their own article. Show where they're referenced.

### 3. Unsourced claims

List factual claims (numbers, quotes, attributions, dates, studies) that aren't backed by a source in `Raw/` or a `source:` / `sources:` frontmatter field. Cite the article and the claim.

### 4. Gap suggestions

Suggest **3 new articles** that would fill clear gaps in the Wiki — based on topics hinted at, orphaned references, or adjacent areas the user clearly cares about. For each, give a one-line hook.

### 5. Overloaded articles (split candidates)

Flag articles that mix unrelated concepts under one title. Judgment calls, not thresholds:

- Section headers whose vocabulary doesn't overlap with the article title (a sub-topic snuck in).
- Frontmatter tags spanning unrelated domains (e.g. `ai` + `finance`).
- The one-sentence summary of the article needs an "and" to join two concepts that don't belong to the same system.

Long articles that enumerate parts of one system (reference docs, component catalogs) are fine — length alone is not a signal. For each real candidate, propose the split: which sections move to a new article, what its title would be, which wikilinks need updating.

### 6. Under-used sources

Complementary to unsourced claims. Unsourced claims flag article statements with no source backing; this flags the opposite — articles whose sources are substantial but whose body is thin, meaning the valuable details were left in `Raw/` and never made it into the wiki.

For each article with 2+ sources, open the sources and check if any of these are in the source but missing from the article:

- Named people, tools, or companies.
- Numbers, dates, versions, metrics.
- Concrete examples (commands, snippets, named workflows).
- Tensions or disagreements between sources that the article flattened into a single view.

If a meaningful amount is missing, flag the article and list 3-5 specific facts that should be there. Don't rewrite — just show the gap so the user can decide which are worth reopening.

### 7. AI-voice tone

Grep `Wiki/**/*.md` for markers of the tone `wiki-compile` should avoid. Report by file with counts, don't auto-correct.

- Decorative em dashes (`—`) outside of quoted material.
- Hype adjectives: `powerful`, `revolutionary`, `groundbreaking`, `seminal`, `elegant`, `robust`.
- Editorial voice: `interestingly`, `notably`, `worth noting`, `crucially`, `importantly`.
- Progressive narrative: `would go on`, `would later`, `eventually became`.

## Rules

- **Read, don't guess** — actually open the articles before flagging. Use `obsidian search` or `Read` for anything not in the pre-loaded context.
- **Cite everything** — every finding references specific articles with wikilinks: `[[Wiki/AI/agent-harness-engineering|Agent Harness Engineering]]`
- **Be specific** — quote the exact contradiction or claim, don't paraphrase vaguely
- **Don't invent problems** — if a section has no findings, say so plainly (e.g., "No contradictions found.")
- **Don't modify files** — this is a read-only audit. Suggest, don't edit.
