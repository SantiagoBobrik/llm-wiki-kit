# Wiki.md

Contract for the `Wiki/` knowledge base. Skills (`wiki-compile`, `wiki-lint`, `wiki-save`, `wiki-search`) read this file as the source of truth for structure and style.

## Structure

```
Raw/          ← immutable source clippings (web, threads, gists, papers)
  log.md      ← append-only compile log
Wiki/         ← LLM-owned articles, synthesized from Raw/
  index.md    ← flat catalog, one line per article
  <Category>/ ← AI, Engineering, Business, Health, Finance, Career, Books, Podcasts…
```

- **Raw/** is the source of truth. Never modified by the LLM.
- **Wiki/** is compiled output. The LLM creates, updates, and reorganizes freely.
- **index.md** and **log.md** are plain markdown, not dataview. One line per entry, wikilinks + hook phrase.

## Frontmatter

Minimum for every Wiki article:

```yaml
type: <concept | guide | pattern | analysis | comparison | reference>
created: YYYY-MM-DD
updated: YYYY-MM-DD
owner: <user>
tags:
  - wiki
  - <topic tags>
sources:
  - "[[Raw/<filename>]]"
```

- `type` is free-form but prefer the list above. Add a new one only if none fit.
- `tags` must include `wiki`. Add topic tags for discoverability.
- `sources` must wikilink every Raw/ clipping that fed the article.

## Writing rules

### One article = one concept

If you can't describe the article in one sentence without using "and" to join two unrelated concepts, it's two articles. Link them instead of merging.

Reference docs that enumerate parts of one system (APIs, component catalogs) are the exception — length is fine when the parts belong together.

### Synthesize, don't summarize

Weave multiple sources into one coherent piece. Don't write "Source A says X, Source B says Y." The article should read as a single voice.

### Preserve specifics

Before closing an article, check it against its sources:

- Named people, tools, companies → survived?
- Numbers, dates, versions → survived?
- Concrete examples (commands, snippets, workflows) → survived?
- Tensions or disagreements between sources → surfaced?

If the article could have been written from the title alone, rewrite it.

### Tone

Wiki articles are read, not sold. Reference entries, not blog posts. Flat, factual, present tense. The reader came for the facts — don't perform, don't hype, don't narrate. If the idea is good, the details show it.

Avoid:

- **Hype adjectives**: powerful, revolutionary, groundbreaking, seminal, elegant, robust.
- **Editorial voice**: interestingly, notably, worth noting, crucially, importantly.
- **Progressive narrative**: would later become, eventually led to, goes on to.
- **Empty qualifiers**: genuine, profound, deep, meaningful.
- **Decorative em dashes**. Use a period, comma, or parentheses. Reserve em dashes for real asides.

Prefer short affirmative sentences. When an author's phrasing carries weight, quote it directly — the quote holds the emotion, the prose stays neutral.

## Linking

- Always wikilink: sources, related Wiki articles, index entries, log entries. No plain paths.
- Link related Wiki articles inline and naturally. No forced "Related" section.
- `Wiki/index.md` format: `- [[Wiki/<Category>/<name>|<Title>]] — <one-line hook>`
- `Raw/log.md` format: one `## [YYYY-MM-DD] compile` header per run, nested wikilinks to articles and sources.
