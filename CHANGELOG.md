# Changelog

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
