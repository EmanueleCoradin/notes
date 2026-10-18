# Research notes

A personal [Obsidian](https://obsidian.md/) knowledge base covering PhD research, physics, scientific computing, and independent investigations. The content is version-controlled here and rendered as a website by [notes-site](https://github.com/EmanueleCoradin/notes-site) using Quartz.

## Organization

- `PhD/`: research questions, literature, and methods.
- `Physics/`: theoretical and experimental physics beyond any single collaboration.
- `Computing/`: scientific software, ML inference, GPU computing, and performance.
- `Concepts/`: durable explanations shared across topics.
- `Research Logs/`: dated investigations, benchmarks, and decisions.
- `Templates/`: starting points for notes.

These are organizational conventions, not mandatory boundaries. Link related ideas using Obsidian wikilinks.

## Note types

**Concept notes** explain a stable topic, motivating questions, mechanisms, trade-offs, limitations, and sources. Use `Templates/Concept Note.md`.

**Research logs** record a specific investigation, including setup, observations, results, interpretation, and next steps. Use `Templates/Research Log.md`.

## Contributing and AI-assisted updates

Propose changes on a `notes/<topic>` branch and open a pull request to `main`. Review factual accuracy, citation quality, publication suitability, and any collaboration-sensitive details before merging. Avoid verbatim chat archives; convert discussions into useful standalone explanations. The detailed instructions for AI-assisted updates are in [AGENTS.md](AGENTS.md).

## Publishing

The separate `notes-site` repository embeds this repository as its `content` Git submodule. Merging a PR here changes the vault; site publication then depends on the configured GitHub Actions and deployment pipeline. Confirm that the submodule reference and live site actually update.
