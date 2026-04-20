# CLAUDE.md

This is the **Paper Vault** plugin — a Claude plugin that searches IVF/reproductive medicine papers via MCP, extracts structured metadata, and generates linked Obsidian markdown notes with taxonomy-based tagging.

## Repo Structure

```
paper-vault/
├── .claude-plugin/
│   └── plugin.json                   # Plugin manifest (name, version, author)
├── skills/
│   └── paper-vault/
│       ├── SKILL.md                  # Main skill file
│       └── references/
│           ├── taxonomy.md           # Fixed tagging classification (research area + affiliation)
│           ├── keyword_normalization.md  # Keyword normalization map
│           └── known_authors.md      # Company → known authors/products for search
├── CHANGELOG.md                      # Version history
├── CLAUDE.md                         # This file
└── README.md                         # GitHub-facing docs
```

## Installation

```bash
# Session-only (for testing)
claude --plugin-dir /path/to/paper-vault

# Permanent (via marketplace — repo is its own marketplace)
/plugin marketplace add nealzsy/paper-vault
/plugin install paper-vault@paper-vault-marketplace
```

During installation, Claude Code will prompt for:
- `vault_papers_path` — Obsidian vault root (e.g. `/Users/yourname/Documents/Obsidian Vault`)

This value is stored in `.claude/settings.json` and injected into both the obsidian MCP and filesystem MCP args via `${user_config.vault_papers_path}`. No manual MCP reconfiguration needed.

## Key Decisions

- **paper-search MCP for retrieval** — searches PubMed, arXiv, Semantic Scholar via DOI or title
- **Google Drive MCP for PDF source** — Mode 2 reads PDFs directly from Google Drive via `mcp__gdrive__*`; no Google Drive Desktop or "available offline" setup required
- **PDF text extraction via Drive MCP** — `mcp__gdrive__read_file_content` returns extracted PDF text; DOI extracted with regex (`10.\d{4,9}/[^\s]+`); pymupdf no longer used
- **Drive metadata in notes** — each Mode 2 note includes `gdrive_file_id`, `gdrive_url`, `added_by` (last modifier email) in frontmatter for direct PDF access from Obsidian
- **Notes always written to Obsidian vault** — .md notes go to `{vault_papers_path}/Papers/Notes/` regardless of Drive folder
- **obsidian MCP for vault reads/writes** — all note create/edit/search/list operations go through `mcp__obsidian__*`
- **filesystem MCP removed** — no longer needed; Drive PDF scanning is handled by `mcp__gdrive__search_files`
- **Nested Obsidian tags** — research area tags use `research/[slug]`, affiliation tags use `affiliation/[slug]`; enables category filtering in Tags panel and Knowledge Graph
- **Fixed taxonomy tagging** — tags matched only against `references/taxonomy.md`; never created freely
- **claude-haiku-4-5-20251001 for analysis** — optimized for speed/cost (swap to `claude-sonnet-4-6` for higher accuracy)
- **Papers/Notes + hub indexes** — notes in `Papers/Notes/`; research-area hubs in `Papers/Keywords/`; affiliation hubs in `Papers/Affiliation/`
- **Keyword normalization** — raw keywords normalized via `references/keyword_normalization.md` before tagging
- **Fail-safe fallback** — if JSON parsing fails, filename is used as title; never hard stops

## Working on This Repo

When editing `skills/paper-vault/SKILL.md`:
- Keep under **5,000 words** (Anthropic best practice for progressive disclosure)
- Move detailed specs to `skills/paper-vault/references/` and link from SKILL.md
- YAML frontmatter must have `name` and `description` with trigger phrases
- Update `CHANGELOG.md` with a summary of changes under the current version

When editing text extraction logic in the skill:
- Python/pymupdf is **no longer used** — PDF text extraction is handled by `mcp__gdrive__read_file_content`
- DOI regex (`r'10\.\d{4,9}/[^\s\]).]+'`) is applied to the text returned by the Drive MCP
- Model name changes must be reflected in the Notes section of SKILL.md as well
- `sanitize_filename()` changes must be synced with filename rules in SKILL.md
- Update `CHANGELOG.md` with the change

When editing `references/taxonomy.md`:
- New tags require explicit user approval before adding
- Affiliation match strings are case-insensitive substring matches
- Sync tag list changes to the taxonomy table in SKILL.md
- Update `CHANGELOG.md` with added/removed tags

When editing `plugin.json`:
- Version bump required for any functional change
- Update `CHANGELOG.md` with the change
- Reflect any MCP or userConfig changes in README.md and CLAUDE.md
- `gdrive` MCP server uses `@modelcontextprotocol/server-gdrive` — requires Google OAuth credentials on first run
- Google Drive 대상 폴더는 `0AAhracftG0XwUk9PVA`로 하드코딩 (userConfig 아님) — 변경 시 SKILL.md Mode 2 step 1과 이 파일을 함께 수정

When making any change across files:
- Update `CHANGELOG.md` under the current version
- Check if README.md, CLAUDE.md, or SKILL.md need to reflect the same change
- Keep all files consistent — same version number, same terminology

## Common Pitfalls

- `claude-haiku` JSON responses may include markdown code fences (` ``` `) — strip before parsing
- Obsidian wikilinks use filename without extension — must match `sanitize_filename()` output exactly
- `max_tokens=4096` for relation analysis may be insufficient for 30+ papers — split into batches of 20
- `vault_papers_path` (userConfig) must point to the Obsidian vault root (e.g. `~/Documents/Obsidian Vault`), not the `Papers/` subfolder — the skill uses `Papers/Notes/`, `Papers/Keywords/`, and `Papers/Affiliation/` under that root
- Google Drive 폴더 ID는 SKILL.md에 하드코딩(`0AAhracftG0XwUk9PVA`) — Shared Drive ID가 아니라 폴더 ID여야 함 (URL: `drive.google.com/drive/folders/{FOLDER_ID}`)
- `mcp__gdrive__read_file_content` may return empty text for scanned (image-only) PDFs — in that case fall back to `get_file_metadata` name for title and proceed with DOI lookup by title
- `added_by` reflects last modifier, not original uploader — document this limitation in notes if relevant
- **`mcp__obsidian__edit-note` schema bug** — in some environments this tool's parameter schema is exported as empty (`"properties": {}`), causing all calls to fail with "Invalid discriminator value". Fallback: read the hub note with `read-note`, modify content in memory, then write directly to `{vault_papers_path}/Papers/Keywords/[file].md` or `{vault_papers_path}/Papers/Affiliation/[file].md` using the built-in Write/Edit file tool. Do NOT use `mcp__filesystem__*` as a fallback — it has path restrictions and is not registered in this plugin.
