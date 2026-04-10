# Contributing

Thanks for your interest in llm-wiki-kit! Here's how to help.

## Getting started

1. Fork and clone the repo
2. Test locally: `claude --plugin-dir .`
3. Make your changes
4. Test your changes against a real Obsidian vault

## What you can contribute

- **Skill improvements** — better prompts, edge case handling, new arguments
- **New templates** — article formats, index layouts, log styles (goes in `skills/wiki-compile/assets/`)
- **Contract tweaks** — writing rules, frontmatter fields, tone guidance (edit `skills/wiki-init/assets/contract-template.md`)
- **Bug reports** — open an issue describing what happened vs. what you expected
- **Documentation** — README fixes, examples, translations

## Guidelines

- **Test with a real vault.** These are LLM skills, not code — the only way to test is to run them against actual Obsidian content.
- **One change per PR.** A skill fix is one PR; a new template is another.
- **Keep skills self-contained.** Each skill should work independently. Don't add cross-skill dependencies.
- **Don't change the contract without discussion.** The contract in `contract-template.md` affects every new vault. Open an issue first to discuss changes.
- **Match the existing style.** Skills use the same argument parsing pattern (`--vault`, positional args, vault resolution). Follow it.

## Reporting issues

Open an issue at [github.com/SantiagoBobrik/llm-wiki-kit/issues](https://github.com/SantiagoBobrik/llm-wiki-kit/issues) with:

- Which skill you ran
- What you expected
- What happened instead
- Your vault structure (if relevant)

## License

By contributing, you agree that your contributions will be licensed under the [MIT License](LICENSE).
