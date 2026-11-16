# AGENTS.md（9 步版）設計說明

> 配套文件：[AGENTS_9Step.md](AGENTS_9Step.md) — 精簡版規則本體
> 搭配權限：[opencode.jsonc](opencode.jsonc) — 真正擋得住 agent 嘅一層

---

## 點用

1. 開一個新 CR 目錄（例如 `CR-001/`），建立 `CR/ OLD/ DATABASE/ SOLUTION/ TEST/ REPORT/`。
2. 將 `AGENTS_9Step.md` 嘅全部內容複製到 `CR-001/AGENTS.md`（OpenCode 會自動讀取）。
3. 將 `opencode.jsonc` 放到同一個工作目錄（或專案根目錄），確認權限設定生效。
4. 用 9 步流程逐步貼 prompt（見 [CR_9Step_Prompt_Flow.md](../../guides/CR_9Step_Prompt_Flow.md)）。

**唔好只做第 2 步。** 只有 AGENTS.md 冇 opencode.jsonc，等於得個「口頭勸告」。

---

## 點解咁寫

| 節 | 設計理由 |
| --- | --- |
| **§3 Safety** | 講**永遠唔准做**嘅嘢：唔連 DB、唔跑 SQL、唔部署、UAT 只准 SELECT。呢節係底線，任何情況都唔可以讓步 |
| **§7 Stop rules** | 講**幾時要停低問你**：需求唔清、缺 dependency、撞到 UNKNOWN、diff 唔符計劃、同一個 fix 失敗兩次。有咗停損點，agent 唔會錯落去 |
| **§5 Evidence** | 防止 agent「作故仔」：每條規則要 file + line、CONFIRMED / INFERRED / UNKNOWN 分級、唔准把猜測升級成事實、冇證據唔准講 PASS |
| **§8 Reply format** | 每次回覆固定格式（最多 12 行），你一眼睇到：做到邊一步、有咩發現、改咗咩檔、要你決定咩 |
| **§6 Minimal change** | 防止 agent「順手」幫你 reformat、改名、refactor — 呢啲係 code review 最難捉嘅風險 |
| **§4 Workflow** | 9 步 + 3 個 Gate，明確寫死「唔可以跳步」同每個 Gate 要邊個批 |

### 為什麼規則用英文、回覆用繁中

- 規則（instructions）用英文：主流模型對英文指令嘅跟隨度較穩定，歧義少，亦方便跨團隊共用。
- 回覆用繁體中文（HK）：你日常睇得快，唔使翻譯。
- SQL 名稱保持英文：避免識別碼被翻譯或改寫。

所以 §1 特登寫 `Reply in Traditional Chinese (HK). Keep SQL names in English.`

---

## 兩層防護：AGENTS.md vs opencode.jsonc

| 層 | 角色 | 限制 |
| --- | --- | --- |
| `AGENTS.md` | **勸告層** — 用文字叫 agent 守規則 | Agent 可以唔守、可以誤解、長對話後可能忘記 |
| `opencode.jsonc` | **強制層** — 用工具權限封死動作 | 呢層唔靠 agent 自律，係 runtime 直接 deny |

**實務上兩樣要一齊用**，因為：

- AGENTS.md 寫「SOLUTION/ 只可以 Gate A 後改」→ 但 agent 想改就改得到；要靠 `opencode.jsonc` 將 `edit: SOLUTION/*` 設成 `ask`，先真正有閘。
- AGENTS.md 寫「唔准 run SQL」→ 靠 `shell: *` deny，只留 `git status` / `git diff`。
- AGENTS.md 寫「唔准上網」→ 靠 `webfetch` / `websearch` / `subagent` 全部 deny。

補充幾點現實限制：

1. **唔連 DB 最徹底嘅做法**係環境本身冇 credential — 唔好靠設定檔。
2. `opencode.jsonc` 要按公司保安要求 review，唔好當佢係萬能。
3. AGENTS.md 唔係 OS 級隔離，佢只係 prompt 層嘅約束。
4. 公司未批准用 AI 之前，唔好將真實客戶資料或 production 連線字串放入工作目錄。

---

## 同其他文件嘅關係

| 文件 | 內容 | 用幾時 |
| --- | --- | --- |
| [AGENTS_9Step.md](AGENTS_9Step.md) | 9 步精簡版規則（本文件配套） | 日常開 CR，直接複製入 `CR-001/AGENTS.md` |
| [AGENTS.md](AGENTS.md) | 8 階段完整版規則 | 需要更詳細嘅階段定義、審批條文時 |
| [SDLC-Framework.md](SDLC-Framework.md) | 7 階段框架、角色責任、風險與 rollback | 培訓、審計、文件存檔 |
| [opencode.jsonc](opencode.jsonc) | 工具權限基線 | 每次開新工作目錄都要放 |
| [../../guides/CR_9Step_Prompt_Flow.md](../../guides/CR_9Step_Prompt_Flow.md) | 9 步逐步 prompt | 每步貼一次 |
