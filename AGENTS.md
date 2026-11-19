# Knowledge base authoring instructions

This repository is the canonical Obsidian vault for research and independent personal interests. It is public and is published through the separate Quartz site repository.

## Scope

- PhD research and source-specific literature notes.
- Physics and computing explanations that can outlive any single paper or project.
- Independent interests (philosophy, books, culture, outdoors, etc.) in `Personal/` and ideas crossing domains in `Concepts/`.
- Preserve reusable understanding, not chat transcripts. Prefer updating an existing relevant note to creating overlapping notes.
- Do not modify the site's source repository unless explicitly requested.

## Folders and note types

- `PhD/Literature/`: one source note per paper, with references and page/figure provenance.
- `PhD/Projects/`: questions, methods, and current research plans.
- `Physics/`: explanatory physics notes; `Physics/Heavy-Ion/` for QGP-related concepts.
- `Computing/`: software, HPC, GPU, and ML concepts.
- `Concepts/`: genuinely cross-disciplinary mental models and methods.
- `Personal/`: interests beyond research; allow cross-links to all other sections.
- `Research Logs/`: dated investigations, benchmarks, and decisions.
- `Inbox/`: notes not yet organized; not private.
- `Templates/`: starter templates, not substantive public notes.

Use folders flexibly; one topic per evergreen concept note. Literature notes and chronological logs record provenance. Link related ideas instead of duplicating their content. Do not rename files without considering wikilinks and public URLs.

## Editorial rules

- Write notes in English unless instructed otherwise.
- Use Markdown and YAML frontmatter supported by Obsidian and Quartz.
- Distinguish established facts, direct observations, personal interpretation, hypotheses, and open questions.
- Attribute measurements to their environment, workload, methodology, and date; never present preliminary benchmarks as universal results.
- Cite primary sources when possible: papers, pages or figures, documentation, source code, PRs, issues, and reproducible commands.
- Use full-path wikilinks when a note basename could be ambiguous; verify links after changes.
- Include LaTeX math when appropriate. Avoid fabricated references or experimental results.
- Do not publish secrets, credentials, private correspondence, private journal material, unpublished CMS collaboration-sensitive material, confidential PDFs, or personal data. Keep these in a separate private space; `draft: true` alone does not guarantee exclusion from publishing.

## GitHub workflow

1. Inspect existing notes and latest `main`.
2. Create a topic branch named `notes/<short-topic>`.
3. Add or edit notes; check metadata, links, references, and publication suitability.
4. Open a pull request against `main` with a summary and validation notes.
5. Leave merging and publication to the repository owner. Never push directly to `main`.

## Writing standards

Preferred YAML fields: `title`, `description`, `tags`, `date`, and `draft`. Use ISO 8601 dates where known; do not invent dates. Use `draft: true` for uncertain or incomplete material that is not ready for editorial publication. Actual publication filtering depends on Quartz configuration.
