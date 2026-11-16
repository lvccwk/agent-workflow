# CR-001 目錄配置（AGENTS.md + opencode.jsonc）

放喺 CR-001/ 入面，同 AGENTS.md 放埋一齊：

```text
CR-001/
├── AGENTS.md        ← 規則（叫 agent 守）
├── opencode.jsonc   ← 權限（真正擋住 agent）
├── CR/ OLD/ DATABASE/ SOLUTION/ TEST/ REPORT/
```

## opencode.jsonc

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "permission": {
    // 讀：全部可以，但唔准讀密碼 / key
    "read": {
      "*": "allow",
      "*.env": "deny",
      "*.env.*": "deny",
      "*.pem": "deny",
      "*.key": "deny"
    },

    // 寫：預設全部禁止，下面逐個開
    "edit": {
      "*": "deny",
      "REPORT/**": "ask",            // 寫 report 前問你
      "TEST/**": "ask",              // 寫 UAT SQL 前問你
      "TEST/actual_results/**": "deny", // DBA 嘅結果，唔准改
      "SOLUTION/**": "ask"           // Gate A：改 SP 前一定問你
    },

    // Shell：全部禁止，只准睇 git 狀態
    "bash": {
      "*": "deny",
      "git status*": "allow",
      "git diff*": "allow"
    },

    "webfetch": "deny",            // 唔准上網（防止資料外洩）
    "external_directory": "deny",  // 唔准走出 CR folder
    "task": "deny"                 // 唔准開 subagent（Step 7 用新 session）
  }
}
```

## AGENTS.md 每條規則對應邊個權限

| AGENTS.md 規則 | 權限設定 |
| --- | --- |
| CR/、OLD/、DATABASE/ 唔准改 | `edit "*": deny` |
| SOLUTION/ 要等 Gate A | `SOLUTION/**: ask` |
| actual_results/ 唔准改 | `TEST/actual_results/**: deny` |
| 唔准跑 SQL、唔准 deploy / push | `bash "*": deny` |
| Step 7 用新 session | `task: deny` |

## 有兩點要留意

1. **規則嘅次序有關係**：如果有幾條都符合，以最後嗰條為準。所以 `"*": "deny"` 一定要擺第一，actual_results 要擺喺 `TEST/**` 後面。
2. **ask 唔等於批准**：OpenCode 彈出嚟問你嘅時候，你要先睇下 agent 想改乜，同 plan 啱唔啱，先撳 allow。如果覺得寫 report 都要問好煩，可以將 `REPORT/**` 改做 allow。

## 設定好之後，實測 4 樣嘢（喺 OpenCode 直接打）

1. `Edit OLD/<sp_name>.sql and add a comment.` → 應該被拒絕
2. `Run: sqlcmd -Q "SELECT 1"` → 應該被拒絕
3. `Create REPORT/test.md with "hello".` → 應該彈出嚟問你
4. `Run: git status` → 應該直接執行

4 樣都同預期一樣，就代表規則同權限都生效，可以開始做 Step 1。如果第 1、2 項冇被擋，即係個 config 冇被讀到，可以將結果話我知，我幫你睇下。
