---
name: paper-vault
description: >
  Obsidian paper management system — search IVF/reproductive medicine papers, auto-tag, build Knowledge Graph.
  Use this skill when:
  - User mentions "find papers", "latest N papers", "search paper", "arXiv", "PubMed"
  - User requests "organize papers", "add tags", "add to vault", "save to obsidian"
  - User asks about "Pregnancy Prediction", "Gardner Grade", "PGT", "IVF", "embryo" papers
  - User searches for papers by company (Alife, Presagen, AiVF, KaiHealth, etc.)
license: MIT
metadata:
  author: nayo
  version: 1.2.2
  category: research
  tags: [obsidian, papers, pdf, markdown, research, knowledge-base, IVF]
---

# Paper Vault Skill

Obsidian-based system for managing IVF/reproductive medicine papers.
Searches papers, saves them as structured notes, tags with a fixed taxonomy, and builds a Knowledge Graph.

## Vault Path

Notes are **always written to the Obsidian vault**, regardless of where PDFs are stored.

Vault path is set via `vault_papers_path` (configured at plugin install time).
Default: `~/Documents/Obsidian Vault`

```
{vault_papers_path}/          ← Obsidian vault (notes written here)
└── Papers/
    ├── Notes/      ← paper notes (.md)
    └── Keywords/   ← category index (Knowledge Graph hub)
```

PDF source can be anywhere — local folder or Google Drive Desktop sync path:
```
~/Library/CloudStorage/GoogleDrive-[email]/Shared drives/[Drive Name]/
```
Pass the PDF folder path when invoking Mode 2. Notes always go to `vault_papers_path`.

---

## Tagging Taxonomy (fixed list)

Do NOT create tags freely. Match only against the two fixed lists below.
If no match — leave empty. Never force a tag.

**Tag format — always use nested tags (Obsidian `/` prefix):**
- Research area tags → `research/[tag-slug]` (e.g. `research/pregnancy-prediction`)
- Affiliation tags → `affiliation/[tag-slug]` (e.g. `affiliation/alife`)
- Base tag → `paper`

This enables filtering by category in Obsidian Tags panel and Knowledge Graph.

> Full classification criteria: [references/taxonomy.md](references/taxonomy.md)

### 1. Research Area

| Tag | Match criteria (if abstract/title contains) |
|-----|---------------------------------------------|
| `Pregnancy Prediction` | pregnancy rate, clinical pregnancy, live birth, implantation prediction model |
| `Gardner Grade` | embryo morphological grading, blastocyst grade, Gardner scoring |
| `PGT Result Prediction` | PGT-A, PGT-M, ploidy prediction, euploidy, aneuploidy |
| `Oocyte` | oocyte quality, egg, ovarian reserve, oocyte selection |
| `Sperm` | sperm quality, motility, morphology, sperm selection, ICSI |
| `Time-lapse` | time-lapse, TLI, TLM, morphokinetics, morphokinetic, time-lapse microscopy |

Multiple tags allowed per paper if applicable.

### 2. Affiliation

Tag if the author's affiliation field contains any of the following (case-insensitive).

| Tag | Match strings |
|-----|---------------|
| `KaiHealth` | KaiHealth, Kai Health, Kai-Health |
| `Alife` | Alife, ALife |
| `Presagen` | Presagen |
| `AiVF` | AiVF, Ai-VF, AIVF |
| `Fairtility` | Fairtility |
| `Vitrolife` | Vitrolife |
| `Future Fertility` | Future Fertility, FutureFertility |
| `IVF 2.0` | IVF 2.0, IVF2.0 |
| `Embryonics` | Embryonics |
| `FertilAI` | FertilAI, Fertil AI |
| `IMVITRO` | IMVITRO, ImVitro |
| `MIM Fertility` | MIM Fertility, MIMFertility |
| `GENESYS AI LABS` | Genesys AI, GENESYS |

---

## Fields to Extract Per Paper

| Field | Description | If missing |
|-------|-------------|------------|
| `title` | Paper title | required |
| `authors` | Author list | required |
| `affiliations` | Author institutions | "" |
| `year` | Publication year | "" |
| `journal` | Journal/conference name | "" |
| `doi` | DOI | "" |
| `arxiv_id` | arXiv ID | "" |
| `abstract` | Full abstract | "" |
| `performance_metrics` | Performance figures (AUC, accuracy, sensitivity, etc.) | "" |
| `is_multicenter` | Whether multi-center study | false |
| `research_area_tags` | Research area tags (taxonomy match) | [] |
| `affiliation_tags` | Affiliation tags (list match) | [] |

### Extracting performance_metrics
Pull numeric results directly from abstract. Examples:
- "AUC 0.82" → `"AUC: 0.82"`
- "sensitivity 78%, specificity 91%" → `"sensitivity: 78%, specificity: 91%"`
- Leave empty if no figures present.

### Determining is_multicenter
Set `true` if abstract/title contains: "multi-center", "multicenter", "multiple centers", "multi-institutional".

---

## Paper Note Format

Save path: `Papers/Notes/[YEAR] [Title].md` (title truncated to 100 chars at word boundary, special chars removed)

> Keyword normalization rules: [references/keyword_normalization.md](references/keyword_normalization.md)

```markdown
---
title: "Paper title"
authors:
  - "Author 1"
  - "Author 2"
affiliations:
  - "Institution 1"
  - "Institution 2"
year: 2024
journal: "Journal name"
doi: "10.xxxx/xxxxx"
arxiv_id: ""
performance_metrics: "AUC: 0.82, sensitivity: 78%"
is_multicenter: false
research_area:
  - "Pregnancy Prediction"
  - "Gardner Grade"
affiliation_tags:
  - "Alife"
tags:
  - paper
  - research/pregnancy-prediction
  - research/gardner-grade
  - affiliation/alife
date_added: "2026-04-14"
---

# Paper title

## Abstract

(full abstract)

## Tags
**Research Area:** [[Pregnancy Prediction]] | [[Gardner Grade]]
**Affiliation:** [[Alife]]

## Performance Metrics
AUC: 0.82, sensitivity: 78%

## Multi-center
No

## Links
- [DOI](https://doi.org/10.xxxx/xxxxx)
```

**Filename rule:** `[YEAR] Title.md` — year omitted if unknown. Remove special chars (`:`, `/`, `?`, etc.). Truncate at 100 chars on word boundary (never cut mid-word).

---

## Keyword Index Note Format

`Papers/Keywords/[category].md` — hub note for each tag.

```markdown
---
type: keyword-index
keyword: "Pregnancy Prediction"
paper_count: 12
last_updated: "2026-04-14"
---

# Pregnancy Prediction

Papers on pregnancy rate, live birth, and implantation prediction models.

## Papers
- [[2024 AI prediction models time-lapse embryo]]
- [[2023 Development and validation of live birth prediction]]

## Related Categories
[[Gardner Grade]] | [[PGT Result Prediction]]
```

When adding a new paper: append to the paper list in the existing index file and update `paper_count` and `last_updated`.

---

## Workflow

### Mode 1: Search & Save Papers

When user requests "find N papers on topic X".

> Company author/product reference: [references/known_authors.md](references/known_authors.md)

**STRICT RULES — must follow exactly:**
- Maximum 2 search calls total per request. Stop after 2 regardless of results.
- Save exactly the number of papers requested by the user. If user says "1편", save 1 note only. If user says "5편", save up to 5.
- Do NOT use fetch, WebFetch, curl, or any URL fetching tool to verify affiliations.
- Do NOT use ORCID, journal websites, or PDF downloads.
- If affiliation cannot be confirmed from DB results → tag as `affiliation_tags: []` and proceed. Never block note creation on affiliation uncertainty.

```
1. Identify topic → choose ONE source. Do NOT search multiple sources.
   - Specific company papers → check references/known_authors.md first
     Use product name or author name, NOT company name directly
     e.g. "KaiHealth" → search "VITA EMBRYO" or "Hye Jun Lee embryo"
   - IVF/reproductive medicine → PubMed ONLY
   - AI/ML methods → arXiv ONLY
   - 0 results → one retry with Semantic Scholar (final attempt)

2. Dedup check BEFORE saving:
   - mcp__obsidian__search-vault(query=doi)
   - DOI already in vault → skip, pick next result
   - Only save papers not already in vault

3. Search via paper-search MCP (maximum 2 calls total)
   mcp__paper-search-mcp-openai-v2__search_pubmed   (medical)
   mcp__paper-search-mcp-openai-v2__search_arxiv    (CS/AI)
   mcp__paper-search-mcp-openai-v2__search_semantic (general + affiliation)

4. Check search result completeness FIRST (skip DB calls if already complete):
   - Search result has title + authors + abstract + DOI → skip to step 5 immediately
   - DOI missing → get_crossref_paper_by_doi to retrieve DOI (then check abstract)
   - Abstract missing (but DOI present):
     a. read_semantic_paper ONCE for abstract only
     b. still missing → proceed with empty abstract. Stop here.
   - Never call CrossRef if the search result already contains title + authors + abstract + DOI.

5. Apply tags (taxonomy match)
   - abstract + title → research_area_tags
   - affiliation → affiliation_tags
   - No match → empty array. Never force.

6. Create note → update keyword index
   - New note → mcp__obsidian__create-note(path="Papers/Notes/[filename].md", content=...)
   - Update keyword index for EACH matched tag (both research_area_tags AND affiliation_tags):
     - Index exists → mcp__obsidian__read-note then mcp__obsidian__edit-note
     - Index missing → mcp__obsidian__create-note(path="Papers/Keywords/[tag].md", content=...)
   - Examples:
     - research/pregnancy-prediction → `Papers/Keywords/Pregnancy Prediction.md`
     - affiliation/alife → `Papers/Keywords/Alife.md`
     - affiliation/kaihealth → `Papers/Keywords/KaiHealth.md`

7. Report summary to user
```

### Mode 2: Tag PDF Files (Local or Google Drive)

When user wants to create notes from PDF files stored locally or in a Google Drive Desktop sync folder.

**Google Drive 사용 시 — "Paper" 공유드라이브만 허용:**
- 허용: `~/Library/CloudStorage/GoogleDrive-[email]/Shared drives/Paper/` 하위 폴더
- 금지: Paper 외 다른 공유드라이브 또는 개인 드라이브 (`My Drive/`)
- 경로에 `Shared drives/Paper`가 포함되지 않으면 실행 전에 사용자에게 경로를 재확인할 것

PDF source path examples:
- Local: `/Users/yourname/Documents/Papers/`
- Google Drive: `~/Library/CloudStorage/GoogleDrive-[email]/Shared drives/Paper/Embryo AI/`

Notes are always written to `{vault_papers_path}/Papers/Notes/` regardless of PDF source.

```
1. Scan PDF folder via mcp__filesystem__list_directory
   List all .pdf files in the target directory

2. Skip already-processed PDFs (dedup check):
   - For each PDF, extract DOI from metadata (step 3a below — fast, no token cost)
   - Search existing notes for that DOI:
       mcp__obsidian__search-vault(query=doi)
   - If DOI found in any existing note → SKIP this PDF
   - If no DOI extractable → search by year+partial-title:
       mcp__obsidian__search-vault(query="[year] [partial title]")
     → SKIP if a matching note is found
   - Only process PDFs with no matching existing note

3. For each unprocessed PDF, extract DOI + title (no full-document read):

   a. Read PDF metadata dictionary (Python via Bash — zero token cost)
      import pymupdf
      meta = pymupdf.open(path).metadata
      → check meta['subject'] or meta['title'] for DOI pattern (10.\d{4,}/...)
      → use meta['title'] as title if present

   b. If no DOI found in metadata → read page 1 only (fallback)
      page1 = doc[0].get_text()
      → extract DOI with regex: r'10\.\d{4,9}/[^\s]+'
      → title comes from DB lookup (no need to parse from page)

   c. If still no DOI → use metadata title for DB search (last resort)

4. Fetch full metadata from DB:
   - DOI found → mcp__paper-search-mcp-openai-v2__get_crossref_paper_by_doi
     (most accurate — title, authors, journal, year)
   - Abstract missing from CrossRef result:
     a. First check PDF metadata['subject'] field (local Bash — no MCP cost)
     b. If metadata['subject'] has useful text → use it as abstract. Stop here.
     c. If still missing → read_semantic_paper ONCE for abstract only (last resort)
   - No DOI, has title → mcp__paper-search-mcp-openai-v2__search_semantic
     (returns abstract, affiliation, citations)

5. Apply tags (taxonomy match)
   - abstract + title → research_area_tags
   - affiliation → affiliation_tags

6. Create note → update keyword index
   - Same as Mode 1 step 6 — update index for EACH matched tag (both research_area_tags AND affiliation_tags)
   - research area tag → `Papers/Keywords/[Research Area].md`
   - affiliation tag → `Papers/Keywords/[Company].md`

7. Report summary
   - N new notes created
   - N skipped (already existed)
   - N failed (no metadata found)
```

### Mode 3: Browse by Category

When user asks "show me papers on Pregnancy Prediction".

```
1. mcp__obsidian__read-note("Papers/Keywords/[category].md")
2. Return paper list
```

---

## Search Source Guide

| Purpose | Recommended source |
|---------|-------------------|
| Latest IVF/reproductive medicine papers | PubMed (`search_pubmed`) |
| AI/deep learning methods | arXiv (`search_arxiv`) |
| Papers by company affiliation | Semantic Scholar (`search_semantic`) |
| High-citation important papers | Semantic Scholar |
| bioRxiv preprints | `search_biorxiv` |

---

## Result Report Format

Summarize to user after completion:

```
✅ N papers saved

| Title | Year | Research Area | Affiliation |
|-------|------|---------------|-------------|
| ...   | 2024 | Pregnancy Prediction | Alife |
| ...   | 2025 | PGT Result Prediction | - |

New category indexes created: Pregnancy Prediction, PGT Result Prediction
Updated indexes: Alife (3→4 papers)
```

---

## Notes

- Large batches (50+ papers) will incur API costs — test with a small set first.
- Relation analysis uses `claude-haiku-4-5-20251001` with max_tokens=4096.
- `vault_papers_path` is set during plugin install (default: `~/Documents/Obsidian Vault`). Must point to vault root — not the `Papers/` subfolder.

---

## File Structure

```
paper-vault/
├── .claude-plugin/
│   └── plugin.json               ← plugin manifest
├── skills/
│   └── paper-vault/
│       ├── SKILL.md              ← this file
│       └── references/
│           ├── taxonomy.md           ← tagging classification
│           └── keyword_normalization.md  ← keyword normalization map
├── CLAUDE.md                     ← developer context
└── README.md                     ← usage guide
```
