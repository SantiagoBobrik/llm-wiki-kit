---
name: wiki-save
description: "Save a conversation answer or insight as a Wiki article. Use when the user says 'save this to wiki', 'wiki-save', or wants to file a good answer into the knowledge base."
user-invocable: true
argument-hint: "[--vault <name>] [topic or title]"
---

# wiki-save

Saves conversation content as a structured Wiki article. This is the "query → file back" operation — when a conversation produces a good answer, analysis, or synthesis, this skill files it into `Wiki/` so it compounds in the knowledge base instead of disappearing into chat history.

**Arguments** — parse `$ARGUMENTS`:

- `--vault <name>` or `--vault=<name>` — explicit vault. Pass `.` to mean "the vault at cwd" (resolved against the pre-loaded list).
- Everything else in `$ARGUMENTS`, after removing the flag and its value, is the **topic** — the article title/topic. If empty, infer from the conversation context.

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

### 1. Identify the content to save

Look at the recent conversation. The content to save is typically:

- An answer to a complex question the user asked
- An analysis or comparison the user found valuable
- A synthesis that connects multiple ideas
- A decision or conclusion reached during discussion

If it's unclear what to save, ask the user.

### 2. Determine placement

Read the chosen vault's index (`cat <path>/Wiki/index.md`) and check for existing articles on the topic:

- **If a related article exists** → ask the user: update the existing article or create a new one?
- **If no related article exists** → create a new one

Pick the right subfolder (`Wiki/<Category>/`) using the same categories as wiki-compile (AI, Engineering, Business, Books, etc.).

### 3. Write the article

Structure the conversation content into a clean article. Not a transcript dump — synthesize it into something readable.

Read the article template from `${CLAUDE_SKILL_DIR}/assets/article-template.md` and fill in the placeholders. That file is the source of truth — edit it to change the default article shape for saved articles.

Key rules:

- **Always use `[[wikilinks]]`** — every reference to a Wiki article or Raw clipping must be a wikilink. No plain paths.
- **`origin: conversation`** in frontmatter — distinguishes articles born from Q&A vs compiled from Raw sources.
- **Synthesize, don't transcript** — the article should read as a standalone piece, not as "I asked X and you said Y."
- **Link related Wiki articles** inline where relevant.
- **Granular type** — pick from `concept`, `guide`, `pattern`, `analysis`, `comparison`, `reference` rather than generic `notes`. Add a new one only if none fit.
- **Preserve specifics** — concrete examples, numbers, code patterns, and named tools/frameworks. Before closing the article, check against the conversation: did the named people / tools / companies survive? The numbers and dates? The concrete examples (commands, snippets, workflows)? The tensions or disagreements? If the article could have been written from the title alone, rewrite it.

#### Enriching vs splitting

If a related Wiki article already exists, the default pull is to append. Resist it when the new material is actually a different concept wearing the same category.

Split into a new article (and wikilink from the old one) when:

- You're about to open a third top-level `##` section about a sub-topic that wasn't in the original scope.
- The sub-topic has its own vocabulary, its own named tools, its own sources — it could stand alone.
- You can't summarize the resulting article in one sentence without using "and" to join two unrelated concepts.
- A reader looking only for the sub-topic would have to scroll past unrelated material to find it.

Enrich the existing article (don't split) when the new material adds depth to what's already there: more examples, sharper definitions, a counter-argument, a concrete number.

#### Tone

Wiki articles are read, not sold. Reference entries, not blog posts. Flat, factual, present tense. The reader came for the facts — don't perform, don't hype, don't narrate.

Avoid:

- **Hype adjectives**: "powerful", "revolutionary", "game-changing", "seminal", "elegant", "robust".
- **Editorial voice**: "interestingly", "notably", "it's worth noting", "crucially", "importantly".
- **Progressive narrative**: "would later become", "eventually led to", "goes on to".
- **Empty qualifiers**: "genuine", "profound", "deep", "meaningful".
- **Decorative em dashes** (—). Use a period, a comma, or parentheses. Reserve em dashes for real asides.

Prefer short affirmative sentences. When a phrase from the conversation carries weight, quote it directly — the quote holds the emotion, the surrounding prose stays neutral.

### 4. Create via obsidian CLI

```bash
obsidian vault=<name> create path="Wiki/<Category>/<article-name>.md" content="<full markdown>" silent
```

Then force date types:

```bash
obsidian vault=<name> property:set name="created" value="YYYY-MM-DD" type=date path="Wiki/<Category>/<article-name>.md"
obsidian vault=<name> property:set name="updated" value="YYYY-MM-DD" type=date path="Wiki/<Category>/<article-name>.md"
```

### 5. Update index

Add the new article to the chosen vault's `Wiki/index.md` under the appropriate category heading:

```markdown
- [[Wiki/<Category>/<name>|<Title>]] — <one-line hook phrase>
```

### 6. Confirm

Tell the user what was saved and where:

```
Saved to Wiki/AI/llm-wiki-pattern.md
Updated Wiki/index.md.
```
