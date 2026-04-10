---
name: wiki-compile
description: "Compile Raw/ clippings into structured Wiki/ articles in the Obsidian vault. Use when the user says 'compile', 'wiki-compile', 'process clippings', 'compile my raws', or wants to turn saved clippings into knowledge base articles."
user-invocable: true
argument-hint: "[--vault <name>] [topic or filename]"
---

# wiki-compile

Compiles clippings from `Raw/` into structured, interlinked articles in `Wiki/`. The goal is to turn saved web content into a curated knowledge base — grouping related sources, synthesizing key ideas, and organizing by topic.

**Arguments** — parse `$ARGUMENTS`:

- `--vault <name>` or `--vault=<name>` — explicit vault. Pass `.` to mean "the vault at cwd" (resolved against the pre-loaded list).
- Everything else in `$ARGUMENTS`, after removing the flag and its value, is the **scope** — a topic or filename used to fuzzy-match against title, tags, or content. If empty, compile all uncompiled clippings.

**Vault resolution** (only when `--vault` is absent):

- Read the pre-loaded vault list:
  - **Exactly one vault** → use it. Do not ask.
  - **Two or more** → ask via `AskUserQuestion`.
  - **None** → bail; Obsidian isn't running.

**`--vault .` handling**: resolve the current working directory against the pre-loaded vault list. If cwd matches a registered vault path, use that vault's name. Otherwise bail — cwd isn't a vault.

## Pre-loaded Context

Registered Obsidian vaults (`name<TAB>path` per line):

!`obsidian vaults verbose 2>/dev/null`

## Core Workflow

### 0. Select the target vault

Parse the pre-loaded vault list (`name<TAB>path` per line). Apply the **Vault** rule from the top of this file to pick one. Hold the chosen vault's `<name>` and absolute `<path>` — every subsequent step uses them.

Then load the working set from that vault:

```bash
obsidian vault=<name> search query='[compiled:false]' path="Raw" format=text
cat <path>/Wiki/index.md
```

The first gives the uncompiled clippings; the second is the existing index for duplicate detection.

If `<path>/Wiki/` or `<path>/Wiki/index.md` is missing, print `ℹ️ Vault not wiki-initialized. Run /wiki-init to bootstrap.` and stop.

### 1. Scan for uncompiled clippings

Use the search output from step 0 directly — no need to read each file's frontmatter individually.

If the search returns zero results, print `ℹ️ No uncompiled clippings found in Raw/. Clip something first or check that your clippings have compiled: false in frontmatter.` and stop.

### 2. Analyze and group by topic

Read each uncompiled clipping's full content. Group them by topic affinity:

- **Same topic** = merge into one article (e.g., 3 threads about LLM knowledge bases → 1 article)
- **Different topics** = separate articles (e.g., prompt engineering, health, infrastructure → 3 articles)

The grouping should be granular — prefer specific topics over broad categories. "LLM Knowledge Bases" is better than "AI". "Anti-fragile Infrastructure" is better than "DevOps". The subfolder handles the broad category; the article handles the specific concept.

Present the proposed grouping to the user using `AskUserQuestion` and **stop until they respond**. Do NOT proceed to steps 3-9 until the user confirms.

Use this format for the question:

```
Found <N> uncompiled clippings. Proposed compilation:

1. Wiki/AI/llm-knowledge-bases.md (3 sources)
   - Thread by @karpathy — LLM-powered wikis
   - Thread by @himanshustwts — Architecture diagram
   - Thread by @himanshustwts 1 — Claude Code memory

2. Wiki/AI/agent-harness-engineering.md (1 source)
   - The Anatomy of an Agent Harness

...

Does this grouping look right?
```

**IMPORTANT**: You MUST use the `AskUserQuestion` tool here — do not present the grouping as plain text output. This ensures the workflow blocks until the user explicitly confirms.

### 3. Check for existing Wiki content

Before creating a new article, check the Wiki index (pre-loaded above) for existing articles on the topic. Each index entry has a title and hook phrase — match against those.

- **If a related article exists** → update/enrich the existing article with new source material
- **If no match in the index** → create a new article

### 4. Create or update Wiki articles

#### New article structure

Read the canonical article template from `${CLAUDE_SKILL_DIR}/assets/article-template.md` and fill in the placeholders. That file is the source of truth — edit it to change the default article shape for every compile run.

#### Key rules for article content

- **Always use `[[wikilinks]]`** — every reference to a Wiki article or Raw clipping must be a wikilink, everywhere: articles, index, log, sources. No plain paths.
- **Synthesize, don't summarize** — if 3 sources talk about the same thing from different angles, weave them together. The article should read as a coherent piece, not as "Source A says X, Source B says Y."
- **Preserve specifics** — keep concrete examples, numbers, code patterns, and named tools/frameworks. The value is in the details, not vague generalizations. Before closing an article, check it against its sources: did the named people / tools / companies survive? The numbers and dates? The concrete examples (commands, snippets, workflows)? The tensions or disagreements between sources? If the article could have been written from the title alone, rewrite it.
- **Wikilink sources** — every article must have `[[Raw/<filename>]]` wikilinks in both the `sources` frontmatter property and the Sources section at the bottom.
- **Wikilink related Wiki articles** — if other Wiki articles are relevant, link them inline naturally. Don't force a "Related" section.
- **Granular type** — use specific types like `concept`, `guide`, `pattern`, `analysis`, `comparison`, `reference` rather than generic `notes`. Pick from this list when possible; add a new one only if none fit. Keep it free-form but consistent so lint and search can rely on it.

#### Enriching vs splitting

When new clippings match an existing article, the default pull is to append. Resist it when the new material is actually a different concept wearing the same category.

Split into a new article (and wikilink from the old one) when:

- You're about to open a third top-level `##` section about a sub-topic that wasn't in the original scope.
- The sub-topic has its own vocabulary, its own named tools, its own sources — it could stand alone.
- You can't summarize the resulting article in one sentence without using "and" to join two concepts.
- A reader looking only for the sub-topic would have to scroll past unrelated material to find it.

Enrich the existing article (don't split) when the new material adds depth to what's already there: more examples, sharper definitions, a counter-argument, a concrete number. Depth is good. Breadth inside one article is not.

#### Tone

Wiki articles read like reference entries, not blog posts or hype threads. Flat, factual, present tense.

Avoid:

- **Hype adjectives**: "powerful", "revolutionary", "game-changing", "seminal", "elegant", "robust". If the idea is good, the facts show it.
- **Editorial voice**: "interestingly", "notably", "it's worth noting", "crucially", "importantly". The reader decides what's notable.
- **Progressive narrative**: "would later become", "eventually led to", "goes on to". State the situation, not the arc.
- **Empty qualifiers**: "genuine", "profound", "deep", "meaningful". If they add no information, drop them.
- **Decorative em dashes** (—). Use a period, a comma, or parentheses. Reserve em dashes for real asides, not for rhythm.

Prefer short affirmative sentences. When an author's phrasing carries weight, quote it directly — the quote holds the emotion, the surrounding prose stays neutral. Define before you judge: first what it is, then why it matters.

#### Subfolder organization

Place articles in `Wiki/<Category>/`:

| Category | When to use |
|----------|-------------|
| AI | LLMs, agents, prompting, ML, AI tools |
| Engineering | Software architecture, infra, DevOps, system design |
| Business | Startups, strategy, fundraising, growth |
| Health | Wellness, fitness, nutrition, mental health |
| Finance | Personal finance, investing, markets |
| Career | Professional development, interviewing, skills |
| Books | Book notes and takeaways |
| Podcasts | Podcast episode notes |

Create new subfolders when content clearly doesn't fit existing ones. Use lowercase, descriptive names. Check what already exists in `Wiki/` before creating — don't duplicate a folder that's already there.

### 5. Create the article via obsidian CLI

```bash
obsidian vault=<name> create path="Wiki/<Category>/<article-name>.md" content="<full markdown content with frontmatter>" silent
```

Then force date types:

```bash
obsidian vault=<name> property:set name="created" value="YYYY-MM-DD" type=date path="Wiki/<Category>/<article-name>.md"
obsidian vault=<name> property:set name="updated" value="YYYY-MM-DD" type=date path="Wiki/<Category>/<article-name>.md"
```

### 6. Mark clippings as compiled

After successfully creating/updating the Wiki article, mark each source clipping:

```bash
obsidian vault=<name> property:set name="compiled" value="true" type=checkbox path="Raw/<filename>.md"
```

### 7. Update index

`Wiki/index.md` is a plain markdown file (not dataview) that serves as a navigable catalog for the LLM. Same pattern as `MEMORY.md` — one line per article, ~150 chars max, with a wikilink and a hook phrase.

Read the canonical shape from `${CLAUDE_SKILL_DIR}/assets/index-entry-template.md` — edit that file to change the default index format.

After creating/updating articles, update `Wiki/index.md`:

- If the file doesn't exist, create it with a `# Wiki Index` header
- Add one line per new article under the appropriate category heading
- If an existing article was updated (not created), update its hook phrase if the scope changed
- Keep entries grouped by category (`## AI`, `## Business`, etc.)

Example:

```markdown
# Wiki Index

## AI
- [[Wiki/AI/llm-knowledge-bases|LLM Knowledge Bases]] — persistent wiki pattern: Raw → LLM compile → interlinked markdown
- [[Wiki/AI/agent-harness-engineering|Agent Harness Engineering]] — Agent = Model + Harness; filesystems, sandboxes, memory, context rot
- [[Wiki/AI/anti-fragile-infrastructure|Anti-fragile Infrastructure]] — 7 patterns for safe agent deployments: branch previews, instant rollbacks, flags
```

### 8. Append to log

Append a timestamped entry to `Raw/log.md`. If the file doesn't exist, create it with a `# Wiki Log` header.

Each entry has the article as a wikilink with a summary, followed by indented bullets listing each Raw source as a wikilink. Read the canonical shape from `${CLAUDE_SKILL_DIR}/assets/log-entry-template.md` — edit that file to change the default log format.

Example:

```markdown
## [2026-04-04] compile | 7 clippings → 5 articles

- [[Wiki/AI/llm-knowledge-bases|LLM Knowledge Bases]] — LLM-maintained personal knowledge bases with Obsidian
  - [[Raw/Thread by @karpathy]]
  - [[Raw/Thread by @himanshustwts]]
  - [[Raw/llm-wiki]]
- [[Wiki/AI/agent-harness-engineering|Agent Harness Engineering]] — harness components and design patterns for coding agents
  - [[Raw/The Anatomy of an Agent Harness]]
- [[Wiki/AI/anti-fragile-infrastructure|Anti-fragile Infrastructure]] — making infra safe for agent-generated code
  - [[Raw/Anti-fragile Infrastructure]]
- [[Wiki/AI/claude-architect-guide|Claude Architect Guide]] — study guide for Claude Code, Agent SDK, MCP, prompt eng
  - [[Raw/I want to become a Claude architect (full course)]]
- [[Wiki/AI/claude-memory-architecture|Claude Code Memory Architecture]] (updated) — Claude Code's 3-layer memory system internals
  - [[Raw/Thread by @himanshustwts 1]]
```

### 9. Report results

After compilation, show a summary:

```
Compiled 6 clippings into 3 articles:

  Wiki/AI/llm-knowledge-bases.md (3 sources)
  Wiki/AI/agent-harness-engineering.md (1 source)
  Wiki/AI/anti-fragile-infrastructure.md (1 source)

Updated Wiki/index.md and Raw/log.md.
Marked 6 clippings as compiled.
```

## Edge Cases

- **Clipping with no clear topic** — ask the user where it should go rather than guessing
- **Clipping that spans multiple topics** — put it in the most relevant article and wikilink from others
- **Very short clipping** (< 100 words) — still compile it, but the article can be brief. Not everything needs to be a long essay
- **Clipping already compiled** (`compiled: true`) — skip it unless the user explicitly asks to recompile
- **User wants to compile specific clippings** — if the user names specific files, only process those instead of scanning all of Raw/
