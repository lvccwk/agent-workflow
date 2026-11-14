# Agent Workflow — OpenCode CR Agent

Change Request（CR）工程工作流程嘅文件同筆記集合，針對 **Java / SQL Server Stored Procedure / Insurance Agency Reporting** 嘅變更工作。

設計原則：AI Agent 提供**可審核嘅成果**；業務需求批准、環境操作、UAT 簽收及正式部署，仍然由獲授權人員負責。

---

## 📁 目錄結構

```
.
├── docs/         規則、指引同專案結構定義
├── guides/       01–07 全流程操作手冊（Markdown + Word）
├── prompts/      可直接複製使用嘅 Prompt
├── notes/        筆記、改善建議、會議紀錄
└── README.md
```

---

## 📚 文件索引

### docs/ — 規則與指引

| 檔案 | 內容 |
|---|---|
| [AGENTS.md](docs/AGENTS.md) | **核心規則**：Agent mission、資料夾契約、安全規則、強制工作流程與停機點、品質閘門、回應格式 |
| [CR-Development-Instructions.md](docs/CR-Development-Instructions.md) | 精簡版開發指引：資料夾規則 + 六步工作流程 |
| [CR-001-structure.md](docs/CR-001-structure.md) | CR-001 標準資料夾結構示例（CR / DATABASE / OLD / SOLUTION / TEST / REPORT） |

### guides/ — 完整操作手冊

| 檔案 | 內容 |
|---|---|
| [OpenCode_CR_Agent_01-07.md](guides/OpenCode_CR_Agent_01-07.md) | **01–07 全流程手冊**：逐階段目的、操作、Prompt、交付物、停機條件；附錄含完整 AGENTS.md |
| [OpenCode_CR_Agent_01-07.docx](guides/OpenCode_CR_Agent_01-07.docx) | 同上手冊嘅 Word 版本（方便派發／列印） |

**階段對照：**

| 步驟 | 階段 | 核心輸出 |
|---|---|---|
| 01 | Requirement Intake（Phase 0） | Requirement Register |
| 02 | As-Is & Impact（Phase 1 + 2） | As-Is / Dependencies / Impact |
| 03 | To-Be & Plan（Gate A） | Implementation Plan（等批准） |
| 04 | Build | SOLUTION/ 修改 + Diff |
| 05 | Independent Verification | Code Review Report |
| 06 | UAT Preparation & Evidence | Test Cases + 實際執行證據 |
| 07 | Release Readiness | Release 評審 |

### prompts/ — 可直接使用嘅 Prompt

| 檔案 | 用途 |
|---|---|
| [START_CR_PROMPT.txt](prompts/START_CR_PROMPT.txt) | 開始或恢復一個 CR（按當前階段繼續，唔准跳閘） |
| [phase5-independent-verification.md](prompts/phase5-independent-verification.md) | Phase 5 獨立驗證：Prompt A（Code Review）+ Prompt B（Test Case 產生） |
| [session-review.md](prompts/session-review.md) | Session 交接：產生 `REPORT/session_handover.md` |

### notes/ — 筆記與改善建議

| 檔案 | 內容 |
|---|---|
| [2026-11-14-opencode-sql-agent-improvement.md](notes/2026-11-14-opencode-sql-agent-improvement.md) | **Agent 改善建議**：Evidence-Driven Development、限定 Target SP、Minimal Diff、Retry Limit、Reviewer Agent 分工 |
| [README.md](notes/README.md) | Notes 資料夾使用說明 |

---

## 🚀 快速開始

1. 將 `docs/` 內嘅 `AGENTS.md` 放入你嘅 CR 工作目錄（OpenCode 會自動讀取）
2. 用 `prompts/START_CR_PROMPT.txt` 開始每個 session
3. 每個階段完成後，用 `prompts/` 內對應嘅驗證 Prompt 做獨立檢查
4. 所有結論標記 `Verified` / `Assumed` / `Unknown`，並附上檔案路徑同行號

---

## ⚠️ 核心安全規則（摘要）

- **永不**執行 production SQL、**永不**自動部署
- `CR/`、`DATABASE/`、`OLD/` 一律 **只讀**
- `SOLUTION/` **只在批准後**修改
- 任何推測必須標記為 `Assumed` 或 `Unknown`，唔可以當成事實
- 自動修正最多 **2 次**，之後必須停手並要求人手覆核

---

*Last updated: 2026-11-14*
