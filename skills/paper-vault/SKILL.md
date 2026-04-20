---
name: paper-vault
description: >
  Obsidian paper management system — search IVF/reproductive medicine papers, auto-tag, build Knowledge Graph.
  Use this skill when:
  - User mentions "find papers", "latest N papers", "search paper", "arXiv", "PubMed"
  - User requests "organize papers", "add tags", "add to vault", "save to obsidian"
  - User asks about "Pregnancy Prediction", "Gardner Grade", "PGT", "IVF", "embryo" papers
  - User searches for papers by company (Alife, Presagen, AiVF, KaiHealth, etc.)
  - User wants to sync or import PDFs from Google Drive
license: MIT
metadata:
  author: nayo
  version: 1.4.1
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
    ├── Notes/        ← paper notes (.md)
    ├── Keywords/     ← research-area hub (one .md per research category)
    └── Affiliation/  ← affiliation hub (one .md per company/institution tag)
```

PDF source is **Google Drive (Shared Drive)**. PDFs are read directly via Google Drive MCP — no Desktop sync or "available offline" setup required.

The target folder is configured as `gdrive_shared_folder_id` (set at plugin install time).
Example shared drive folder ID: `0AAhracftG0XwUk9PVA`

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
Set `true` if abstract/title contains any of: "multi-center", "multicenter", "multiple centers", "multi-institutional", "multiple clinics", or if the study design mentions data collected from **2 or more named clinics/hospitals**.

---

## Paper Note Format

Save path: `Papers/Notes/[YEAR] [Title].md` (title truncated to 200 chars at word boundary, special chars removed)

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
pdf_source: "gdrive"
gdrive_file_id: "1abc..."
gdrive_url: "https://drive.google.com/file/d/1abc.../view"
added_by: "user@example.com"
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
- [Google Drive](https://drive.google.com/file/d/1abc.../view)
```

**Google Drive 필드 규칙:**
- `pdf_source`: Mode 2 (Drive) 경유 시 항상 `"gdrive"` 고정; Mode 1 (검색) 경유 시 필드 생략
- `gdrive_file_id`: `get_file_metadata` 응답의 `id` 값
- `gdrive_url`: `https://drive.google.com/file/d/{file_id}/view`
- `added_by`: `get_file_metadata` 응답의 `lastModifyingUser.emailAddress` (없으면 `""`)
- 위 세 필드는 Mode 2 전용 — Mode 1(검색)에서 생성하는 노트에는 포함하지 않는다

**Filename rule:** `[YEAR] Title.md` — year omitted if unknown. Remove special chars (`:`, `/`, `?`, etc.). Truncate at 200 chars on word boundary (never cut mid-word).

---

## Keyword index (research area)

`Papers/Keywords/[Research Area].md` — hub note for each **research_area** tag.

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

## Affiliation index

`Papers/Affiliation/[Company].md` — hub note for each **affiliation** tag (taxonomy company name as filename).

```markdown
---
type: affiliation-index
affiliation: "Alife"
paper_count: 5
last_updated: "2026-04-14"
---

# Alife

Papers with author affiliation matching Alife (per taxonomy).

## Papers
- [[2024 Example paper title]]

## Related affiliations
[[KaiHealth]] | [[Presagen]]
```

When adding a new paper: append the paper link to **each** hub file that applies (Keywords and/or Affiliation) and update `paper_count` and `last_updated` in that file.

> **⚠️ edit-note fallback:** `mcp__obsidian__edit-note` may fail with a schema error in some environments. If it does, use this fallback sequence:
> 1. `mcp__obsidian__read-note` — read the current hub note content
> 2. Modify the content in memory: increment `paper_count`, update `last_updated`, append the new paper link under `## Papers`
> 3. Write the full updated content directly to the vault path using the built-in **Write** or **Edit** file tool at `{vault_papers_path}/Papers/Keywords/[Research Area].md` or `{vault_papers_path}/Papers/Affiliation/[Company].md`
>
> The vault folder is always accessible as a mounted path in Cowork — do **not** fall back to `mcp__filesystem__*` tools.

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

6. Create note → update hub indexes
   - New note → mcp__obsidian__create-note(path="Papers/Notes/[filename].md", content=...)
   - For EACH `research_area_tags` match:
     - Index exists → mcp__obsidian__read-note then mcp__obsidian__edit-note
     - Index missing → mcp__obsidian__create-note(path="Papers/Keywords/[Research Area].md", content=...)
   - For EACH `affiliation_tags` match:
     - Index exists → mcp__obsidian__read-note then mcp__obsidian__edit-note
     - Index missing → mcp__obsidian__create-note(path="Papers/Affiliation/[Company].md", content=...)
   - Examples:
     - research/pregnancy-prediction → `Papers/Keywords/Pregnancy Prediction.md`
     - affiliation/alife → `Papers/Affiliation/Alife.md`
     - affiliation/kaihealth → `Papers/Affiliation/KaiHealth.md`

7. Report summary to user
```

### Mode 2: Tag PDF Files (Google Drive)

When user wants to create notes from PDF files stored in the Google Drive Shared Drive.

PDF는 Google Drive MCP로 **클라우드에서 직접** 읽는다 — Google Drive Desktop 설치나 "오프라인 사용 설정" 불필요.
Notes는 항상 `{vault_papers_path}/Papers/Notes/`에 저장된다.

대상 폴더: `https://drive.google.com/drive/folders/0AAhracftG0XwUk9PVA` (고정)

**수량 규칙:**
- 사용자가 수량을 명시한 경우 (예: "2편만", "5개") → 새 노트 N개 생성 후 즉시 중단. 이미 있는 노트는 카운트에 포함하지 않음.
- 수량 미명시 시 → 폴더 전체 처리

```
1. Find PDFs in Drive folder
   mcp__gdrive__search_files(
     folder_id="0AAhracftG0XwUk9PVA",
     query="mimeType='application/pdf'"
   )
   → file list: [{id, name, modifiedTime, lastModifyingUser}, ...]

2. Skip already-processed PDFs (dedup check) + count guard:
   - For each file, extract a candidate DOI hint from file name if possible
     (e.g. "10.1016_j.rbmo.2024.104123.pdf" → DOI hint)
   - mcp__obsidian__search-vault(query=doi_hint or file_name_stem)
   - Matching note found → SKIP (스킵된 파일은 N 카운트에 포함 안 함)
   - 새 노트 생성 완료 수가 N에 도달하면 → 남은 파일 처리 없이 즉시 step 7로 이동
   - Only process files with no matching existing note

3. For each unprocessed PDF, extract text and pull DOI + title:
   mcp__gdrive__read_file_content(file_id=...)
   → Returns extracted text of the PDF

   a. Apply DOI regex to extracted text:
      r'10\.\d{4,9}/[^\s\])]+'
      → use first match as DOI

   b. If no DOI found → extract candidate title from first 500 chars
      (first long line that is not a URL, author list, or journal name)
      → use as title for DB search fallback

   c. Get file metadata for Drive link + uploader:
      mcp__gdrive__get_file_metadata(file_id=...)
      → gdrive_file_id = file.id
      → gdrive_url = "https://drive.google.com/file/d/{file_id}/view"
      → added_by = file.lastModifyingUser.emailAddress (or "")

4. Fetch full metadata from DB:
   - DOI found → mcp__paper-search-mcp-openai-v2__get_crossref_paper_by_doi
     (most accurate — title, authors, journal, year)
   - Abstract missing from CrossRef result:
     a. Search extracted PDF text for "Abstract" section (regex, no MCP cost)
     b. If found → use as abstract. Stop here.
     c. If still missing → mcp__paper-search-mcp-openai-v2__read_semantic_paper
        ONCE for abstract only (last resort)
   - No DOI, has title → mcp__paper-search-mcp-openai-v2__search_semantic
     (returns abstract, affiliation, citations)

5. Apply tags (taxonomy match)
   - abstract + title → research_area_tags
   - affiliation → affiliation_tags

6. Create note → update hub indexes
   - Include Drive fields in frontmatter: pdf_source, gdrive_file_id, gdrive_url, added_by
   - Same hub update logic as Mode 1 step 6
   - research area tag → `Papers/Keywords/[Research Area].md`
   - affiliation tag → `Papers/Affiliation/[Company].md`

7. Report summary
   - N new notes created
   - N skipped (already existed)
   - N failed (no metadata found)
```

### Mode 3: Browse by Category

When user asks "show me papers on Pregnancy Prediction" or "papers from Alife".

```
1. Resolve category against taxonomy:
   - Research Area (Pregnancy Prediction, Gardner Grade, …) → mcp__obsidian__read-note("Papers/Keywords/[category].md")
   - Affiliation (KaiHealth, Alife, …) → mcp__obsidian__read-note("Papers/Affiliation/[category].md")
2. Return paper list from the hub note's ## Papers section
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

New keyword indexes: Pregnancy Prediction, PGT Result Prediction
New affiliation indexes: Alife
Updated keyword indexes: Gardner Grade (5→6 papers)
Updated affiliation indexes: KaiHealth (2→3 papers)
```

---

## Notes

- Large batches (50+ papers) will incur API costs — test with a small set first.
- Relation analysis uses `claude-haiku-4-5-20251001` with max_tokens=4096.
- `vault_papers_path` is set during plugin install (default: `~/Documents/Obsidian Vault`). Must point to vault root — not the `Papers/` subfolder.
- Google Drive 대상 폴더는 `0AAhracftG0XwUk9PVA`로 고정 (`https://drive.google.com/drive/folders/0AAhracftG0XwUk9PVA`). 변경 시 SKILL.md Mode 2 step 1의 `folder_id` 값을 직접 수정.
- Mode 2 reads PDFs directly from Google Drive via MCP — no Google Drive Desktop or "offline" sync required.
- `mcp__gdrive__read_file_content` returns text extracted from the PDF; DOI regex is applied to this text. Quality is comparable to pymupdf page-1 extraction for well-formatted PDFs.
- `added_by` reflects the **last modifier**, not the original uploader — this is a Google Drive API limitation.

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
