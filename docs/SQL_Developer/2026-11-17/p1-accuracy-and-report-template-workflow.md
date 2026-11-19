先講點樣令 P1 準確啲，之後我幫你整一份有圖嘅 report 範本。

## 點樣令 P1 驗證準確啲

### 1. 先將 Word 同 Excel 轉做文字檔（最重要）

Agent 讀 .docx 同入面夾住嘅 Excel 經常會漏嘢。開工之前，你自己用幾分鐘轉一轉：

- CR Word → 另存做 `CR/CR9.txt`
- Word 入面嘅 Excel 附件 → 打開再另存做 `CR/CR9_attachment.csv`
- NSC transition plan → 另存做 `transition_plan.txt `

### 2. 用工具搵改動，唔好靠 agent 自己睇

`git diff` 係由電腦逐行對比，一行都唔會漏，而你嘅權限設定本身已經容許 `git *`。Agent 只負責解釋每一處改動，唔使自己搵。

### 3. 雙向對照

每一處改動都要對應到一個需求；每一個需求都要搵到對應嘅改動。任何一邊對唔上，就係要問嘅位。

### 4. 開新 session 再核對一次

叫另一個 session 用 raw diff 逐項核對 P1 張表。

## 新版 P1（準確版）

```text
Step 1: Run this and save output to REPORT/00_raw_diff.txt:
  git diff --no-index -w OLD/ SOLUTION/

Step 2: Read CR/CR9.txt, CR/CR9_attachment.csv and
transition_plan.txt .

Step 3: Create REPORT/01_changes.md. For EVERY diff hunk in
00_raw_diff.txt, one row:
| C-ID | SP | Line | Before | After | REQ-ID | CR evidence (quote) |

Step 4: Two checks at the bottom:
- Changes with NO matching requirement → list as ⚠️
- Requirements with NO matching change → list as ⚠️

Rules: number of rows must equal number of diff hunks.
Quote exact SQL and exact CR sentence. Unsure = UNKNOWN.
Do not modify OLD/ or SOLUTION/.
```

## P1 核對（開新 session）

```text
You are an independent reviewer.
Compare REPORT/00_raw_diff.txt with REPORT/01_changes.md.
Report: any missing hunk, wrong line number, wrong Before/After,
wrong REQ mapping. Output a table only. Do not modify files.
```

## P1 冇問題之後，叫 agent 填範本

先將 CR_Report_Template.md 放入 REPORT/，然後打：

```text
Use REPORT/CR_Report_Template.md as the exact format.
Fill it using REPORT/01_changes.md, OLD/, SOLUTION/,
DATABASE/ and CR/. Save as REPORT/CR9_Report.md.

- Section 3, 4: draw real Mermaid diagrams from the actual SP.
  Max 10 boxes per diagram. Changed = red, new = green.
- Section 5: one `diff block per change from 01_changes.md.
- Section 6: search DATABASE/ for callers of changed SPs/tables.
- Section 7: leave results as NOT RUN until actual results exist.
Keep the template structure. Unsure = UNKNOWN.
```

睇圖嘅方法：喺 VS Code 安裝「Markdown Preview Mermaid Support」extension，然後撳 Ctrl+Shift+V 就睇到啲圖。放上 GitHub 嘅話，佢本身會自動顯示。
