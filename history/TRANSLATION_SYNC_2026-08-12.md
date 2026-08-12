# 繁體中文翻譯同步記錄 2026-08-12

本次同步涵蓋上游 A2UI 專案 `2276f8cc` 到 `43a7bdd8`（共 27 個 commit）之間，`docs/public/`、`mkdocs.yaml` 與 `README.md` 的所有變更。本次基準與 `docs-ja`、`docs-ko` 一致，三個倉庫已回到同一節奏。

## 上游變更概觀

27 個 commit 中，實際更動 `docs/public/**`、`README.md`、`mkdocs.yaml` 的內容集中於兩個 commit：

- `ec97cb0d` Update documentation to reflect the new A2UI Composer application. (#2201)
- `43a7bdd8` Push composer documentation down into the doc/composer directory. (#2215)

另有兩個 commit 帶來零星文件變動：

- `79931d87` docs: add glossary definitions for Catalog Transformer, A2UI Tag, Tag Unwrapping, and Compilation (#2021)
- `2226b0f8` fix: strengthen iframe security by explicitly omitting navigation permissions to prevent top-level window hijacking. (#2218)

其餘 commit 屬於 Swift/SwiftUI renderer 新增、v1.0 規格 schema 變更、用戶端安全強化、CI 與相依套件升級等，皆超出本倉庫的同步範圍。

本輪文件變更實際上是同一條主線：**A2UI Composer 從 CopilotKit 的第三方 Widget Builder，換成由 A2UI 專案自行營運的官方工具**（`https://a2ui-project.github.io/composer/`），原本單頁的 `composer.md` 也隨之擴充為 `composer/` 目錄下的兩篇文件。

## 內容翻譯

- **A2UI Composer（`docs/public/composer.md` → `docs/public/composer/index.md`）**：刪除原有的 `composer.md`（介紹 CopilotKit Widget Builder 的 16 行文件），並依上游新建的 `composer/index.md` 全文翻譯。涵蓋：上手三步驟、Composer 各面板說明（Gemini 助理、渲染後的 A2UI 預覽、A2UI JSON 編輯器、底部偵錯與檢查頁籤的 Data Model / Events / Errors / Raw Messages）、元件庫、設定（切換 renderer 應用程式、Gemini API 金鑰取得步驟與基於 Web Crypto API 的本機加密儲存說明）、進行中的工作，以及 Raw Messages 訊息清單（`RENDERER_READY`、`A2UI_CATALOG`、`COMPONENT_USAGES`、`DATA_MODEL_CHANGE`、`LLM_REQUEST`、`LLM_RESPONSE`）。
  - 上游原文的 `#gemini-api-key` 錨點無法由中文標題（「Gemini API 金鑰」）產生，因此借助已啟用的 `attr_list` 擴充，為該標題明確指定 `{#gemini-api}` 錨點，內文連結同步調整。

- **A2UI Composer 整合手冊（`docs/public/composer/composer_renderer_integration.md`，新增）**：依上游新建文件全文翻譯。包含背景（Composer 不感知特定 catalog / renderer，仰賴託管於 iframe 的「renderer 應用程式」，透過 postMessage 通訊）、橋接層與各框架封裝、三個範例 renderer 應用程式（Angular/Lit/React），以及以 Angular 建構 renderer 應用程式的完整流程（加入相依套件 → 建立包裝元件 → 以 `provideA2uiSandbox` 啟動），並保留 Zone / Zoneless 變更偵測相容性的 NOTE。
  - 依本倉庫慣例，程式碼區塊與程式碼註解維持上游原文不變。

- **詞彙表（`docs/public/concepts/glossary.md`）**：新增翻譯上游 4 個詞條，位置與上游一致（`Catalog Transformer` 置於 `Basic Catalog` 與 `Surface` 之間，其餘三條置於 `Surface` 與 `Agent 架構` 之間）：
  - `Catalog Transformer`（含「為什麼需要」「範例」兩個子節）：在產生系統提示詞指令或編譯驗證 schema 之前，對 catalog 進行過濾與改寫的規則集，涵蓋上下文視窗 token 最佳化、面向任務的能力護欄、精簡模型簽章三類動機，以及 `ComponentPruningTransformer` / `FunctionPruningTransformer` 兩個範例。
  - `A2UI Tag`、`Tag Unwrapping`、`Compilation`：LLM 回應解析流程的相關詞條。
  - 詞條標題沿用本倉庫既有慣例（`Catalog`、`Basic Catalog`、`Surface`、`Action` 等協議術語保留英文原文）。

- **首頁（`docs/public/index.md`）**：底部「A2UI Composer」一節由 CopilotKit Widget Builder 介紹 + 截圖連結，改為官方 Composer 連結與新增的 Composer 文件連結（`./composer/index.md`）；與上游一致地移除 `A2UI-widget-builder.png` 圖片。

- **智慧體 UI 生態（`docs/public/introduction/agent-ui-ecosystem.md`）**：在「A2UI vs AG-UI / CopilotKit」一節中刪去「他們同時也貢獻了 A2UI Composer」以及指向 `../composer.md` 的連結（Composer 已改由 A2UI 專案自行維護）。

- **A2UI 中的 MCP Apps（`docs/public/guides/mcp-apps-in-a2ui.md`）**：內層 iframe 權限說明的禁用清單新增 `allow-top-navigation` 與 `allow-top-navigation-by-user-activation`；新增翻譯「防禦頂層視窗劫持」一條（說明省略這兩項可阻止 frame busting 攻擊重新導向 host 視窗）。

## `mkdocs.yaml` 與 `README.md`

- **`README.md`**：起步路徑表格中的 Composer 一列，移除 `Widget Builder`（`go.copilotkit.ai`）連結，並將 Composer 網址由 `https://a2ui-composer.ag-ui.com/` 更新為 `https://a2ui-project.github.io/composer/`。
- **`mkdocs.yaml`**：`A2UI Composer ⭐: composer.md` 單一項目改為與上游一致的兩層結構（`composer/index.md` + `Composer 整合: composer/composer_renderer_integration.md`）。

## 素材

- 新增（自上游複製，6 個）：`composer_workspace.png`、`composer_components_gallery.png`、`composer_editor_tooltip.png`、`composer_paperclip.png`、`composer_camera.png`、`composer_copy.png`
- 刪除：`A2UI-widget-builder.png`（引用它的文件已全部移除）

## 驗證

本機未安裝 `mkdocs-material`、`mkdocs-macros-plugin`、`mkdocs-mermaid2-plugin`，無法執行 `mkdocs build`。改以腳本核對下列項目，皆通過：

- `mkdocs.yaml` 中 `nav` 指向的檔案全部存在
- 兩篇新增 Composer 文件的相對連結與圖片路徑全部可解析
- 已刪除的 `composer.md` / `A2UI-widget-builder.png` 沒有任何殘留引用
- 兩篇新增文件的行內程式碼、URL、圖片 token 與上游逐一比對一致（唯一差異是 MDN 連結依語言切換為 `zh-TW`），程式碼區塊與上游逐位元組相同
- 詞彙表新增部分的標題數（6）、清單項目數（5）、段落數（15）、程式碼 token 數（8）與上游完全一致
- 6 個新增素材的 SHA-256 與上游一致

## 未處理範圍

- `docs/contributing/**`、`eval/`、頂層 `specification/`（JSON 規格）等，依既定方針完全排除。
- 本次 diff 以外的其他 `docs/public/**` 檔案未作更動。

## 已知待處理項目（建議後續處理）

- 上游的 `docs/public/index.md` 卡片區與 `docs/public/guides/a2ui-with-any-agent-framework.md` 中仍保留舊的 Composer 網址（`https://a2ui-composer.ag-ui.com/`）。這是上游本身的不一致，本次維持與上游同步，中文譯文未作更動；待上游統一後再一併更新。
- 連結檢查發現，本倉庫中大量指向上游原始碼路徑的相對連結（`../../../samples/...`、`../../../renderers/...`、`../specification/...` 等）在本倉庫內無法解析。這是因為本倉庫僅包含文件，屬於結構性問題，與本次 diff 無關（在 HEAD 時同樣存在）。這些相對路徑是 2026-08-01 同步時跟隨上游 `75d2a5d0` 的改動所引入；若要修正，建議統一替換為 `https://github.com/a2ui-project/a2ui/blob/main/...` 形式的絕對網址。
