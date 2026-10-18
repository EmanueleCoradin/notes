# Knowledge base authoring instructions

This repository is the canonical Obsidian vault for research and independent scientific study. Content is published through the separate Quartz site repository.

## Scope
- PhD research, physics (including topics beyond CMS and the Standard Model), computing, algorithms, and exploratory investigations.
- Preserve reusable knowledge, not chat transcripts.
- Prefer updating an existing relevant note to creating overlapping notes.
- Do not modify the site's source repository unless explicitly requested.

## Editorial rules
- Write notes in English unless instructed otherwise.
- Use Markdown and YAML frontmatter supported by Obsidian and Quartz.
- One topic per conceptual note; chronological investigations belong in research logs.
- Distinguish established facts, direct observations, hypotheses, and open questions.
- Attribute measurements to their environment, workload, methodology, and date. Never present preliminary benchmarks as universal results.
- Cite primary sources when possible: papers, documentation, source code, PRs, issues, and reproducible commands.
- Use relative Markdown links or Obsidian wikilinks for internal cross-references. Check target names when editing.
- Include LaTeX math when appropriate. Avoid fabricated references or experimental results.
- Do not publish secrets, credentials, private correspondence, unpublished collaboration-sensitive material, or personal data.

## Suggested folders
- `PhD/` — research themes and literature
- `Physics/` — experimental and theoretical physics
- `Computing/` — software, HPC, GPU and ML topics
- `Concepts/` — reusable cross-disciplinary concepts
- `Research Logs/` — experiments, decisions and observations
- `Templates/` — drafting templates; not substantive public notes

Use folders flexibly. Keep existing filenames intact unless a rename is agreed on.

## GitHub workflow
1. Inspect existing notes and latest `main`.
2. Create a topic branch named `notes/<short-topic>`.
3. Add or edit notes; check metadata, links, references, and clarity.
4. Open a pull request against `main` with a summary, sources, and uncertainty/validation notes.
5. Leave merging and publication to the repository owner. Never push directly to `main`.

## Writing standards
Preferred YAML fields: `title`, `description`, `tags`, `date`, and `draft`. Use ISO 8601 dates and lowercase, simple tags. Do not invent dates: omit unknown values. Use `draft: true` for uncertain or incomplete material that is not ready to publish. Note that publication filtering depends on Quartz configuration.
