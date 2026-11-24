# OpenCode CLI：隱藏 Reasoning（思考過程）顯示設定

在 OpenCode CLI / TUI 中，**遮住 reasoning 畫面顯示（模型照常思考，只隱藏輸出區塊）** 的設定並非寫在專案的 `opencode.json(c)`，而是寫在全域 CLI 設定檔 `~/.config/opencode/cli.json`。

- `opencode.json(c)`：負責 Server、Project、Model 及 Tool Permissions 設定。
- `cli.json`：負責 Terminal TUI / Mini 介面顯示、快捷鍵與外觀，且**只讀取全域設定檔**（放在 project 資料夾內不會生效）。

---

## 1. 設定檔位置

- **Linux / macOS**：`~/.config/opencode/cli.json`（若有設定 `$XDG_CONFIG_HOME`，則為 `$XDG_CONFIG_HOME/opencode/cli.json`）
- **Windows**：`%USERPROFILE%\.config\opencode\cli.json`

---

## 2. `cli.json` 設定內容（方案 A）

打開或建立 `cli.json`，加入以下設定：

```json
{
  "$schema": "https://opencode.ai/v2/cli.json",
  "session": {
    "thinking": "hide",
    "verbosity": "low"
  },
  "mini": {
    "thinking": "hide"
  },
  "keybinds": {
    "session.toggle.thinking": "<leader>t"
  }
}
```

---

## 3. 各欄位作用說明

| 欄位 | 可選值 | 作用 |
|---|---|---|
| `session.thinking` | `"show"` / `"hide"` | 全螢幕 TUI 預設顯示或隱藏模型的 reasoning / thinking 區塊。設為 `"hide"` 後模型仍會在背景完成推理，不會影響分析與 SQL 準確度。 |
| `session.verbosity` | `"low"` / `"medium"` / `"high"` | （可選）控制對話紀錄詳細度；設為 `"low"` 可將思考與連續工具執行過程收摺成簡短摘要。 |
| `mini.thinking` | `"show"` / `"hide"` | 若使用 `opencode mini` 簡約互動模式，預設一併隱藏 reasoning 顯示。 |
| `keybinds.session.toggle.thinking` | 快捷鍵字串（預設為 `"none"`） | 綁定即時切換顯示／隱藏 reasoning 的快捷鍵。設為 `"<leader>t"` 後，在 TUI 按 `Ctrl+X` 再按 `t` 即可隨時展開或收起思考過程。 |

---

## 4. TUI 介面直接修改方式

在 OpenCode TUI 執行期間：

1. 按 `Ctrl+P` 打開 Command Palette。
2. 選擇 **Open settings**。
3. 將 **Thinking** 改為 **Hide**（會自動寫入全域 `cli.json` 並即時生效）。
