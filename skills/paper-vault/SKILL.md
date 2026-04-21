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
  version: 1.5.7
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

> **⚠️ 허브 파일 업데이트 규칙 — 반드시 준수:**
> - 허브 파일이 **이미 존재하면** → `Read` 툴로 읽은 후 `Edit` 툴로 수정 (paper_count +1, last_updated 갱신, `## Papers` 섹션에 링크 추가)
> - 허브 파일이 **없을 때만** → `Write` 툴로 신규 생성
> - **절대 금지:** `Write` 툴이나 Bash `cat >` 명령으로 기존 허브 파일을 덮어쓰는 행위 → 기존 논문 목록이 삭제됨
> - 존재 여부 확인 방법: `Glob("{vault_mount}/Papers/Keywords/*.md")` 또는 `Glob("{vault_mount}/Papers/Affiliation/*.md")` 로 미리 파일 목록을 취득한 후, 대상 파일명이 포함되어 있으면 Read+Edit, 없으면 Write

> **⚠️ obsidian-mcp 툴 동작 현황 (Cowork 환경):**
>
> | 툴 | 상태 | 사용 |
> |----|------|------|
> | `create-note` | ⚠️ 간헐적 타임아웃 | 노트/허브 신규 생성에 사용. 중복 시 에러로 안전하게 차단. 타임아웃 시 내장 `Write` 툴로 fallback |
> | `read-note` | ✅ 정상 | 노트 읽기에 사용 가능 |
> | `search-vault` | ✅ 정상 | 보조 검색에 사용 가능 |
> | `delete-note` | ✅ 정상 | 삭제에 사용 가능 |
> | `edit-note` | ❌ schema 버그 | `Invalid discriminator value` — 사용 금지 |
> | `list-available-vaults` | ❌ 타임아웃 | 사용 금지 |
>
> **작업별 툴 선택:**
> - **노트 신규 생성** → `mcp__obsidian-mcp__create-note` (vault, filename, folder, content); 타임아웃 시 내장 `Write` 툴로 `{vault_mount}/Papers/Notes/{filename}`에 직접 저장
> - **허브 신규 생성** → `mcp__obsidian-mcp__create-note`; 타임아웃 시 내장 `Write` 툴로 `{vault_mount}/Papers/Keywords/` 또는 `Papers/Affiliation/`에 직접 저장
> - **허브 업데이트 (기존 파일)** → 내장 `Read` 툴로 읽고 내장 `Edit` 툴로 수정 (`edit-note` 사용 금지)
> - **dedup 체크** → `Glob("{vault_mount}/Papers/Notes/*.md")` 또는 Bash `ls`로 파일명 비교 (`search-vault` 결과 부정확할 수 있음)
> - **볼트 경로** → `mcp__filesystem__list_allowed_directories`로 마운트 경로 취득. 하드코딩 금지.
>
> 절대 `mcp__filesystem__*` 툴은 사용하지 않는다.

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
1. Pre-fetch existing notes list (dedup용 — 검색 전 1회만):
   - Glob("{vault_mount}/Papers/Notes/*.md") 로 기존 노트 파일명 목록 취득
   - 이 목록을 이후 모든 dedup 비교에 재사용 (반복 Glob 금지)

2. Identify topic → choose ONE source. Do NOT search multiple sources.
   - Specific company papers → check references/known_authors.md first
     Use product name or author name, NOT company name directly
     e.g. "KaiHealth" → search "VITA EMBRYO" or "Hye Jun Lee embryo"
   - IVF/reproductive medicine → PubMed ONLY
   - AI/ML methods → arXiv ONLY
   - 0 results → one retry with Semantic Scholar (final attempt)

3. Search via paper-search MCP (maximum 2 calls total)
   mcp__plugin_paper-vault_paper-search-mcp-openai-v2__search_pubmed   (medical)
   mcp__plugin_paper-vault_paper-search-mcp-openai-v2__search_arxiv    (CS/AI)
   mcp__plugin_paper-vault_paper-search-mcp-openai-v2__search_semantic (general + affiliation)

4. Dedup check per result — 노트가 이미 있으면 즉시 skip:
   - 검색 결과의 DOI 또는 title keyword가 step 1 목록의 파일명에 포함되어 있으면
     → 해당 논문은 skip. step 5~7 일절 실행하지 않고 다음 결과로 넘어감
   - 이미 존재하는 노트는 메타데이터 조회, 태깅, 저장 어떤 것도 하지 않는다

5. Check search result completeness FIRST (skip DB calls if already complete):
   - Search result has title + authors + abstract + DOI → skip to step 6 immediately
   - DOI missing → get_crossref_paper_by_doi to retrieve DOI (then check abstract)
   - Abstract missing (but DOI present):
     a. read_semantic_paper ONCE for abstract only
     b. still missing → proceed with empty abstract. Stop here.
   - Never call CrossRef if the search result already contains title + authors + abstract + DOI.

6. Normalize keywords + apply tags
   a. Extract keywords from DB result's keywords field (or top terms from abstract if empty)
   b. Normalize using references/keyword_normalization.md → store as `keywords` list
   c. Match normalized keywords + title against taxonomy → research_area_tags
   d. Match affiliation field against taxonomy → affiliation_tags
   - No match → empty array. Never force.

7. Create note → update hub indexes
   - 노트 신규 생성 → mcp__obsidian-mcp__create-note (vault, filename, folder="Papers/Notes", content)
   - 허브 업데이트 전: Glob으로 Keywords/*.md, Affiliation/*.md 파일 목록을 먼저 취득
   - For EACH `research_area_tags` match:
     - Index exists (Glob 목록에 있음) → 내장 Read 툴로 읽고 내장 Edit 툴로 수정
       (paper_count +1, last_updated 갱신, ## Papers 섹션에 링크 추가)
     - Index missing (Glob 목록에 없음) → mcp__obsidian-mcp__create-note로 신규 생성
       (folder="Papers/Keywords")
     - ⛔ 기존 파일을 Write/cat >로 덮어쓰지 말 것 — 기존 논문 목록 삭제됨
   - For EACH `affiliation_tags` match:
     - Index exists (Glob 목록에 있음) → 내장 Read 툴로 읽고 내장 Edit 툴로 수정
       (paper_count +1, last_updated 갱신, ## Papers 섹션에 링크 추가)
     - Index missing (Glob 목록에 없음) → mcp__obsidian-mcp__create-note로 신규 생성
       (folder="Papers/Affiliation")
     - ⛔ 기존 파일을 Write/cat >로 덮어쓰지 말 것 — 기존 논문 목록 삭제됨
   - Examples:
     - research/pregnancy-prediction → folder="Papers/Keywords", filename="Pregnancy Prediction.md"
     - affiliation/alife → folder="Papers/Affiliation", filename="Alife.md"
     - affiliation/kaihealth → folder="Papers/Affiliation", filename="KaiHealth.md"

8. Report summary to user
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

6. Create note → update hub indexes
   - Include Drive fields in frontmatter: pdf_source, gdrive_file_id, gdrive_url, added_by
   - 노트 신규 생성 → mcp__obsidian-mcp__create-note (vault, filename, folder="Papers/Notes", content)
   - 허브 업데이트 전: Glob으로 Keywords/*.md, Affiliation/*.md 파일 목록을 먼저 취득
   - research area tag → 파일 존재 여부 확인:
     - 존재 (Glob 목록에 있음) → 내장 Read 툴로 읽고 내장 Edit 툴로 수정
       (paper_count +1, last_updated 갱신, ## Papers 섹션에 링크 추가)
     - 없음 (Glob 목록에 없음) → mcp__obsidian-mcp__create-note로 신규 생성
       (folder="Papers/Keywords")
     - ⛔ 기존 파일을 Write/cat >로 덮어쓰지 말 것 — 기존 논문 목록 삭제됨
   - affiliation tag → 파일 존재 여부 확인:
     - 존재 (Glob 목록에 있음) → 내장 Read 툴로 읽고 내장 Edit 툴로 수정
       (paper_count +1, last_updated 갱신, ## Papers 섹션에 링크 추가)
     - 없음 (Glob 목록에 없음) → mcp__obsidian-mcp__create-note로 신규 생성
       (folder="Papers/Affiliation")
     - ⛔ 기존 파일을 Write/cat >로 덮어쓰지 말 것 — 기존 논문 목록 삭제됨

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
