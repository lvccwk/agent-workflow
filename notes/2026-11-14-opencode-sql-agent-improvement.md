# OpenCode SQL Agent 改善建議 — Evidence-Driven Development

> 目標：解決 Agent 不斷做錯、重複分析、改錯 Stored Procedure 嘅問題。
> 核心轉變：由「叫 Agent 完成工作」變成「叫 Agent 提供證據，證明每一步正確」。

---

## 一、點解 Agent 不斷做錯？

| 問題 | 可能原因 | 改善方法 |
|---|---|---|
| 理解錯 Business Logic | Agent 自行推斷需求 | 要求引用 SQL 證據 |
| 改錯 Stored Procedure | 分析範圍太大 | 指定 Target SP |
| 修改咗不相關 SQL | 冇明確限制修改範圍 | 限制 Minimal Diff |
| 不斷重新分析 | Session Context 太長 | 分階段開 Session |
| UAT SQL 不準確 | Agent 自行假設 Expected Result | 先確認 Business Rules |
| Agent 不斷重試 | 缺少停止條件 | 設定 Retry Limit |

**最建議改變嘅係**：由「叫 Agent 完成工作」變成「叫 Agent 提供證據，證明每一步正確」。

---

## 二、第一個改善：唔好一開始分析成個 Project

之前用嘅 prompt：

```text
Analyze this project and explain:
System architecture and data flow
Key business logic
Stored procedure dependencies
Potential risks and edge cases
```

**問題**：範圍太廣。如果 `DATABASE/` 有 20 個 Stored Procedures，Agent 可能將大量時間用喺不相關嘅 SQL 上，甚至混淆唔同 Procedure 嘅業務用途。

### 建議改用 Prompt 1.1

```text
Act as a Senior SQL Server System Analyst.
Your task is to understand the EXISTING
implementation of this Change Request.

Scope:
CR/ : Business requirements
OLD/ : Original stored procedures
DATABASE/ : Reference procedures and schema

Instructions:
Identify the target stored procedure(s)
based on the CR requirements.
Analyze ONLY the target procedures
and their confirmed dependencies.
Explain the existing logic step by step.
Identify:
Input parameters
Source tables
JOIN conditions
WHERE conditions
CASE WHEN logic
Temporary tables
Output tables
For each business rule, provide:
SQL file
Procedure name
Relevant line number
Explanation
Confidence level
Mark any unverified business rule
as UNKNOWN.

Do not propose implementation changes.
Do not modify files or execute SQL.
STOP after the As-Is analysis.

Explain in Traditional Chinese.
```

**最重要係第 5 點** — 要求提供：

```
Business Rule → SQL Statement → File Reference
```

例如：

| Business Rule | SQL Evidence | Status |
|---|---|---|
| Title Code >= 100 | sp_Agency.sql Line 145 | Verified |
| Inforce Agency Head | sp_Agency.sql Line 178 | Verified |
| Reporting Relationship | Missing reference | Unknown |

呢個可以迫使 Agent 區分**已證實嘅 Code 行為**同**佢自己推測嘅業務解釋**。

---

## 三、第二個改善：實施 Evidence-Driven Development

建議喺 `AGENTS.md` 加入以下規則：

### Evidence-Driven SQL Development Rules

#### 1. No Assumptions

- Never invent business rules.
- If a requirement is unclear:
  - Identify the ambiguity.
  - Explain why it matters.
  - Ask for clarification.
  - Stop implementation if the ambiguity affects correctness.

#### 2. Evidence Required

Every proposed SQL modification must include:

- Requirement ID
- Procedure name
- Original SQL statement
- Proposed SQL statement
- Reason for modification
- Potential side effects

#### 3. Minimal Changes

Only modify SQL statements required by approved requirements.

Do not:

- Refactor unrelated SQL
- Rename variables unnecessarily
- Change existing calculations
- Modify unrelated JOIN conditions
- Change output schema without approval

#### 4. Validation

After implementation:

- Compare OLD vs SOLUTION.
- Identify all SQL changes.
- Map every change to requirements.
- Check for unintended changes.
- Identify unverified behavior.

#### 5. Stop Conditions

STOP immediately if:

- Business requirements conflict.
- Required SQL dependencies are missing.
- A proposed change affects unknown logic.
- Tests fail repeatedly.
- The approved implementation plan cannot be followed.

Do not automatically attempt unlimited fixes.

**Maximum automatic correction attempts: 2.**

After 2 failed attempts:

- Stop.
- Report the error.
- Explain possible root causes.
- Request human review.

#### 6. Safety

- Never execute production SQL.
- Never deploy changes automatically.
- Never modify `OLD/` or `DATABASE/`.
- All explanations must be in Traditional Chinese.

> 呢個係 Prompt 層面嘅控制，唔係絕對保證。亦應該配合 OpenCode Permissions 限制檔案修改及 Shell 執行權限。

---

## 四、第三個改善：每次只交一個細 Task

例如要修改三個 Stored Procedures。

**❌ 不建議：**

```text
Implement all CR changes and verify
that everything is correct.
```

**✅ 建議：**

```text
Implement Requirement R01 only.

Target:
SOLUTION/sp_AgencyReport.sql

Approved change:
Update the agency head eligibility logic
according to the implementation plan.

Restrictions:
Modify only this stored procedure.
Preserve all unrelated logic.
Do not change output columns.
Do not modify other files.
Do not execute SQL.

After modification:
Show the exact SQL diff.
Explain each changed statement.
Map changes to Requirement R01.
Identify regression risks.
Stop and wait for review.
```

完成 R01 → 檢查 Diff → 先繼續 R02。

呢種做法比一次過修改所有 SP 更容易發現問題。

---

## 五、第四個改善：加入獨立 Reviewer Agent

OpenCode 支援建立專門嘅 Subagent（例如 Reviewer），可以用另一個 Context 專門做 Code Review。

可以將工作分成：

| Agent | 責任 |
|---|---|
| Analyst | 分析 Requirement、As-Is |
| Developer | 根據 Approved Plan 修改 SQL |
| Reviewer | 重新比較 CR、OLD、SOLUTION |
| QA | 產生 UAT Cases、檢查實際結果 |

唔一定要建立四個自訂 Agent。初期用不同 Session 及不同 Prompt 已經可以實行。

不過，同一個 Model 即使換 Session，仍然可能犯相同錯誤，因此 **Reviewer 必須有清楚嘅驗證依據**。

---

## 六、GPT-6 Luna High 應該點樣用？

建議先試以下安排，唔好即刻全部改用更昂貴嘅 Model。

| SDLC 工作 | 建議 |
|---|---|
| As-Is Analysis | Luna High + 限定 Target SP |
| Requirement Analysis | Luna High + Evidence Matrix |
| Impact Analysis | Luna High + Dependency List |
| Implementation Plan | Luna High + Human Approval |
| SQL Modification | Luna High + 每次一個 Requirement |
| Independent Review | 新 Session，必要時換 Model |
| UAT Test Generation | 以確認後嘅 Business Rules 為依據 |
| UAT Acceptance | 實際執行證據 + Human Sign-off |

> High Reasoning 可以增加推理資源，但唔會自動補足缺失嘅 Schema、Business Rules 或執行結果。

---

## 七、核心問題分辨

Luna High「不斷做不準 Prompt」，需要分辨係邊種錯誤：

| 類型 | 問題描述 |
|---|---|
| **A. 理解能力問題** | Agent 分析 Stored Procedure 時，經常理解錯 SQL Business Logic |
| **B. 執行能力問題** | Agent 明明分析啱，但真正修改 SQL 時改錯位置、漏改條件 |
| **C. 驗證能力問題** | Agent 修改後聲稱冇問題，但人工檢查發現 SQL 或 UAT Test Cases 有錯 |
| **D. Workflow 問題** | Agent 不斷重新分析、重複修改，同一個問題要 Loop 好多次先完成 |

- 如果係 **A** → 重新設計 Analysis Prompt
- 如果係 **B 或 C** → 加強 Diff Verification 同 Test Evidence
- 如果係 **D** → 調整整個 OpenCode Agent Loop

---

*整理日期：2026-11-14*
