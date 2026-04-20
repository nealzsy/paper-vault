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
  version: 1.5.5
  category: research
  tags: [obsidian, papers, pdf, markdown, research, knowledge-base, IVF]
---

# Paper Vault Skill

Obsidian-based system for managing IVF/reproductive medicine papers.
Searches papers, saves them as structured notes, tags with a fixed taxonomy, and builds a Knowledge Graph.

## Vault Path

Notes are **always written to the Obsidian vault**, regardless of where PDFs are stored.

**Before doing anything else, auto-detect the vault path:**

1. Call `mcp__filesystem__list_allowed_directories` to get all mounted paths.
2. The vault is the mounted path that is **also listed as an obsidian-mcp vault arg** in the user's MCP config.
   - If multiple paths are mounted, pick the one that matches an obsidian-mcp vault path.
   - If still ambiguous, check which path contains a `Papers/` subfolder.
3. **Never ask the user** — always determine from mounted directories automatically.

> Example: if `list_allowed_directories` returns `/Users/x/agent/paper-vault` and `/Users/x/agent/link-vault`,
> and obsidian-mcp is configured with `/Users/x/agent/link-vault` as a vault arg → use `link-vault` as vault root.

Vault path is auto-detected per session via `mcp__filesystem__list_allowed_directories`.

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
| `keywords` | Normalized keywords from paper metadata or abstract | [] |
| `performance_metrics` | Performance figures (AUC, accuracy, sensitivity, etc.) | "" |
| `is_multicenter` | Whether multi-center study | false |
| `research_area_tags` | Research area tags (taxonomy match) | [] |
| `affiliation_tags` | Affiliation tags (list match) | [] |

### Extracting and normalizing keywords
1. Source: use paper's own `keywords` field from DB result (CrossRef/Semantic). If empty, extract 3–7 key terms from abstract.
2. Normalize each keyword using [references/keyword_normalization.md](references/keyword_normalization.md):
   - Look up raw form in the map → replace with normalized form
   - e.g. `"Deep Learning"` → `"deep learning"`, `"PGT-A"` → `"preimplantation genetic testing"`
   - No match → keep original, lowercased
3. Store normalized list in `keywords` frontmatter field.
4. **Use normalized keywords for taxonomy matching** (step Apply tags) — more reliable than raw abstract text.

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
keywords:
  - "deep learning"
  - "embryo selection"
  - "time-lapse imaging"
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
```

When adding a new paper: append the paper link to **each** hub file that applies (Keywords and/or Affiliation) and update `paper_count` and `last_updated` in that file.

> **⚠️ obsidian-mcp 쓰기 불가 (Cowork 환경):** `create-note`, `edit-note` 등 모든 **쓰기** 작업은 obsidian-mcp에서 타임아웃이 발생한다. Cowork에서는 볼트 폴더가 직접 마운트되어 있으므로 **쓰기는 항상 내장 파일 툴을 사용**한다.
>
> **dedup 체크도 `search-vault` 대신 파일시스템으로:** `search-vault`는 결과가 부정확할 수 있으므로 Glob 또는 Bash `ls`로 `Papers/Notes/` 파일 목록을 직접 확인한다.
>
> **볼트 경로 (Cowork 마운트):** `mcp__filesystem__list_allowed_directories`로 마운트 경로 목록 취득 → obsidian-mcp vault args와 대조해 일치하는 경로를 볼트로 사용. 하드코딩 금지.
>
> 작업별 대체 방법:
> - **노트 생성** → built-in `Write` 툴로 `{vault_mount}/Papers/Notes/[filename].md` 직접 작성
> - **허브 읽기** → built-in `Read` 툴로 `{vault_mount}/Papers/Keywords/[Research Area].md` 읽기
> - **허브 업데이트** → `Read` 후 `Edit` 툴로 수정 (paper_count +1, last_updated, ## Papers에 링크 추가)
> - **dedup 체크** → `Glob("{vault_mount}/Papers/Notes/*.md")` 또는 Bash `ls`로 파일명 비교
> - **노트 읽기** → built-in `Read` 툴 사용
>
> obsidian-mcp는 **읽기 전용 보조**로만 사용 (list-available-vaults로 볼트명 확인 등). 절대 `mcp__filesystem__*` 툴은 사용하지 않는다.

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

2. Dedup check BEFORE saving (파일시스템 직접 확인):
   - Glob("{vault_mount}/Papers/Notes/*.md") 로 기존 노트 파일명 목록 취득
   - DOI 또는 title keyword가 파일명에 포함되어 있으면 → skip, pick next result
   - Only save papers not already in vault

3. Search via paper-search MCP (maximum 2 calls total)
   mcp__plugin_paper-vault_paper-search-mcp-openai-v2__search_pubmed   (medical)
   mcp__plugin_paper-vault_paper-search-mcp-openai-v2__search_arxiv    (CS/AI)
   mcp__plugin_paper-vault_paper-search-mcp-openai-v2__search_semantic (general + affiliation)

4. Check search result completeness FIRST (skip DB calls if already complete):
   - Search result has title + authors + abstract + DOI → skip to step 5 immediately
   - DOI missing → get_crossref_paper_by_doi to retrieve DOI (then check abstract)
   - Abstract missing (but DOI present):
     a. read_semantic_paper ONCE for abstract only
     b. still missing → proceed with empty abstract. Stop here.
   - Never call CrossRef if the search result already contains title + authors + abstract + DOI.

5. Normalize keywords + apply tags
   a. Extract keywords from DB result's keywords field (or top terms from abstract if empty)
   b. Normalize using references/keyword_normalization.md → store as `keywords` list
   c. Match normalized keywords + title against taxonomy → research_area_tags
   d. Match affiliation field against taxonomy → affiliation_tags
   - No match → empty array. Never force.

6. Create note → update hub indexes (파일 툴 직접 사용)
   - New note → built-in Write 툴로 `{vault_mount}/Papers/Notes/[filename].md` 직접 작성
   - For EACH `research_area_tags` match:
     - Index exists → Read 툴로 읽고 Edit 툴로 수정 (paper_count +1, last_updated, ## Papers에 링크 추가)
     - Index missing → Write 툴로 `{vault_mount}/Papers/Keywords/[Research Area].md` 신규 생성
   - For EACH `affiliation_tags` match:
     - Index exists → Read 툴로 읽고 Edit 툴로 수정
     - Index missing → Write 툴로 `{vault_mount}/Papers/Affiliation/[Company].md` 신규 생성
   - Examples:
     - research/pregnancy-prediction → `{vault_mount}/Papers/Keywords/Pregnancy Prediction.md`
     - affiliation/alife → `{vault_mount}/Papers/Affiliation/Alife.md`
     - affiliation/kaihealth → `{vault_mount}/Papers/Affiliation/KaiHealth.md`

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
   gdrive__search_files(query="mimeType='application/pdf'")
   → `parents` 필터 미지원 — folder_id 파라미터 사용 금지. mimeType만으로 검색.
   → file list: [{id, name, title, viewUrl, modifiedTime}, ...]

2. Skip already-processed PDFs (dedup check) + count guard:
   - Glob("{vault_mount}/Papers/Notes/*.md") 로 기존 노트 파일명 목록 취득 (1회만, 전체 처리 전 미리)
   - For each file, extract a candidate DOI hint or title stem from file name
     (e.g. "10.1016_j.rbmo.2024.104123.pdf" → DOI hint; "deae108.553.pdf" → stem "deae108.553")
   - 기존 노트 목록에 doi_hint 또는 file_name_stem이 포함된 파일명이 있으면 → SKIP
   - 새 노트 생성 완료 수가 N에 도달하면 → 남은 파일 처리 없이 즉시 step 7로 이동
   - Only process files with no matching existing note

3. For each unprocessed PDF, call get_file_metadata ONCE — do NOT call read_file_content:
   gdrive__get_file_metadata(file_id=...)
   → Returns: id, viewUrl, lastModifyingUser, contentSnippet (partial PDF text)

   a. Apply DOI regex to contentSnippet:
      r'10\.\d{4,9}/[^\s\])]+'
      → use first match as DOI

   b. If no DOI found → extract candidate title from contentSnippet first 500 chars
      (first long line that is not a URL, author list, or journal name)
      → use as title for DB search fallback

   c. Set Drive fields from get_file_metadata response:
      → gdrive_file_id = file.id
      → gdrive_url = "https://drive.google.com/file/d/{file_id}/view"
      → added_by = file.lastModifyingUser.emailAddress (or "")

4. Fetch full metadata from DB:
   - DOI found → mcp__plugin_paper-vault_paper-search-mcp-openai-v2__get_crossref_paper_by_doi
     (most accurate — title, authors, journal, year)
   - Abstract missing from CrossRef result:
     a. Search contentSnippet (from get_file_metadata) for "Abstract" section (regex, no MCP cost)
     b. If found → use as abstract. Stop here.
     c. If still missing → read_semantic_paper ONCE for abstract only (last resort)
   - No DOI, has title → mcp__plugin_paper-vault_paper-search-mcp-openai-v2__search_semantic
     (returns abstract, affiliation, citations)

5. Normalize keywords + apply tags
   a. Extract keywords from DB result's keywords field (or top terms from abstract if empty)
   b. Normalize using references/keyword_normalization.md → store as `keywords` list
   c. Match normalized keywords + title against taxonomy → research_area_tags
   d. Match affiliation field against taxonomy → affiliation_tags
   - No match → empty array. Never force.

6. Create note → update hub indexes (파일 툴 직접 사용 — obsidian-mcp 쓰기 사용 금지)
   - Include Drive fields in frontmatter: pdf_source, gdrive_file_id, gdrive_url, added_by
   - Write 툴로 `{vault_mount}/Papers/Notes/[filename].md` 직접 작성
   - research area tag → Read/Edit 툴로 `{vault_mount}/Papers/Keywords/[Research Area].md` 업데이트
   - affiliation tag → Read/Edit 툴로 `{vault_mount}/Papers/Affiliation/[Company].md` 업데이트

7. Report summary
   - N new notes created
   - N skipped (already existed)
   - N failed (no metadata found)
```

### Mode 3: Browse by Category

When user asks "show me papers on Pregnancy Prediction" or "papers from Alife".

```
1. Resolve category against taxonomy:
   - Research Area (Pregnancy Prediction, Gardner Grade, …) → Read 툴로 `{vault_mount}/Papers/Keywords/[category].md`
   - Affiliation (KaiHealth, Alife, …) → Read 툴로 `{vault_mount}/Papers/Affiliation/[category].md`
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
- Mode 2 PDF 처리는 `get_file_metadata`의 `contentSnippet`으로 DOI 추출. `read_file_content` 호출 금지 (속도 저하).
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
