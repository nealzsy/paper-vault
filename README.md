# Paper Vault

A Claude Code plugin that turns IVF/reproductive medicine PDFs and online searches into a linked Obsidian knowledge base — automatically.

## What It Does

**Mode 1 — Search & Save:** Ask Claude to find papers online → notes created in your vault.
```
"Find 5 latest papers on IVF embryo grading"
→ Searches PubMed / arXiv / Semantic Scholar
→ Extracts title, authors, abstract, performance metrics
→ Auto-tags with fixed taxonomy
→ Creates .md notes in Obsidian with Knowledge Graph links
```

**Mode 2 — Tag Local or Google Drive PDFs:** Point at a folder of PDFs → notes created for each.
```
"Create notes for all PDFs in my Google Drive Paper folder"
→ Extracts DOI from PDF metadata (zero token cost)
→ Falls back to page 1 text if metadata missing
→ Looks up full metadata via CrossRef or Semantic Scholar
→ Skips PDFs that already have a note (dedup by DOI)
→ Writes .md notes to Obsidian vault
```

**Mode 3 — Browse:** Ask Claude to show papers by category.

## How to Use

### Mode 1 — 논문 검색 & 저장

**핵심 규칙: 찾고 싶은 논문 수를 반드시 명시하세요.**

```
✅ 좋은 예시
"embryo grading AI 논문 3편 찾아와"
"Pregnancy Prediction 최신 논문 5편 저장해줘"
"VITA EMBRYO 논문 1편 찾아와"
"Hye Jun Lee IVF AI 논문 2편 찾아와"

❌ 피해야 할 예시
"KaiHealth 논문 찾아와"     ← 회사명은 DB에 affiliation으로 인덱싱 안 됨
"논문 찾아줘"               ← 수량 미지정 시 결과 수 예측 불가
```

**회사 논문을 찾을 때:**

| 방법 | 예시 |
|------|------|
| 제품명으로 | `"VITA EMBRYO 논문 1편 찾아와"` |
| 저자명으로 | `"Hye Jun Lee embryo AI 논문 찾아와"` |
| 주제로 | `"KaiHealth 스타일 embryo ranking AI 논문 3편"` |

회사명 직접 검색(`"KaiHealth 논문"`)은 Semantic Scholar에 affiliation이 정확히 인덱싱되지 않아 결과가 없거나 부정확할 수 있어요.

---

### Mode 2 — PDF → 노트 변환

```
"이 폴더에 있는 PDF 노트로 만들어줘: /Users/yourname/Documents/Papers/"
"Google Drive Paper 폴더 PDF 처리해줘: ~/Library/CloudStorage/GoogleDrive-.../Shared drives/Paper/"
```

---

### Mode 3 — 카테고리별 조회

```
"Pregnancy Prediction 논문 목록 보여줘"
"Gardner Grade 관련 논문 뭐 있어?"
```

---

## Output Format

```markdown
---
title: "Automated Blastocyst Grading Using Deep Learning"
authors:
  - "Kim J"
  - "Lee S"
year: 2023
journal: "Human Reproduction"
doi: "10.xxxx/xxxxx"
performance_metrics: "AUC: 0.91, sensitivity: 87%"
is_multicenter: false
research_area:
  - "Gardner Grade"
affiliation_tags:
  - "Alife"
tags:
  - paper
  - research/gardner-grade
  - affiliation/alife
---

## Abstract
...

## Tags
**Research Area:** [[Gardner Grade]]
**Affiliation:** [[Alife]]
```

Tags use Obsidian nested format (`research/`, `affiliation/`) for filtering in the Tags panel and Knowledge Graph.

## Prerequisites

Complete these steps before installing the plugin.

### 1. Obsidian

Install [Obsidian](https://obsidian.md) and create a vault. Note the vault root path — you'll enter it during plugin setup.

Example vault paths:
```
~/Documents/Obsidian Vault
/Users/yourname/Documents/My Vault
```

### 2. Node.js v18+ (required)

Both MCP servers run via `npx` (bundled with Node.js). Install Node.js v18 or later:
```bash
# Check version
node --version   # should be v18.x or higher

# Install via Homebrew (macOS)
brew install node
```

> Tested with: Node.js v22.5.1

### 3. Python dependency (for Mode 2 — PDF processing)

```bash
pip install pymupdf
```

Required for extracting DOI and title from PDF metadata. Must be installed manually. Skip if you only use Mode 1 (online search).

> Tested with: Python 3.12.2, pymupdf 1.27.2.2

### 4. Google Drive Desktop (optional — only if PDFs are in Google Drive)

If your PDFs are stored in Google Drive:

1. Install [Google Drive for Desktop](https://www.google.com/drive/download/)
2. Sign in and let it sync
3. In **Finder → Google Drive → Shared drives**, right-click the folder containing your PDFs → **"Make available offline"**
   - ⚠️ This step is required. Without it, PDFs appear as cloud stubs (0 bytes) and cannot be read.
4. Wait for sync to complete before running the skill

Your PDF folder path will look like:
```
~/Library/CloudStorage/GoogleDrive-[your@email.com]/Shared drives/[Drive Name]/
```

---

## Usage (Cowork)

새 채팅을 시작할 때 **반드시 두 폴더를 같이 선택**하세요:

| 폴더 | 역할 |
|------|------|
| `paper-vault` | 플러그인 로드 |
| `Obsidian Vault` | 노트 저장 위치 |

⚠️ Obsidian Vault 폴더 없이 시작하면 노트가 잘못된 위치에 저장됩니다. 세션이 시작된 후에는 폴더를 추가할 수 없으므로 반드시 처음에 같이 선택하세요.

---

## Installation

```bash
# Session-only (testing)
claude --plugin-dir /path/to/paper-vault

# Permanent (via org marketplace)
/plugin marketplace add your-org/plugin-catalog
```

During install, Claude will prompt:
```
? vault_papers_path (~/Documents/Obsidian Vault):
```
Enter the Obsidian vault root path from step 1. Press Enter to use the default.

**paper-search MCP 인증:** 처음 실행 시 `mcp-remote`가 브라우저에서 Smithery 로그인을 요청해요. 한 번 로그인하면 이후 자동으로 인증돼요.

---

## MCP Servers (auto-configured)

All three MCP servers are configured automatically via `plugin.json` when you install the plugin — `vault_papers_path` is injected from your user config, no manual path editing needed.

| MCP | Purpose |
|-----|---------|
| `paper-search-mcp-openai-v2` | Search PubMed, arXiv, Semantic Scholar (via Smithery) |
| `obsidian` | Create, edit, search notes in your Obsidian vault |
| `filesystem` | Scan PDF source folders (Mode 2 only) |

### Cowork 모드 사용 시 (수동 설정 필요)

Cowork 모드는 `plugin.json`의 MCP 자동 주입을 아직 지원하지 않아요. 아래 설정을 `~/Library/Application Support/Claude/claude_desktop_config.json`의 `mcpServers`에 직접 추가하고 앱을 재시작하세요:

```json
"obsidian-mcp": {
  "command": "npx",
  "args": ["-y", "obsidian-mcp", "obsidian vault path"]
}
```

`obsidian vault path` 부분을 해당 플러그인의 결과가 저장될 obsidian vault 경로로 교체하세요.


## PDF Source

PDFs can be local or synced via Google Drive Desktop (see Prerequisites above).
Notes are always written to `{vault_papers_path}/Papers/Notes/` regardless of PDF source.

## Vault Structure

```
{vault_papers_path}/
└── Papers/
    ├── Notes/      ← one .md per paper
    └── Keywords/   ← category index hub notes
```

## Tagging Taxonomy

Tags are matched against a fixed list — never created freely.

**Research Area:** `Pregnancy Prediction`, `Gardner Grade`, `PGT Result Prediction`, `Oocyte`, `Sperm`, `Time-lapse`

**Affiliation:** KaiHealth, Alife, Presagen, AiVF, Fairtility, Vitrolife, Future Fertility, IVF 2.0, Embryonics, FertilAI, IMVITRO, MIM Fertility, GENESYS AI LABS

## Project Structure

```
paper-vault/
├── .claude-plugin/
│   └── plugin.json                   # Plugin manifest
├── skills/
│   └── paper-vault/
│       ├── SKILL.md                  # Core skill instructions
│       └── references/
│           ├── taxonomy.md           # Tagging classification
│           ├── keyword_normalization.md
│           └── known_authors.md      # Company → known authors/products
├── CHANGELOG.md
├── CLAUDE.md                         # Developer context
└── README.md                         # This file
```
