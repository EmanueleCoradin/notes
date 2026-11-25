# Notes & explorations

**Website:** [https://emanuelecoradin.github.io/notes-site/](https://emanuelecoradin.github.io/notes-site/)

A personal [Obsidian](https://obsidian.md/) knowledge base for PhD research, physics, computing, and independent interests. The Markdown notes are version-controlled here and rendered as a website by [notes-site](https://github.com/EmanueleCoradin/notes-site) using Quartz.

## Structure

| Path | Purpose |
| --- | --- |
| `index.md` | Curated homepage and entry points |
| `PhD/` | PhD-specific literature, research questions, and projects |
| `Physics/` | Reusable physics explanations independent of a single source |
| `Computing/` | Scientific software, ML, GPU computing, and systems |
| `Concepts/` | Cross-disciplinary concepts and durable mental models |
| `Personal/` | Publicly shareable independent interests beyond research |
| `Research Logs/` | Dated investigations, measurements, and decisions |
| `Inbox/` | Unsorted ideas and legacy test notes |
| `Templates/` | Starting points for new notes |

The folders are useful entry points, not separate knowledge silos. Link between them with Obsidian wikilinks (ideally full paths such as `[[Physics/Heavy-Ion/index]]` when names are ambiguous).

### Two complementary note types

- **Literature notes** belong to sources: one entry per paper in `PhD/Literature/`, with citations, source-specific claims, your interpretation, and questions. Use `Templates/Literature Note.md`.
- **Concept notes** belong to ideas: a reusable explanation updated as you encounter multiple sources. Place them in `Physics/`, `Computing/`, `Concepts/`, or `Personal/` as appropriate. Use `Templates/Concept Note.md`.

**Research logs** capture dated experiments and decisions using `Templates/Research Log.md`. Use `Inbox/` for temporary drafts before finding their natural place.

Keep PDFs and bibliographic metadata in Zotero / the literature library rather than duplicating the PDFs in this public repository.

## Public versus private material

The repository is **public**, including `Personal/` and `Inbox/`. Personal means an independent subject area, not a private journal. Do not commit private notes, unpublished CMS collaboration material, confidential PDFs, credentials, personal data, or restricted work. Use a separate private vault or repository for such material. `draft: true` is editorial metadata and **not a security boundary**; whether Quartz excludes drafts depends on the site configuration.

## Contributing and AI-assisted updates

Propose changes on a `notes/<topic>` branch and open a pull request to `main`. Review factual accuracy, citations, cross-links, and publication safety before merging. Prefer standalone syntheses to pasted chat transcripts. See [AGENTS.md](AGENTS.md) for details.

## Publishing

The separate [notes-site](https://github.com/EmanueleCoradin/notes-site) repository embeds this repository as its `content` Git submodule. Merging here updates the vault, not necessarily the deployed website; verify that the submodule reference and site deployment update. Moving notes may change published URLs, so check links and redirects before publishing.
