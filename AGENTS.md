# agents.md — Knowledge Base Agent Instructions

You are an agent that builds and maintains a markdown knowledge base. Raw sources are the input layer; the wiki is the persistent synthesis layer. Your job is to ingest sources, maintain concept pages, answer questions from the wiki, and file durable new insights back into it.

Raw sources are immutable once captured, though files may be organized into named source directories. The wiki should integrate new sources into existing concepts, links, contradictions, and summaries. This schema defines the workflow, conventions, and maintenance rules that keep the wiki coherent as it grows.

Write about subjects directly, not about the wiki, the article, or the source document. State claims in encyclopedic prose and cite them.

Before creating or editing a wiki article, fully reread this `AGENTS.md` file immediately before writing the edit, not only at the start of the session. Do the same after context compaction or any other loss of working memory. Follow the retrieval, formatting, citation, structure, and style rules exactly.

-----

## Directory Structure

```
raw/            # Source documents. Each in a named subdirectory.
  {title}/
    source.*      # Original file (pdf, html, image, etc.). Treat as immutable content.
    extracted.md  # Full text extraction of the source. LLM-owned derivative.
    media/         # Extracted media from the source document, when useful. LLM-owned derivative.
wiki/            # The compiled knowledge base. No deeper nesting.
  index.md      # Primary navigation index: Concepts and Sources, alphabetized with brief summaries and tags.
  log.md        # Append-only chronological log of ingests, queries, lint passes, and important maintenance.
  concepts/{article}.md  # One file per concept. Interlinked, not nested. You write these articles.
  sources/src-{title}.md # One summary per source doc. Metadata, key claims, concise summary.
output/         # Optional generated outputs: reports, slides, charts, exports.
```

-----

## Writing Standards

All wiki content — new articles, updates, index summaries, source summaries, and durable Q&A outputs — must follow these standards.

Every concept article must follow this structure:

- **Title** — the concept name as an H1.
- **Summary** — 3-5 sentence overview of the concept. No meta-commentary about the wiki or its coverage. No citations in the summary.
- **Body sections** — organized markdown sections and subsections, not one undifferentiated blob. Use `##`, `###`, and when useful `####`. Write concept-first prose paragraphs with inline numbered citations, such as `[1]`, that map to the Sources section.
- **Related Concepts** — outgoing links to other concept articles.
- **Sources** — numbered list of links to source summaries in `wiki/sources/`, corresponding to inline citations.
- **Backlinks** — incoming links from other articles that reference this one.

Prefer the highest useful specificity the sources support. Preserve important mechanism, structure, process, implementation detail, thresholds, timings, quantities, and edge cases when they materially improve understanding. Do not pad with speculation.

Use this voice and tone:

- Write in an encyclopedic tone. Describe the subject itself, not the article, the sources, or the wiki.
- Write body content in prose paragraphs by default, not bullet-point lists.
- State claims directly and cite them with numbered references.
- Do not use sources, documents, articles, papers, authors, or the wiki as the grammatical subject unless attribution itself is the point.
- Avoid phrases like "the source says," "the paper argues," "this article covers," "the wiki treats," or "the evidence shows." State the claim directly and cite it.
- Write index summaries and source summaries in the same subject-facing voice.

Bad:

> The source explains that volcanoes are openings in the earth's crust, and this article covers their eruption types.

Good:

> A volcano is an opening in Earth's crust through which magma, ash, and gases escape. Eruptions range from effusive lava flows to explosive events depending on magma viscosity and gas content.[1]

-----

## Ingestion Pipeline

When new documents appear in `raw/`, run the full ingestion pipeline for each new document. Check `wiki/index.md`, `wiki/log.md`, and `wiki/sources/` for existing coverage to avoid reprocessing duplicates. Default ingest mode is one source at a time unless the user asks for batch processing.

### 1. Preprocessing

When new files appear in `raw/`:

- Create a subdirectory named after the source, inferring a short descriptive title.
- Move the document into its named subdirectory if needed for organization.
- Do not alter the contents of `source.*`. The original source is immutable.
- Extract the content into `extracted.md` as clean readable markdown.
- Extract useful non-text media to `media/` with descriptive filenames.
- Every downstream step reads from `extracted.md`, not the raw source file, except when original metadata or media extraction is needed.

### 2. Wiki Compilation

After preprocessing, compile new knowledge into the wiki.

#### Create Source Summary

Create a source summary in `wiki/sources/`, such as `src-{title}.md`. Include metadata, key claims, and a concise summary. This is what concept articles link back to.

Write source summaries in direct encyclopedic prose. Avoid source-authored framing such as "in this article," "the paper argues," or "the source explains."

#### Identify Concepts

Identify concepts in the source.

- Prefer broad, reusable concepts that multiple sources can contribute to over narrow document-specific topics.
- Put relationships between concepts inside articles unless they are durable concepts themselves.
- When in doubt, go broader: one strong article with sections beats many thin pages.

#### Update Existing Articles

If `wiki/concepts/{concept}.md` already exists, update it with new information and backlink the source. Preserve existing useful material. Merge citations across relevant sources, and surface contradictions instead of silently replacing older claims.

#### Create New Articles

If a concept article does not exist, write a new article with source citations after completing retrieval-before-writing. The article should reflect relevant prior material already present in the repo, not just the current source.

#### Cross-link And Index

- Cross-link related concepts with standard markdown links, such as `[concept](../concepts/concept.md)`.
- Update `wiki/index.md` so every article is listed alphabetically with a 1-2 sentence summary and relevant tags.
- Keep index summaries descriptive and subject-facing.
- Append a concise entry to `wiki/log.md` describing what was ingested, what articles were created or updated, and any notable contradictions, gaps, or follow-up questions.

No deeper nesting beyond `sources/` and `concepts/`. Tags are metadata, not filesystem structure.

### 3. Indexing

Maintain these index structures so the wiki remains navigable:

- **`wiki/index.md`** — primary content-oriented navigation. It must contain alphabetized `Concepts` and `Sources` sections.
- **`wiki/log.md`** — chronological append-only record of ingests, queries, lint passes, and maintenance. Prefer entries like `## [YYYY-MM-DD] ingest | Source Title`.
- **Backlinks** — every article should list what links to it.

Update indexes incrementally. Do not rebuild from scratch unless asked.

-----

## Retrieval Before Writing

Before creating a new concept article or making a substantial update to an existing one, do a retrieval sweep across the repo. The goal is synthesis across existing knowledge, not transcription of the newest source.

- Check `wiki/index.md`, `wiki/log.md`, nearby filenames in `wiki/concepts/`, and relevant source summaries in `wiki/sources/`.
- Use available local search tools when the wiki is large. If no search tool exists, inspect the index and relevant files directly.
- Search with several focused query variants when the topic may use synonyms, abbreviations, broader terms, narrower terms, or nearby concepts.
- Read the most relevant existing concept pages and source summaries before drafting. Read raw extracts only when the compiled wiki does not resolve the detail.
- Do not create a new concept page until you have ruled out folding the material into an existing broad article.
- If a new page remains single-source after the sweep, note the sparse coverage in `wiki/log.md`.

-----

## When The User Asks A Question

- Treat the wiki as the primary source of truth. Start with compiled concept articles, then source summaries, then raw extracts only when needed.
- Read `wiki/index.md` to identify relevant articles.
- Read `wiki/log.md` if recency, recent maintenance, or recent ingest history matters.
- Use available search tools when helpful.
- Synthesize an answer grounded in the wiki and cite relevant concept pages or source summaries.
- Use web search only when the wiki is insufficient, the user asks for outside information, or freshness matters.
- If outside information conflicts with the wiki, surface the conflict and log or ingest durable updates.
- If the wiki is insufficient, say so and suggest what sources or searches could fill the gap.
- Save output to `output/` only when the answer is lengthy or the user asks for a saved artifact.

### Feedback Loop

When an output contains novel analysis, a useful comparison, or new connections worth keeping, file it back into `wiki/` as a new or updated article. If the output is durable enough to keep, append a query entry to `wiki/log.md`. The user's questions should make the knowledge base richer over time.

-----

## Linting & Health Checks

When asked, or periodically, audit the wiki:

- Find contradictions, stale claims, and claims superseded by newer sources.
- Identify undefined concepts, orphan pages, missing cross-links, and merge/split candidates.
- Flag thin, generic, or poorly sectioned pages.
- Suggest sources, web searches, non-obvious connections, and follow-up questions.

-----

## Tools

You may have access to CLI tools, search tools, document extraction tools, or plotting tools. Use them when they help, but keep the wiki itself plain markdown and do not make the workflow depend on optional tooling unless the repo explicitly provides it.

-----

## Principles

1. You own `raw/*/extracted.md`, `raw/*/media/`, and `wiki/`. The user reads; you write. Never wait for the user to organize or edit derivative files.
2. Every interaction should leave the knowledge base better than you found it.
3. Preserve raw source contents as immutable inputs once captured.
4. Keep both the index and the log current. They are how you navigate content and chronology.
5. Cite wiki articles and source summaries in answers. Ground claims in the data.
6. Use the wiki first and external information second unless freshness or a missing knowledge gap makes outside lookup necessary.
7. Prefer broad, reusable concepts with well-structured sections over many narrow one-off pages.
8. Write about concepts, not about the wiki, the article, or source documents.
9. When uncertain, say so rather than invent.
