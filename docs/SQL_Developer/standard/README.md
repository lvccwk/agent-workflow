# 開始使用：建立 CR 工作資料夾

先按本頁建立工作位置及放好檔案，再執行 [SP_CR_Workflow_Lite.md](SP_CR_Workflow_Lite.md) 的 Step 1。

## 1. 工作資料夾放在哪裡？

`standard/` 是共用文件與範本的位置。每個 CR 另建一個獨立工作資料夾，放在公司核准的工作位置。下圖只表示資料夾關係；「範本位置」與「工作位置」由使用者自行選擇，不代表固定磁碟或個人路徑。

```text
<範本位置>/
└── SQL_Developer/
    ├── opencode.jsonc
    └── standard/          ← 本頁、AGENTS.md、流程及報告範本

<工作位置>/
├── CR-001/                ← 第一個 CR，從這裡開始工作
└── CR-002/                ← 下一個 CR 另開一個資料夾
```

`CR-001` 請換成實際 CR 編號。以下所有 `CR/`、`OLD/`、`REPORT/` 等相對路徑，都以該 CR 工作資料夾為起點。

## 2. 建立資料夾並複製範本

在檔案總管開啟 `standard/` 資料夾，於該處開啟 PowerShell，再貼上以下指令。程式會自動識別範本位置，並提示你輸入工作位置的完整路徑（不含 `CR-001`）。將 `$crId` 改成實際 CR 編號後執行；這段只建立資料夾及複製文件，不會執行 SQL。

```powershell
# Run from standard/ and adjust the CR number before running.
$templateRoot = (Get-Location).Path
$sourceRoot = Split-Path -Parent $templateRoot
$workRoot = Read-Host 'Enter the absolute path for your CR workspace (without the CR number)'
if ([string]::IsNullOrWhiteSpace($workRoot) -or -not [System.IO.Path]::IsPathRooted($workRoot)) {
    throw 'An absolute workspace path is required.'
}
$crId = 'CR-001'
$crRoot = Join-Path $workRoot $crId

# Use a new CR folder to keep existing work intact.
if (Test-Path -LiteralPath $crRoot) {
    throw "CR folder already exists: $crRoot. Choose a new CR folder or inspect the existing one."
}

$filesToCopy = @(
    @{ Source = Join-Path $templateRoot 'AGENTS.md'; Target = 'AGENTS.md' }
    @{ Source = Join-Path $sourceRoot 'opencode.jsonc'; Target = 'opencode.jsonc' }
    @{
        Source = Join-Path $templateRoot 'CR_Report_Template_Business.md'
        Target = 'REPORT\CR_Report_Template_Business.md'
    }
)

# Check all source files before creating the workspace.
foreach ($file in $filesToCopy) {
    if (-not (Test-Path -LiteralPath $file.Source -PathType Leaf)) {
        throw "Missing source file: $($file.Source)"
    }
}

$folders = @(
    'CR', 'DATABASE', 'OLD', 'SOLUTION', 'REPORT',
    'TEST\verification_scripts', 'TEST\expected_results',
    'TEST\actual_results\before', 'TEST\actual_results\after'
)

New-Item -ItemType Directory -Path $crRoot -ErrorAction Stop | Out-Null
foreach ($folder in $folders) {
    New-Item -ItemType Directory -Path (Join-Path $crRoot $folder) -ErrorAction Stop | Out-Null
}
foreach ($file in $filesToCopy) {
    Copy-Item -LiteralPath $file.Source -Destination (Join-Path $crRoot $file.Target) -ErrorAction Stop
}

Set-Location -LiteralPath $crRoot
Get-Location
Get-ChildItem -Force
```

完成後，`Get-Location` 應顯示你所選工作位置下的 `CR-001/`（即 `$crRoot`）。`AGENTS.md` 取自 `standard/`，`opencode.jsonc` 取自上一層 `SQL_Developer/`；正式使用前須確認符合組織核准設定。資料夾建立本身不會設定檔案系統唯讀權限。

目前設定未允許建立 `TEST/uat_queries.sql`。開始 Step 2 前，須由設定管理者核准該路徑的寫入權限；不可用其他工具繞過限制。測試 SQL 的執行仍由 DBA 負責。

## 3. 將資料放到正確位置

以下路徑均以 `CR-001/` 為根目錄，與你選用的磁碟及工作位置無關。

| 你手上的資料 | 放置位置 | 準備方式 |
|---|---|---|
| 已核准 CR Word | `CR/CR.txt` | 匯出為文字，保留需求及驗收條件 |
| CR 的 Excel 附件 | `CR/CR_attachment.csv` | 匯出相關工作表；多張表可另存並註明對應關係 |
| 報表連線清單 | `CR/report_list.csv` | 先移除密碼、Token 等憑證，保留分析所需欄位 |
| NSC transition plan | `transition_plan.txt ` | 放在 CR 根目錄，與 `AGENTS.md` 同層 |
| 相關 DDL、SP、呼叫端參考資料 | `DATABASE/` | 放入已授權的參考副本，分析時保持不變 |
| 修改前的 SP | `OLD/` | 保留原始版本，分析時保持不變 |
| 已修改、待比較的 SP | `SOLUTION/` | 使用已核准變更的版本，檔名及相對路徑與 `OLD/` 對應 |
| 業務核准的預期結果（如有） | `TEST/expected_results/` | 使用獨立驗收依據，不能由修改後結果倒推 |

`REPORT/` 的報告及 `TEST/uat_queries.sql` 會在流程中產生；`TEST/actual_results/before/` 和 `after/` 先留空，之後由 DBA 提供去識別化的執行證據。

若只有原始 SP，尚未有修改後版本，先完成需求分析與變更核准；待 `SOLUTION/` 準備好後才開始這份比較流程。

## 4. 從 CR 根目錄開始工作

1. 在編輯器或 Agent 工具開啟剛建立的 `CR-001/` 作為工作資料夾，讓工具讀取該處的 `AGENTS.md` 及專案設定。
2. 確認 Terminal 的目前位置也是該資料夾；若仍使用上面的 PowerShell 工作階段，可執行 `Set-Location -LiteralPath $crRoot`。若已重開 Terminal，請從 `CR-001/` 資料夾開啟。
3. 確認 `OLD/`、`SOLUTION/` 均有對應 SQL 檔案，需求及參考資料已放好，且 `REPORT/CR_Report_Template_Business.md` 存在。
4. 開啟 [SP_CR_Workflow_Lite.md](SP_CR_Workflow_Lite.md)，由 Step 1 開始。其指令應在 CR 根目錄執行。

例如，流程中的 `REPORT/01_changes.md` 就是 `CR-001/REPORT/01_changes.md`。完整目錄樹見流程文件的 Folder Structure。
