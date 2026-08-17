# 繁體中文翻譯同步記錄 2026-08-17

本次同步涵蓋上游 A2UI 專案（`https://github.com/a2ui-project/a2ui.git`）`43a7bdd8` 到 `44a420b6`（共 21 個 commit）之間，`docs/public/` 與 `mkdocs.yaml` 的所有變更。本次基準與 `docs-ja`、`docs-ko`（這兩個倉庫已於 2026-08-16 完成同一區間的同步）一致。

## 上游變更概觀

這 21 個 commit 絕大多數集中在程式碼與規格層面（v1.0 規格 schema 擴充、`a2ui_core` / `a2ui_agent` 搬遷、conformance 測試重新編排、CSP 與 iframe 安全強化、指令碼整理等）。在 `docs/public/**` 與根目錄 `mkdocs.yaml` 實際產生 diff 的檔案共 5 個，沒有新增或刪除的文件與素材：

| 檔案 | 上游 commit |
| --- | --- |
| `docs/public/concepts/catalogs.md` | `ab35e5bd`（PR #2184，issue #2152） |
| `docs/public/guides/mcp-apps-in-a2ui.md` | `f3614ecc`（PR #2266） |
| `docs/public/guides/renderer-development.md` | `e4882a5a`（PR #2210） |
| `docs/public/stylesheets/custom.css` | `0b614905`（PR #2227） |
| `mkdocs.yaml` | `0b614905`（PR #2227） |

## 內容翻譯

- **概念 - 目錄（`docs/public/concepts/catalogs.md`）** — PR #2184
  - 在 3 個範例目錄 JSON 中，於 `$id` 之後補上相同值的 `catalogId` 欄位：`hello_world`、`hello_world_with_all_basic`、`hello_world_with_some_basic`。上游這 3 處變更在本倉庫都有對應位置（`docs-ko` 因為缺少後兩個範例，只落地了 1 處）。
  - 在「`catalogId` 命名慣例」小節中新增翻譯 **JSON Schema 相容性（`$id` 與 `catalogId`）** 條目：由於 A2UI 目錄目前是以 JSON Schema 文件的形式表示，目錄定義應同時包含 `$id`（供 JSON Schema 工具使用）與 `catalogId`（供 A2UI SDK 與目錄協商使用），且兩者設為相同的 URI。
  - 本倉庫已有完整的 `## 目錄命名與版本控制` 與 `### catalogId 命名慣例` 小節，因此不需要像 `docs-ko` 那樣補譯整節，只追加這一個條目。

- **指南 - A2UI 中的 MCP Apps（`docs/public/guides/mcp-apps-in-a2ui.md`）** — PR #2266
  - 在內層 iframe 的安全條目中新增翻譯「防禦透過超連結的資料外洩」：省略 `allow-popups` 並攔截連結導覽，可以阻止不受信任的內容透過點擊劫持到新開啟的視窗來外洩資料。

- **指南 - Renderer 開發（`docs/public/guides/renderer-development.md`）** — PR #2210（v1.0 雙向函式呼叫）
  - 依上游改版替換 `v1.0 (Candidate)` 分頁中的協定需求：
    - `- **Action 回應（RPC）**` → `- **方向性函式呼叫（RPC）**`：原本描述處理來自 server 的 `actionResponse`，改為處理來自智慧體的 `callRendererFunction` 並回傳 `rendererFunctionResponse`（或 `error`）。
    - `**Client-to-Server 通訊**`：刪除「產生並包含 `actionId`」「支援 `wantResponse: true`」兩條，改為「支援向智慧體發起 `callAgentFunction` 訊息，以執行遠端函式」。
  - `a2uiClientCapabilities` 條目與 `**能力**` 小節維持與上游一致，未更動。

## `mkdocs.yaml` 與樣式表

依 PR #2227（統一版權聲明）將授權標頭正規化為與上游相同，對正文內容沒有影響。

- **`docs/public/stylesheets/custom.css`**：註解起始符 `/**` → `/*`、`Copyright 2025` → `Copyright 2024`、授權 URL `http://` → `https://`
- **`mkdocs.yaml`**：`# Copyright 2025 Google LLC` → `# Copyright 2024 Google LLC`

本區間 `mkdocs.yaml` 的 `nav` 結構沒有變化，`README.md` 也沒有上游變更。

## 附加修正：修好 3 個被建成巨集錯誤頁的文件

本次同步開始導入真實的 `mkdocs build` 驗證，發現有 **3 份文件被建置成錯誤頁而非正文**。這些都與本次 diff 無關，屬於既有問題，與 `docs-ko` / `docs-ja` 在 2026-08-16 修正的完全同類。

統一的處理方式是在文件頂端加入 `render_macros: false` 的 YAML front matter。這 3 份文件都沒有真正使用巨集，因此沒有副作用。

### `Macro Rendering Error` — 源自上游的問題

- **對象**：`concepts/catalogs.md`、`guides/authoring-components.md`
- **原因**：兩份文件的 Angular 範本程式碼區塊中含有 `{{ message() }}` / `{{ title() }}`，被 `mkdocs-macros-plugin`（Jinja2）當成巨集變數求值而拋出 `UndefinedError`。即使寫在程式碼區塊內也無法倖免，因為巨集外掛在 Markdown 解析之前就先對全文做了渲染。
- **範圍**：直接建置上游 A2UI 倉庫同樣可重現，屬於上游本身的問題。這是上游沒有的本地修改，若上游日後自行修正，需依其做法再整理。
- 在此修正之前，本次翻譯的 `catalogs.md` 變更在建置產物中完全看不到。

### `Macro Syntax Error` — 上次同步（2026-08-12）引入的回歸

- **對象**：`composer/index.md`
- **原因**：上次同步為了對齊中文標題錨點而加入的 `### Gemini API 金鑰 {#gemini-api}`，其中的 `{#...}` 被 Jinja2 解析為**註解起始標籤（`{# ... #}`）**，出現 `Missing end of comment tag`，導致整份 Composer 文件被換成錯誤頁。
- **範圍**：上游沒有這個錨點，因此上游不會重現；這是在地化倉庫特有的回歸。
- **結果**：修正後已在建置產物中確認 `id="gemini-api"` 錨點與文內連結皆正常。
- **後續注意**：日後凡是使用 `attr_list` 錨點（`{#...}`）的文件，都要一併加上 `render_macros: false`。

## 驗證

在臨時 venv 中安裝 `mkdocs-material`、`mkdocs-macros-plugin`、`mkdocs-mermaid2-plugin` 等相依套件後，以真實建置進行驗證。

- `mkdocs build` 成功（exit 0）。比對變更前（`git stash`）與變更後的警告清單，確認**本次作業沒有產生任何新增警告**。
- 上述附加修正讓 3 條巨集錯誤警告全部消失（警告數 7 → 4）。
- 在建置產物 HTML 中直接確認了以下內容：
  - `concepts/catalogs`：頁面正常渲染（不再是錯誤頁），新增的「JSON Schema 相容性」條目已呈現，程式碼區塊中的 `catalogId` 行已呈現
  - `guides/authoring-components`、`composer/index`：恢復正常渲染（`composer/index` 另確認 `id="gemini-api"` 錨點）
  - `guides/mcp-apps-in-a2ui`：新增的「防禦透過超連結的資料外洩」條目已呈現
  - `guides/renderer-development`：`callRendererFunction` / `callAgentFunction` / `rendererFunctionResponse` 與「方向性函式呼叫」已呈現，且 `=== "v1.0 (Candidate)"` 分頁縮排維持完好；`actionResponse` / `wantResponse` 已依上游移除
  - `stylesheets/custom.css`：建置產物中的授權標頭已更新為 `Copyright 2024`
- `--strict` 建置仍會因下方既有的連結警告而失敗（與本次作業無關，變更前同樣如此）。

## 未處理範圍

- `docs/contributing/**`、`eval/`、頂層 `specification/`（規格原文 JSON、`specification/v1_0/docs/evolution_guide.md` 等）— 依既有方針完全排除。
  - 本區間上游 PR #2235 修訂了 `specification/v1_0/docs/evolution_guide.md`，但 `docs/public/specification/v1.0-evolution-guide.md` 只是用 `--8<--` 引入該檔案的 stub，本倉庫沒有規格原文來源，因此不在處理範圍內。
- 本次 diff 以外的其餘 `docs/public/**` 檔案未做更動。

## 已知待處理項目（建議後續處理）

- **相對連結無法解析（持續）**：本倉庫是純文件倉庫，`../specification/...` 這類指向上游原始碼路徑的連結無法解析。附加修正後剩下的 4 條建置警告全部源於此（`concepts/glossary.md`），也是 `--strict` 建置失敗的原因。建議統一換成 `https://github.com/a2ui-project/a2ui/blob/main/...` 絕對 URL。
- **上游文件本身的矛盾（沿用上游原文）**：`guides/mcp-apps-in-a2ui.md` 中，權限條目仍寫成 `sandbox="allow-scripts allow-forms allow-popups allow-modals"`（包含 `allow-popups`），但本次新增的「防禦透過超連結的資料外洩」條目卻要求省略 `allow-popups`。翻譯時忠實沿用上游原文，待上游整理後再一併跟進。
- **上游殘留的舊 Composer URL（持續）**：`docs/public/index.md`、`guides/a2ui-with-any-agent-framework.md` 中的 `https://a2ui-composer.ag-ui.com/` 在本區間仍未被上游整理。
- 上次記錄（2026-08-12）中的其餘待處理項目不在本次 diff 範圍內，仍然保留。
