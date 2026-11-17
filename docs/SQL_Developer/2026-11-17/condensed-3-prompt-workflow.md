而家已經有 report 範本，P2 至 P4（改前、改後、影響分析）全部可以交俾範本一次過做，5 個 prompt 減做 3 個：

| 新 | 合併咗 | 產出 |
|---|---|---|
| ① 搵改動 | P1 | 01_changes.md |
| ② 出 Report | P2 + P3 + P4 | CR9_Report.md（有齊圖） |
| ③ Test Case | P5 + before/after | test_cases.md + uat_queries.sql |

## ① 搵改動

```text
Run: git diff --no-index -w OLD/ SOLUTION/ > REPORT/00_raw_diff.txt
Read CR/ and NSC plan. Create REPORT/01_changes.md:
| C-ID | SP | Line | Before | After | REQ-ID | CR quote |
One row per diff hunk. At the end, list ⚠️ changes without
a requirement, and requirements without a change.
```

👉 你自己睇一次，呢份啱咗，後面先會啱。

## ② 出 Report

```text
Fill REPORT/CR_Report_Template_EN.md using 01_changes.md,
OLD/, SOLUTION/ and DATABASE/. Save as REPORT/CR9_Report.md.
Real Mermaid diagrams, max 10 boxes, changed = red.
Search DATABASE/ for callers. Unsure = UNKNOWN. English.
```

## ③ Test Case + SQL

```text
From CR/ and CR9_Report.md sections 5–6, create:
- REPORT/test_cases.md: | Test-ID | C-ID | Scenario | Expected |
  Before | After | Result | Evidence | (cover normal, boundary,
  NULL, regression; Result = NOT RUN)
- TEST/uat_queries.sql: SELECT only. Per Test-ID: COUNT, SUM,
  top 10 rows. Run before and after deployment, save as
  before/<Test-ID>.csv and after/<Test-ID>.csv.
```

UAT 有結果之後再用一句：

```text
Read TEST/actual_results/, update test_cases.md and
section 7 of CR9_Report.md with PASS/FAIL.
```

開工之前，記得先將 CR Word、Excel 附件同 NSC plan 轉做 .txt / .csv，同埋將範本放入 REPORT/。時間夠嘅話，① 做完開個新 session 核對一次，會準確好多。
