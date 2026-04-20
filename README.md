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

**Mode 2 — Tag Google Drive PDFs:** Point at a Shared Drive folder → notes created for each PDF directly from the cloud.
```
"Google Drive Paper 폴더 PDF 노트로 만들어줘"
→ Google Drive MCP로 PDF 목록 스캔 (Drive Desktop 불필요)
→ PDF 전문 텍스트 추출 → DOI 추출
→ CrossRef 또는 Semantic Scholar로 전체 메타데이터 조회
→ 이미 노트 있는 PDF는 스킵 (DOI 중복 체크)
→ Obsidian vault에 .md 노트 저장 (Drive 링크 포함)
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

### Mode 2 — Google Drive PDF → 노트 변환

```
"Google Drive Paper 폴더 PDF 노트로 만들어줘"
"Drive에 있는 PDF 전부 Obsidian에 추가해줘"
```

Drive 폴더는 설치 시 입력한 `gdrive_shared_folder_id`가 자동으로 사용됩니다. 별도로 경로를 지정하지 않아도 돼요.

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
date_added: "2026-04-20"
pdf_source: "gdrive"
gdrive_file_id: "1abc..."
gdrive_url: "https://drive.google.com/file/d/1abc.../view"
added_by: "user@example.com"
---

## Abstract
...

## Tags
**Research Area:** [[Gardner Grade]]
**Affiliation:** [[Alife]]

## Links
- [DOI](https://doi.org/10.xxxx/xxxxx)
- [Google Drive](https://drive.google.com/file/d/1abc.../view)
```

Tags use Obsidian nested format (`research/`, `affiliation/`) for filtering in the Tags panel and Knowledge Graph. `pdf_source`, `gdrive_file_id`, `gdrive_url`, `added_by` fields are added for Mode 2 (Drive import) notes only.

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

### 3. Google Drive MCP OAuth (Mode 2 사용 시 필수)

Mode 2(Drive PDF 임포트)를 사용하려면 Google Drive MCP에 OAuth 인증이 필요합니다.

1. 플러그인 설치 후 처음 Mode 2를 실행하면 Claude가 Google OAuth 인증을 요청합니다.
2. 브라우저에서 Google 계정으로 로그인 → 권한 허용
3. 이후 세션에서는 자동 인증됩니다.

Google Drive Desktop 설치나 "오프라인 사용 설정"은 **불필요**합니다 — Drive MCP가 클라우드에서 직접 읽습니다.

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

# Permanent (via marketplace)
/plugin marketplace add nealzsy/paper-vault
/plugin install paper-vault@paper-vault-marketplace
```

During install, Claude will prompt:
```
? vault_papers_path (~/Documents/Obsidian Vault):
→ Obsidian 볼트 루트 경로 입력 (예: ~/Documents/Obsidian Vault)
```

Google Drive 폴더는 `0AAhracftG0XwUk9PVA`로 고정되어 있어 별도 입력이 필요 없습니다.

**paper-search MCP 인증:** 처음 실행 시 `mcp-remote`가 브라우저에서 Smithery 로그인을 요청해요. 한 번 로그인하면 이후 자동으로 인증돼요.

**Google Drive MCP 인증:** Mode 2 첫 실행 시 Google OAuth 인증 창이 열립니다. Google 계정으로 로그인하고 Drive 접근 권한을 허용하면 이후 자동 인증됩니다.

---

## MCP Servers (auto-configured)

All three MCP servers are configured automatically via `plugin.json` when you install the plugin — `vault_papers_path`와 `gdrive_shared_folder_id`는 설치 시 입력한 값이 자동으로 주입됩니다.

| MCP | Purpose |
|-----|---------|
| `paper-search-mcp-openai-v2` | Search PubMed, arXiv, Semantic Scholar (via Smithery) |
| `obsidian` | Create, edit, search notes in your Obsidian vault |
| `gdrive` | Read PDF files and metadata directly from Google Drive (Mode 2) |

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

PDFs are read **directly from Google Drive** via the `gdrive` MCP — no local sync or Desktop app required.
Folder to scan is set once at install time (`gdrive_shared_folder_id`).
Notes are always written to `{vault_papers_path}/Papers/Notes/` in your Obsidian vault.

## Vault Structure

```
{vault_papers_path}/
└── Papers/
    ├── Notes/         ← one .md per paper
    ├── Keywords/      ← research-area hub (one .md per category)
    └── Affiliation/   ← affiliation hub (one .md per company tag)
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
