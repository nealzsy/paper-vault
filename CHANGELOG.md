# Changelog

## [1.4.1] - 2026-04-20

### Added
- **`.claude-plugin/marketplace.json` 추가** — repo를 조직 내부 플러그인 마켓플레이스로 쓸 수 있도록 manifest 추가. `/plugin marketplace add nealzsy/paper-vault` 후 `/plugin install paper-vault@paper-vault-marketplace`로 설치 가능. `source: "."`로 자기 자신을 플러그인으로 등재.

### Fixed
- **`edit-note` fallback 명시** — obsidian MCP의 `edit-note` 툴이 일부 환경에서 스키마 오류로 실패하는 버그 대응. SKILL.md hub 인덱스 업데이트 섹션 및 CLAUDE.md Common Pitfalls에 fallback 절차 추가: `read-note`로 현재 내용 읽기 → 메모리에서 수정 → 빌트인 Write/Edit 파일 도구로 vault 마운트 경로에 직접 덮어쓰기. `mcp__filesystem__*`는 경로 제한으로 사용 불가 — 명시적으로 금지.
- **`is_multicenter` 판단 기준 보완** — 기존 키워드("multi-center", "multicenter" 등) 외에 "multiple clinics" 및 "2개 이상의 named 클리닉/병원에서 데이터 수집" 패턴도 `true`로 판단하도록 수정. ESHRE abstract처럼 "seven in vitro fertilization clinics" 표현을 감지하지 못하던 문제 수정.
- **파일명 길이 100 → 200자로 확장** — 긴 논문 제목이 의미 있는 단어에서 잘리는 문제 수정.

---

## [1.4.0] - 2026-04-20

### Changed
- **Mode 2: Google Drive MCP 직접 읽기로 전환** — 기존 filesystem MCP + pymupdf 방식 제거. `mcp__gdrive__search_files`로 PDF 목록 조회, `mcp__gdrive__read_file_content`로 텍스트 추출, `mcp__gdrive__get_file_metadata`로 파일 ID·업로더 수집. Google Drive Desktop 설치 및 "오프라인 사용 설정" 불필요.
- **노트 frontmatter에 Drive 필드 추가** — Mode 2 경유 노트에 `pdf_source: gdrive`, `gdrive_file_id`, `gdrive_url`, `added_by` 필드 포함. Obsidian에서 원본 PDF로 1-click 이동 가능.
- **plugin.json: `gdrive` MCP 서버 추가** — `@modelcontextprotocol/server-gdrive` 사용; `filesystem` MCP 제거.
- **plugin.json: `gdrive_shared_folder_id` userConfig 추가** — 설치 시 Google Drive 폴더 ID 입력; Mode 2 자동 사용.
- **SKILL.md**: Mode 2 워크플로우 전면 재작성, Notes 섹션 보강, 버전 1.4.0.
- **CLAUDE.md**: Key Decisions 업데이트 (pymupdf 제거, Drive MCP 추가), Common Pitfalls 보강.

### Added
- **Mode 2 수량 제한** — 사용자가 "N편만" 같이 수량을 명시하면 새 노트 N개 생성 후 즉시 중단. 스킵(이미 존재)된 노트는 카운트 미포함. 수량 미명시 시 폴더 전체 처리.

### Removed
- **pymupdf 의존성 제거** — PDF 텍스트 추출을 Drive MCP가 대체하므로 Python/pymupdf 설치 불필요.
- **filesystem MCP 제거** — PDF 스캔 역할을 `mcp__gdrive__search_files`가 대체.
- **`gdrive_shared_folder_id` userConfig 제거** — Drive 폴더 ID를 `0AAhracftG0XwUk9PVA`로 SKILL.md에 하드코딩; 설치 시 입력 불필요.

---

## [1.3.0] - 2026-04-16

### Changed
- **Affiliation hubs moved to `Papers/Affiliation/`** — research-area index notes stay in `Papers/Keywords/`; each matched affiliation tag gets/updates `Papers/Affiliation/[Company].md` (Mode 1 & 2 step 6). Mode 3 resolves Research Area vs Affiliation from taxonomy and reads the correct hub path.
- **SKILL.md** — vault tree, hub index templates (keyword vs affiliation), result report examples, and `version` → 1.3.0
- **README + CLAUDE** — vault layout documents `Papers/Affiliation/`

Existing vaults: affiliation hub `.md` files previously under `Papers/Keywords/` are not moved automatically; copy or move them to `Papers/Affiliation/` (same filenames) to avoid duplicate hubs.

---

## [1.2.2] - 2026-04-16

### Fixed
- **obsidian MCP package name corrected** — `mcp-obsidian` → `obsidian-mcp` in both `plugin.json` and README manual config section
- **Mode 2 Google Drive scope restricted** — only `Shared drives/Paper/` path allowed; Claude now re-confirms with user if path does not contain `Shared drives/Paper`
- **Affiliation keyword index now explicit** — Mode 1 and Mode 2 step 6 updated to clarify that BOTH `research_area_tags` AND `affiliation_tags` generate/update `Papers/Keywords/[tag].md` index files

### Documentation
- **README Cowork manual MCP snippet** — example server key set to `obsidian-mcp` (matches typical `mcpServers` entry); vault path placeholder and Korean copy updated for Claude Desktop hand configuration

---

## [1.2.1] - 2026-04-16

### Performance
- **Mode 1 redundant CrossRef call eliminated** — Step 4 now checks search result completeness first; if title + authors + abstract + DOI are all present in the search result, CrossRef call is skipped entirely and flow jumps straight to tagging (step 5). Reduces MCP round-trips by 1–2 calls in the common case.
- **Mode 2 abstract fallback order fixed** — When CrossRef returns no abstract, PDF metadata['subject'] is now checked first (local Bash, zero MCP cost) before falling back to `read_semantic_paper`. Avoids a Semantic Scholar call when the PDF itself contains the abstract in its metadata.

---

## [1.2.0] - 2026-04-16

### Fixed
- **plugin.json obsidian MCP package name corrected** — `obsidian-mcp` → `mcp-obsidian` (실제 동작하는 npm 패키지명으로 수정)
- **SKILL.md obsidian tool names corrected** — underscore → hyphen format to match actual MCP tool names; `search_notes` → `search-vault`; `list_notes` (non-existent) replaced with `search-vault` query
- **SKILL.md version synced** — 1.1.0 → 1.2.0
- **⭐ own company marker removed** — KaiHealth ⭐ annotation removed from SKILL.md, taxonomy.md, known_authors.md for public distribution

### Changed
- **obsidian MCP added** — vault read/write operations now use `mcp__obsidian__*` instead of `mcp__filesystem__*`
  - Note create: `mcp__obsidian__create_note`
  - Note read/update: `mcp__obsidian__read_note`, `mcp__obsidian__edit_note`
  - Dedup search: `mcp__obsidian__search_notes(query=doi)`
  - Note listing: `mcp__obsidian__list_notes("Papers/Notes")`
- **filesystem MCP demoted to PDF-only** — now `required: false`; used only for scanning PDF source folders in Mode 2
- **plugin.json version bump** — 1.1.0 → 1.2.0

---

## [1.1.0] - 2026-04-15

### Added
- **Mode 2: PDF metadata extraction** — reads PDF metadata dict first (zero token cost), falls back to page 1 only if needed; never reads full document
- **Google Drive Desktop support** — PDF source can be a Google Drive Desktop sync folder; notes always written to Obsidian vault regardless of PDF source
- **Dedup check in Mode 2** — skips PDFs that already have a matching note (DOI-based match); only processes new papers
- **Nested Obsidian tags** — research area tags use `research/[slug]`, affiliation tags use `affiliation/[slug]` for category filtering in Tags panel and Knowledge Graph
- **`userConfig.vault_papers_path`** — plugin prompts for vault path on install; default `~/Documents/Obsidian Vault`; injected into filesystem MCP args automatically
- **`trust: true`** on both MCP servers — suppresses per-call approval prompts after install
- **`paper-search-mcp-openai-v2` auto-install** — added `command`/`args` to plugin.json; MCP now configured automatically on plugin install via `mcp-remote` + Smithery hosted server; no manual setup needed

### Changed
- **Mode 1 search strategy** — "choose ONE source and stick to it"; company papers → Semantic Scholar ONLY, medical → PubMed ONLY, AI/ML → arXiv ONLY; one retry max on 0 results
- **Mode 1 strict rules added** — max 2 search calls per request; save exactly N papers as requested; forbidden tools list (fetch, WebFetch, ORCID, PDF download); affiliation unconfirmed → proceed with empty array instead of blocking
- **Mode 1 dedup check** — search existing notes by DOI before saving; skip papers already in vault (same logic as Mode 2)
- **Mode 1 metadata fallback order fixed** — CrossRef → Semantic Scholar (abstract only, once) → proceed with empty; no further attempts
- **`references/known_authors.md`** — new reference file mapping company names to known author names and product names; used by Mode 1 to search by product/author instead of company name (KaiHealth → VITA EMBRYO / Hye Jun Lee)
- **`Time-lapse` research area tag** — added to taxonomy.md and SKILL.md; match keywords: time-lapse, TLI, TLM, morphokinetics, morphokinetic, time-lapse microscopy
- **Filename length** — increased from 50 to 100 chars, truncated at word boundary (never mid-word)
- **Mode 2 renamed** — "Tag Existing PDFs" → "Tag PDF Files (Local or Google Drive)"
- **PDF source strategy** — replaced filename-based DB search with DOI extraction from metadata/page 1 → CrossRef or Semantic Scholar lookup
- **filesystem MCP config** — now includes `command`, `args`, and `${user_config.vault_papers_path}` in plugin.json
- **`env.VAULT_PAPERS_PATH`** replaced by `userConfig.vault_papers_path` in plugin.json
- **Abstract sourcing** — CrossRef abstract supplemented by PDF metadata subject field or page 1 when missing
- **Keyword index files** — all content now in English (previously mixed Korean/English)
- **README Prerequisites** — added ordered setup section with Node.js v18+, pymupdf, Google Drive Desktop instructions and tested versions
- **README How to Use** — added prompt guide with good/bad examples, company paper search workaround (product name / author name), Mode 1/2/3 example prompts
- **README Usage (Cowork)** — added warning: must mount both paper-vault AND Obsidian Vault at session start; cannot add folders mid-session

### Fixed
- Install command corrected: `claude --plugin-dir /path` (not `claude plugin install --plugin-dir`)
- `affiliation_tags` regex bug in tag update script — nested tag replacement now uses exact `\ntags:` pattern

## [1.0.2] - 2026-04-13

### Initial release
- Mode 1: Search & Save Papers (PubMed, arXiv, Semantic Scholar)
- Mode 2: Tag Existing PDFs (filename-based search)
- Mode 3: Browse by Category
- Fixed taxonomy tagging (research area + affiliation)
- Obsidian Knowledge Graph via keyword index files
