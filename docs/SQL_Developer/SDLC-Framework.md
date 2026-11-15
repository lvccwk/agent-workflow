# SP Change Request SDLC 框架

> Stored Procedure 變更請求嘅標準流程、角色分工同交付物定義。
> 適用於 Production 資料庫嘅 SP 修改，確保每次變更都可追溯、可審核、可回滾。

---

## 1. 摘要

**問題**：現時 SP 變更缺乏統一流程，影響分析、Code Review、UAT 同部署之間嘅責任邊界模糊，出事時難以追溯。

**目標**：建立一份所有持份者都跟嘅 SDLC 框架，令每次 SP 變更都有明確階段、負責人、交付物同簽核點。

**建議方案**：採用 7 階段流程（CR → 影響分析 → 開發／Review → UAT → CAB → 部署 → 驗證），配合固定報告範本同 Git 版本控制。

---

## 2. 範圍

### ✅ 適用

- 所有 Production 資料庫嘅 Stored Procedure 新增、修改、刪除
- 涉及 SP 參數、輸出欄位、業務邏輯、效能優化嘅變更
- 由 SP 衍生嘅 scheduler job、trigger、view 變更

### ❌ 不適用

- 純資料修正（data fix）— 走獨立 Data Change 流程
- 資料表結構變更（DDL）— 走 Schema Change 流程
- 非 Production 環境嘅實驗性修改
- 報表工具層（SSRS / Power BI）嘅格式調整

---

## 3. 流程圖

```mermaid
flowchart LR
    A[提出 CR] --> B[影響分析]
    B --> C[開發 + Code Review]
    C --> D[UAT 測試]
    D --> E[CAB 批准]
    E --> F[部署 Prod]
    F --> G[驗證 / Rollback]

    B -.Gate A.-> C
    D -.Gate B.-> E
    F -.Gate C.-> G
```

### 階段說明

| # | 階段 | 目的 | 停機點 |
|---|---|---|---|
| 1 | **提出 CR** | 記錄業務需求、驗收標準 | Requirement 未清 → 停 |
| 2 | **影響分析** | 找出受影響 SP、table、caller、job、輸出 | 依賴缺失 → 停（Gate A）|
| 3 | **開發 + Code Review** | 最小差異修改 + 獨立審查 | 未經批准 → 停 |
| 4 | **UAT 測試** | 用獨立預期結果驗證 | 預期結果無依據 → 停（Gate B）|
| 5 | **CAB 批准** | 變更諮詢委員會審批 | 未批准 → 停 |
| 6 | **部署 Prod** | 按 deploy script 執行 | 無 rollback 方案 → 停 |
| 7 | **驗證 / Rollback** | 確認結果，必要時回滾 | 驗證失敗 → 即刻回滾（Gate C）|

---

## 4. 各階段責任

| 階段 | 負責人 | 交付物 | Sign-off |
|---|---|---|---|
| 提出 CR | Business Owner / BA | CR 文件（需求 + 驗收標準） | Business Owner |
| 影響分析 | Dev | 受影響 table / app / job 清單、As-Is 分析 | Tech Lead |
| 開發 | Dev | SOLUTION/ 修改後 SQL + Diff | Dev Lead |
| Code Review | DBA | Review 記錄（逐項對照需求） | DBA |
| UAT | QA / User | 測試案例 + 實際執行證據 | Business Owner |
| CAB 批准 | Change Manager | CAB 會議紀錄 | CAB |
| 部署 | DBA | Deploy script + Rollback script | Change Manager |
| 驗證 | DBA / Dev | 部署後驗證報告 | Tech Lead |

> **原則**：同一個人唔可以同時做「開發」同「獨立 Review」；Agent 只可以提供建議，唔可以代任何人 sign-off。

---

## 5. 風險同 Rollback 方案

### 常見風險

| 風險 | 影響 | 緩解措施 |
|---|---|---|
| 改動影響未識別嘅 caller | 其他報表出錯 | 影響分析必須列出所有 caller（job / app / view）|
| 輸出欄位順序改變 | 下游 ETL 或報表失敗 | 禁止未經批准改 output schema |
| 效能退化 | Production 逾時 | UAT 需包含執行計劃（execution plan）對比 |
| 資料不一致 | 數字對唔上 | 部署前後做 reconciliation check |
| Rollback 失敗 | 無法復原 | Rollback script 必須喺 UAT 環境預演過 |

### Rollback 原則

1. **每次變更都要有 rollback script**，同 deploy script 一齊交
2. Rollback script 必須喺非 Production 環境**實際執行過**，並記錄證據
3. 部署後驗證失敗 → **即刻回滾**，唔好嘗試即場修補
4. 回滾後 24 小時內提交 incident report

### 停機條件（STOP）

遇到以下任何一項，即刻停止並上報：

- Business rule 有衝突或唔清晰
- 必要嘅 SQL dependency 缺失
- 改動影響到 UNKNOWN 邏輯
- 測試連續失敗兩次
- 無法跟隨已批准嘅實施計劃

---

## 6. 版本控制

### 基本規則

| 項目 | 規則 |
|---|---|
| 存放位置 | 所有 SP 源碼放入 Git repo（`DATABASE/` 唯讀參考、`OLD/` 基線、`SOLUTION/` 修改）|
| 分支策略 | `main` = Production 對應版本；每個 CR 開 `cr/<CR-ID>` 分支 |
| Commit message | `CR-<ID>: <Requirement-ID> <簡短描述>` |
| 命名規則 | 檔名 = SP 名稱（例如 `sp_AgencyReport.sql`），一個 SP 一個檔 |
| Tag | 每次 Production 部署打 tag：`prod-YYYY-MM-DD-<CR-ID>` |
| 禁止 | 唔准 `push --force`、唔准改寫已部署嘅 history |

### 資料夾契約

```
CR/         業務需求（唯讀）
DATABASE/   參考 DDL、其他 SP、schema（唯讀）
OLD/        原始基線源碼（唯讀，不可改）
SOLUTION/   批准後嘅修改版本
TEST/       測試腳本、預期結果、實際證據
REPORT/     各階段分析報告
```

---

## 7. 待決定事項

| # | 問題 | 影響 | 負責人 | 期限 |
|---|---|---|---|---|
| 1 | CAB 審批週期係每週定每兩週？ | 影響 CR 排期 | Change Manager | 待定 |
| 2 | UAT 環境係咪有 Production 級別嘅資料量？ | 影響效能測試可信度 | DBA | 待定 |
| 3 | 緊急變更（Emergency CR）係咪走簡化流程？ | 影響事故應變速度 | Tech Lead | 待定 |
| 4 | Rollback script 由 Dev 定 DBA 撰寫？ | 影響責任歸屬 | 管理層 | 待定 |
| 5 | 舊 SP（未有 Git 版本）點樣納入基線？ | 影響覆蓋範圍 | DBA | 待定 |

---

## 附錄 A：報告寫作指引

### 點解會覺得難寫

1. **一次過叫 AI 寫完整份 report** — AI 冇公司 context，出嚟嘅內容會好空泛
2. **用文字段落描述多角色流程** — Dev / DBA / QA / Approver 混在一起會好亂
3. **冇先講結論** — 讀者睇唔到重點

### 正確做法

| 做法 | 原因 |
|---|---|
| 先定 MD 範本，每份跟住寫 | 結構固定，內容自然齊 |
| 流程用 Mermaid 畫 | 一圖勝千字，MD 原生支援 |
| 角色同責任用表格 | 每個階段邊個做、交咩、邊個簽，一目了然 |
| 叫 AI 一次寫一節 | 每次俾範本 + 公司實際做法 + 一個滿意例子 |
| 最後叫 AI 做 reviewer | 「扮 IT manager 讀呢份 report，指出邊度唔清楚」 |

### 最重要嘅兩節

**第 1 節（摘要）同第 3 節（流程圖）** — 老闆通常只睇呢兩部分。呢兩節清楚，成份 report 已經清楚大半。

---

## 附錄 B：使用方式

1. 複製呢份文件做新 CR 嘅 `REPORT/00_framework.md`
2. 按實際情況填寫第 1、2 節
3. Mermaid 流程圖按 CR 複雜度調整（可加／減階段）
4. 第 4 節角色表按公司實際編制修改
5. 每次提交報告前，用附錄 A 嘅檢查清單自我審核

---

*版本：1.0 ｜ 最後更新：2026-11-14*
