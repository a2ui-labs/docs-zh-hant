# 繁體中文翻譯同步記錄 2026-07-19

本次同步涵蓋上游 A2UI 專案 `305336f1` 到 `3708c069`（共 223 個 commit）之間，`docs/`、`mkdocs.yaml` 與 `README.md` 的所有變更。

## 結構調整

- 上游將 mkdocs 的 `docs_dir` 從 `docs` 改為 `docs/public`，因此本倉庫也用 `git mv` 把幾乎所有文件從 `docs/<path>` 搬到 `docs/public/<path>`，以保留檔案歷史。`docs/scripts/**`（開發用腳本）維持原位不動。
- `docs/glossary.md` 移入 `docs/public/concepts/glossary.md`（不只是路徑前綴，還多了一層 `concepts/` 子目錄）。已檢查並修正檔案內所有相對連結（含指向 repo 根目錄 `specification/`、`renderers/` 的連結，以及 `overview.md` 中指向詞彙表的連結）。
- 從上游複製了新的二進位資源：`assets/favicon.svg`（新檔案），以及已存在但重新同步過的 `assets/guides-a2ui-over-mcp-tour.mp4`、`assets/landscape-architect-demo.mp4`、`assets/mcp-apps-calculator-demo.gif`。
- `docs/scripts/convert_docs.py`、`docs/scripts/test_convert_docs.py` 為開發工具腳本，依原文逐字複製、不翻譯；`docs/scripts/README.md` 上游無變更，維持原樣未動。

## `mkdocs.yaml`

- 新增 `docs_dir: docs/public`。
- Concepts 導覽新增「詞彙表」項目。
- Guides 導覽把三個 MCP 相關頁面收整進新的「A2UI + MCP」子選單；「在任意 Agent 框架中使用 A2UI (AG-UI)」項目更名為「在任意 Agent 框架與 Harness 中使用 A2UI」。
- Specifications 導覽重整為四個版本群組：v1.0（候選版）、v0.9.1（現行版）、v0.9（前一穩定版）、v0.8（舊版），並新增 v0.9.1 與 v1.0 各自的 A2UI 協議、A2A 擴充、演進指南、基礎 Catalog 指南頁面。
- `repo_name` / `repo_url` / `edit_uri` 更新為 `a2ui-project/a2ui`（原為 `google/A2UI`）。
- `favicon` 改為 `assets/favicon.svg`。
- `exclude_docs` 新增 `specification/v*/**/*.md`。

## 內容翻譯

依上游差異更新了以下文件的翻譯（保留未變動段落，只翻譯新增/變更內容）：

- 根目錄：`README.md`（v0.8 → v0.9.1 狀態說明、AG-UI 用字、Corepack/yarn 安裝流程、`a2ui-project/a2ui` 倉庫網址、AG-UI CLI 流程）。
- `index.md`、`quickstart.md`、`roadmap.md`
- `concepts/`：`actions.md`、`catalogs.md`、`components.md`、`data-binding.md`、`data-flow.md`、`glossary.md`（由 `docs/glossary.md` 移入）、`overview.md`、`transports.md`
- `guides/`：`a2ui-in-mcp-apps.md`、`a2ui_over_mcp.md`、`agent-development.md`、`authoring-components.md`、`client-setup.md`、`defining-your-own-catalog.md`、`mcp-apps-in-a2ui.md`、`theming.md`
- `guides/a2ui-with-any-agent-framework.md` 與 `guides/renderer-development.md`：上游整篇改寫，採全新翻譯（僅參考舊譯文的用詞風格）。
- `ecosystem/a2ui-in-the-world.md`、`ecosystem/community.md`
- `ecosystem/renderers.md`：上游整篇改寫，採全新翻譯。
- `introduction/agent-ui-ecosystem.md`、`introduction/how-to-use.md`、`introduction/what-is-a2ui.md`、`introduction/who-is-it-for.md`
- `reference/agents.md`、`reference/components.md`、`reference/messages.md`、`reference/renderers.md`
- `specification/v0.8-a2ui.md`、`v0.8-a2a-extension.md`、`v0.9-a2ui.md`、`v0.9-evolution-guide.md`（版本徽章與交叉連結更新）
- `specification/v0.9.1-*.md`（4 個新檔案）、`specification/v1.0-*.md`（4 個新檔案）：全新翻譯（皆為簡短的索引/摘要頁，實際規格內容透過 `--8<--` 從上游 `specification/` 匯入，不在本倉庫範圍內）。

`composer.md` 上游無變化，僅路徑搬移，未做文字更動。

## 未變動 / 略過項目

- 頂層 `specification/`（原始 JSON schema）與 `eval/` 資料夾不屬於文件翻譯範圍，本倉庫本來就沒有對應目錄，略過。
- 沒有加入 `docs/public/CNAME`（本倉庫從一開始就刻意不使用上游的自訂網域，見 `TRANSLATION_BOOTSTRAP_2026-03-21.md`）。

## 後續可留意事項

- `specification/v0.9.1-*.md`、`v1.0-*.md` 目前只是索引頁；若之後要讓 `--8<--` include 實際生效，需要額外準備上游 `specification/v0_9_1/` 與 `specification/v1_0/` 底下的原始文件（不在本次任務範圍內）。
