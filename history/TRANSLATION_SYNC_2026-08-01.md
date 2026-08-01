# 繁體中文翻譯同步記錄 2026-08-01

本次同步涵蓋上游 A2UI 專案 `3708c069` 到 `2276f8cc`（共 59 個 commit）之間，`docs/public/`、`mkdocs.yaml` 與 `README.md` 的所有變更。

本倉庫上一次同步記錄於 `history/TRANSLATION_SYNC_2026-07-19.md`（基準 commit `3708c069`），中間錯過了一次同步週期（docs-ja、docs-ko 已完成該週期），因此這次一次補齊兩輪份量。

## `mkdocs.yaml` 與 `README.md`

兩者在 `3708c069` 到 `2276f8cc` 這段期間，上游皆無任何變更（`git diff 3708c069 2276f8cc -- README.md mkdocs.yaml` 輸出為空）。已核對確認，本倉庫這兩個檔案的譯文維持不變，未做任何修改。

## 內容翻譯

依上游差異更新了以下 14 個檔案的翻譯（保留未變動段落，只翻譯新增／變更內容）：

- `concepts/catalogs.md`：「實作 Renderer」一節改寫為新的三步驟流程（`ComponentApi` + `CatalogComponent` + `AngularCatalog`，取代舊版 `DynamicComponent` / `inputBinding` 寫法），並新增一則 `[!NOTE]`，說明 Orchestrator 示範目前仍為 v0.8、v0.9 範例請見 Angular explorer 的 `DemoCatalog`，以及客戶端函式的執行邊界由目錄定義決定。
- `ecosystem/a2ui-in-the-world.md`：僅將 3 個指向上游 GitHub 的絕對連結（Agent SDK、Restaurant Finder、Component Gallery 範例）改為相對路徑連結，文字內容未變。
- `ecosystem/renderers.md`：社群渲染器表格新增 **BoteAI/a2ui**（`@boteai/a2ui-render`，React）與 **yessGlory17/generative-mui**（`@yessglory/generative-mui-react`，React + Material UI）兩列，移除已下架的 **TanXudong-Vivo/A2UI-Android-Renderer** 一列；**BBC6BAE9/a2ui-swift** 的平台欄位與重點介紹段落全面改寫（改為強調透過共用 `A2UISwiftCore` 層同時支援 SwiftUI／UIKit／AppKit、支援 v0.8/v0.9/v0.9.1、以規格對齊與原生平台行為為優先）；新增 yessGlory17/generative-mui 的重點介紹段落；`community samples` 連結改為相對路徑。
- `guides/a2ui-in-mcp-apps.md`：4 處指向上游 GitHub 原始碼的絕對連結（README、`inline.js`、`MessageProcessor`、`main.ts`）改為相對路徑連結，文字內容未變。
- `guides/a2ui_over_mcp.md`：**整篇重新翻譯**。這個檔案的既有譯文明顯是更早期版本（沿用了不存在於 `3708c069`／`2276f8cc` 任何規格版本的 `v0.10` catalog ID，且缺少上游現行版本的 Prerequisites、Quick Start（MCP Inspector／Recipe Client Web App 兩種示範方式）、Resources vs. Tools 用途劃分、Verbalization、A2UI Agent SDK 等整節內容），與現行上游內容嚴重脫節，屬於翻譯漂移。已依目前上游 `docs/public/guides/a2ui_over_mcp.md`（v0.9 catalog、含教學影片與 Recipe Studio 示範）全文重新翻譯，並延續本檔案既有的用詞慣例（`server`／`client`／`catalog`／`action` 等維持原文小寫）。
- `guides/authoring-components.md`：步驟 2「實作元件（客戶端）」與步驟 3「注冊到 Renderer」改寫為新的 `ComponentApi` + `CatalogComponent` + `AngularCatalog` 寫法，取代舊版 `DynamicComponent` / `inputBinding` 範例；`Chart` 元件範例補上完整的 `props()` signal 存取方式。
- `guides/client-setup.md`：既有譯文同樣落後於上游數個版本（渲染器表格仍列出已移除的 SwiftUI 列、缺少 Jetpack Compose 的「規劃中」列，`Component Catalogs` 一節仍是舊版說明且未連結到 `defining-your-own-catalog.md`，多處仍是 `TODO：新增 XXX 範例` 佔位文字、Angular 一節仍是舊版 `A2UI_RENDERER_CONFIG` 寫法且缺少 `provideA2Ui` 與依賴注入段落、多個原始碼連結仍是絕對 GitHub 網址），屬於翻譯漂移。已依目前上游內容整篇重新翻譯／校對：渲染器表格改為相對路徑連結並移除 SwiftUI 列；`Component Catalogs` 一節改寫並改連結 `defining-your-own-catalog.md`；Angular 一節改為 `provideA2Ui` 寫法並新增「在 Action Handler 中做依賴注入」小節；補上 Transports／處理使用者操作／錯誤處理三節原本缺漏的實際範例連結（取代 TODO 佔位文字）。
- `guides/mcp-apps-in-a2ui.md`：Sandbox Proxy 段落中 3 處指向上游 GitHub 原始碼的絕對連結改為相對路徑連結，文字內容未變。
- `guides/renderer-development.md`：`web_core` 套件與 React／Lit／Angular renderer 範例連結改為相對路徑，文字內容未變。
- `introduction/what-is-a2ui.md`：v0.9 差異說明句改寫（移除「`createSurface` 取代 `beginRendering`」等 v0.8→v0.9 比較用語，改為直接描述目前訊息格式）。
- `quickstart.md`：v0.9 訊息範例下方的附註句同步簡化（移除版本比較用語）；元件畫廊一節新增「若從全新複製的倉庫執行，需先建置畫廊與其工作區相依套件（`yarn build`）」的說明，並將啟動指令由 `yarn start gallery` 改為 `yarn dev`。
- `reference/agents.md`：3 個示例智慧體（Restaurant Finder、Rizzcharts、Orchestrator）的連結改為相對路徑，文字內容未變。
- `reference/messages.md`：`createSurface` 與元件結構兩處說明句同步簡化（移除「在 v0.9 中，`createSurface` 取代了 `beginRendering`」等版本比較用語）。
- `reference/renderers.md`：React／Lit／Angular 的原始碼連結改為相對路徑；生態系渲染器清單新增 **AGenUI**（跨平台原生渲染器，支援 iOS、Android、HarmonyOS，v0.9）一項。

## 順帶修正的翻譯漂移

在處理 `guides/a2ui_over_mcp.md` 與 `guides/client-setup.md` 時發現，這兩個檔案的既有譯文早在 `3708c069` 基準之前就已經與上游脫節（`a2ui_over_mcp.md` 仍停留在提及不存在的 `v0.10` 目錄版本的舊稿；`client-setup.md` 仍保留多處 `TODO` 佔位文字與已移除的舊版 API）。由於兩者都在本次 diff 範圍內，已依本次任務規則一併重新翻譯校正至目前上游內容，其餘未出現在本次 diff 清單中的檔案則未另行稽核。

## 未變動 / 略過項目

- 頂層 `specification/`（原始 JSON schema）、`docs/contributing/**`、`eval/` 等資料夾不屬於文件翻譯範圍，略過。
- 沒有加入 `docs/public/CNAME`（本倉庫從一開始就刻意不使用上游的自訂網域，見 `TRANSLATION_BOOTSTRAP_2026-03-21.md`）。
