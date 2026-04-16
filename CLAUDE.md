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

# Permanent (via org marketplace)
/plugin marketplace add your-org/plugin-catalog
```

During installation, Claude Code will prompt for:
- `vault_papers_path` — Obsidian vault root (e.g. `/Users/yourname/Documents/Obsidian Vault`)

This value is stored in `.claude/settings.json` and injected into both the obsidian MCP and filesystem MCP args via `${user_config.vault_papers_path}`. No manual MCP reconfiguration needed.

## Key Decisions

- **paper-search MCP for retrieval** — searches PubMed, arXiv, Semantic Scholar via DOI or title
- **PDF metadata extraction** — reads only PDF metadata dict + page 1 (fallback) to extract DOI/title; never reads full document
- **PDF source is flexible** — PDFs can be local or Google Drive Desktop sync folder (`~/Library/CloudStorage/GoogleDrive-.../Shared drives/...`); Google Drive files must be set to "available offline" in Finder
- **Notes always written to Obsidian vault** — regardless of PDF source, .md notes go to `{vault_papers_path}/Papers/Notes/`
- **obsidian MCP for vault reads/writes** — all note create/edit/search/list operations go through `mcp__obsidian__*`; filesystem MCP is kept only for scanning PDF source folders in Mode 2
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

When editing Python code in the skill:
- Python (pymupdf) is used only for reading PDF metadata — vault writes go through the obsidian MCP, not Python
- Never hardcode vault paths in Python — vault path is handled by `userConfig.vault_papers_path` injected into filesystem MCP args
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

When making any change across files:
- Update `CHANGELOG.md` under the current version
- Check if README.md, CLAUDE.md, or SKILL.md need to reflect the same change
- Keep all files consistent — same version number, same terminology

## Common Pitfalls

- `claude-haiku` JSON responses may include markdown code fences (` ``` `) — strip before parsing
- Obsidian wikilinks use filename without extension — must match `sanitize_filename()` output exactly
- `max_tokens=4096` for relation analysis may be insufficient for 30+ papers — split into batches of 20
- `vault_papers_path` (userConfig) must point to the Obsidian vault root (e.g. `~/Documents/Obsidian Vault`), not the `Papers/` subfolder — the skill uses `Papers/Notes/`, `Papers/Keywords/`, and `Papers/Affiliation/` under that root
