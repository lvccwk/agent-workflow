# UAT 改前 vs 改後結果記錄

大部分都要 run 兩次，因為要證明兩樣嘢：

| 種類 | 改前 vs 改後 | 預期結果 |
|---|---|---|
| 有改動嘅邏輯 | 兩次結果應該唔同 | 改後結果要符合 CR 要求 |
| 冇改動嘅部分（regression） | 兩次結果應該一樣 | 冇改嘅地方唔可以變 |

只 run 改後嗰次，你就證明唔到「其他嘢冇整壞」。

## 執行次序（同 DBA 夾好）

1. UAT 仲係舊版 SP → DBA run SP，再 run 你嘅 SELECT query → 儲存結果到 before
2. DBA deploy 新版 SP
3. 用同一份 data、同一組參數再 run 一次 → 儲存結果到 after
4. 對比兩次結果

⚠️ 中間唔可以有其他人改 UAT 嘅 data，否則兩次結果冇得比較。

## 點樣記錄

Folder：

```text
TEST/actual_results/
├── before/   TC01.csv, TC02.csv ...
└── after/    TC01.csv, TC02.csv ...
```

每個 query 嘅結果用 Test-ID 做檔名，export 做 CSV 或者截圖都得。

Test case 表就加兩欄：

| Test-ID | 場景 | 預期 | Before | After | 結果 | 證據 |
|---|---|---|---|---|---|---|
| TC01 | Title Code = 120 | 應該計入 | 冇 | 有 | PASS | after/TC01.csv |
| TC02 | Title Code = 100（boundary） | 唔計 | 有 | 冇 | PASS | after/TC02.csv |
| TC03 | 其他 agency（regression） | 唔變 | 50 行 | 50 行 | PASS | before/after TC03 |

## 慳時間嘅做法

唔使每一行 data 都對，每個 test 記低三樣就夠：

- 行數：COUNT(*)
- 總數：SUM(金額) 之類嘅重要數字
- 幾行樣本：例如 5 至 10 行有代表性嘅 key record

## 叫 agent 補 query 嘅 prompt

```text
Update TEST/uat_queries.sql:
For each Test-ID, write SELECT queries for COUNT(*),
SUM of key amounts, and top 10 sample rows by key.
Add comment: run BEFORE and AFTER deployment with same data
and parameters, save as before/<Test-ID>.csv and after/<Test-ID>.csv.
Update REPORT/05_test_cases.md with columns:
Before | After | Result | Evidence.
```

拎齊 before 同 after 嘅結果之後，用 P5 之後嗰個 Step 9 prompt，叫 agent 讀 actual_results/ 自動填 PASS/FAIL 就得。
