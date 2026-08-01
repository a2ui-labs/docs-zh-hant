# A2UI Renderer 實作指南

本文說明 A2UI 協定之新 renderer 實作所需的功能。這份文件是提供給正在建立新 renderer 的開發者，例如 React、Flutter、iOS 等。

!!! info "多版本指南"
    本指南提供 v0.8、v0.9.1（目前的正式版本）以及 v1.0（候選版本）的實作檢查清單。請使用下方分頁選擇你要開發的目標版本。

## Web 渲染器：使用 `@a2ui/web_core`（`web_core`）

如果你正在為 Web（React、Vue、Svelte 等）建立 renderer，就不需要從零實作訊息處理、狀態管理或 schema 驗證。**[`@a2ui/web_core`](../../../renderers/web_core)** 套件（`web_core`）提供了所有維護中 Lit、Angular 與 React renderer 共用的框架無關邏輯。

### `web_core` 提供什麼

| 模組 | 功能 |
|--------|-------------|
| **`MessageProcessor`** | 處理 A2UI JSONL 串流、派送訊息、管理 surface 生命週期 |
| **`SurfaceModel` / `SurfaceGroupModel`** | 管理 surfaces、元件與 data model 的狀態 |
| **`DataModel` / `DataContext`** | 資料繫結解析、依路徑查找、模板清單渲染 |
| **`ComponentModel`** | 元件樹狀態、鄰接清單 → 樹狀結構的解析 |
| **`Types & Schemas`** | 所有 A2UI 元件、primitive、顏色、樣式與 JSON schema 驗證的 TypeScript 型別 |
| **Expression parser** | Client 端函式求值（v0.9+） |

### 維護中的 renderer 如何使用它

這三個 Web renderer 都遵循同樣的模式 - `web_core` 處理協定，renderer 處理 UI：

```typescript
// Types — shared across all renderers
import type * as Types from '@a2ui/web_core/types/types';
import type * as Primitives from '@a2ui/web_core/types/primitives';

// v0.8: Message processing and state
import {A2uiMessageProcessor} from '@a2ui/web_core/data/model-processor';

// v0.9.1 / v1.0: Message processing, surfaces, catalogs
import {MessageProcessor} from '@a2ui/web_core/v0_9';
import {SurfaceModel} from '@a2ui/web_core/v0_9';

// Styles and layout helpers
import * as Styles from '@a2ui/web_core/styles/index';
```

你的 renderer 只需要：

1. **把 A2UI 元件型別對應到你的框架元件**（例如 `Text` → `<p>`、`Button` → `<button>`）
2. **訂閱 `web_core` 的狀態變更**，並重新渲染
3. **透過 `MessageProcessor` 把使用者操作往回傳送**

請參考 [React renderer](../../../renderers/react)、[Lit renderer](../../../renderers/lit) 與 [Angular renderer](../../../renderers/angular) 來看這個模式的實作範例。

### 版本支援

`web_core` 依版本個別輸出對應的 API 組合：

- `@a2ui/web_core/v0_8` — 穩定的 v0.8
- `@a2ui/web_core/v0_9` — 支援 `createSurface`、自訂 catalog、client 端函式的 v0.9 / v0.9.1
- `@a2ui/web_core/v1_0` — 支援候選版本 v1.0，包含 RPC action 回應

!!! tip "從 `web_core` 開始"
    如果不使用 `web_core` 就建立 Web renderer，等於要重新實作約 3,000 行的訊息處理、狀態管理與 schema 驗證。除非你有明確理由要偏離，否則請直接使用它。

---

## I. 核心協定實作檢查清單

這一節詳細說明 A2UI 協定的基本機制。要讓 renderer 符合規範，就必須實作這些系統，才能成功解析 server 串流、管理狀態並處理使用者互動。

=== "v0.8"

    - **JSONL 串流解析**：逐行讀取串流回應，將每一行解碼成獨立的 JSON 物件。
    - **訊息分派器**：辨識訊息類型（`beginRendering`、`surfaceUpdate`、`dataModelUpdate`、`deleteSurface`），並路由到正確的 handler。
    - **Surface 管理**：
        - 以 `surfaceId` 作為 surface 的鍵值。
        - 處理 `surfaceUpdate`：在 surface 的緩衝區中新增／更新元件。
        - 處理 `deleteSurface`：移除該 surface 及其所有相關資料／元件。
    - **元件緩衝**：
        - 為每個 surface 維護一個元件緩衝區（例如 `Map<String, Component>`）。
        - 透過解析 `id` 參照（`children.explicitList`、`child`、`contentChild` 等）重建 UI 樹。
    - **資料模型儲存**：
        - 為每個 surface 維護 data model 狀態。
        - 處理 `dataModelUpdate`：使用鄰接清單表示法（`[{ "key": "name", "valueString": "Bob" }]`）更新指定路徑的值。
    - **漸進式渲染**：
        - 在收到 `beginRendering` 之前，先緩衝所有更新。
        - 收到 `beginRendering` 後，從指定的 `root` ID 開始渲染，並套用主題樣式。
    - **資料繫結解析**：
        - 使用 `literalString` / `literalNumber` / `path` 解析 `BoundValue` 物件。
    - **動態清單**：
        - 對於 `children.template`，迭代 `template.dataBinding` 所指的資料清單，並使用 `template.componentId` 渲染元件。
    - **Client-to-Server 通訊**：
        - 將包含已解析路徑 context 的 `userAction` 傳送到 server。
        - 在傳輸層 metadata 中包含 `a2uiClientCapabilities`。

=== "v0.9.1 (Current)"

    - **JSONL 串流解析**：逐行讀取串流回應，將每一行解碼成獨立的 JSON 物件。
    - **訊息分派器**：辨識訊息類型（`createSurface`、`updateComponents`、`updateDataModel`、`deleteSurface`），並路由到正確的 handler。
    - **MIME 型別驗證**：根據標準化的 `application/a2ui+json` MIME 型別攔截 payload。
    - **Surface 管理**：
        - 以 `surfaceId` 作為 surface 的鍵值。
        - 處理 `createSurface`：建立 surface、綁定 `catalogId`，並註冊 `theme` 與 `sendDataModel`。
        - 處理 `updateComponents`：使用 `"component": "Type"` 判別欄位，以扁平格式新增／更新元件。
        - 處理 `deleteSurface`：移除該 surface 及其所有相關資料／元件。
    - **元件緩衝**：
        - 為每個 surface 維護一個元件緩衝區（例如 `Map<String, Component>`）。
        - 透過解析容器元件 `children` 陣列或 `child` 欄位中的 ID 參照來重建 UI 樹。
    - **資料模型儲存**：
        - 為每個 surface 維護 data model 狀態。
        - 處理 `updateDataModel`：以標準 JSON 物件並採 upsert 語意來更新指定路徑的資料。
    - **漸進式渲染**：
        - 一旦在 `updateComponents` 中解析到有效的 root 元件（ID 為 `root`），就立即渲染，不需要等待特殊的渲染信號。
    - **資料繫結解析**：
        - 解析簡化後的 bound value（可以是字面值，或是 `{"path": "..."}`）。
    - **動態清單**：
        - 對於子項目 template，迭代 `path` 所指的資料陣列，並渲染 `componentId` 指定的 template。
    - **Client 端函式**：
        - 求值已註冊、由 catalog 定義的函式（例如 `formatString` 字串插值）。
    - **Client-to-Server 通訊**：
        - 傳送包含已解析路徑 context 的 `action`（取代 `userAction`）。
        - 如果有要求 `sendDataModel`，就自動在 metadata 中包含完整的 client 端 data model。
        - 若 schema 驗證失敗，向 server 傳送結構化的 `ValidationFailed` 錯誤訊息。

=== "v1.0 (Candidate)"

    包含 v0.9.1 的所有需求，並加上以下擴充：
    - **Surface 屬性**：
        - 處理帶有 `surfaceProperties`（取代 `theme`）的 `createSurface`。surface schema 內部不再支援自訂主要品牌色彩。
    - **Action 回應（RPC）**：
        - 處理來自 server、包含 `actionId` 以及回傳 `value` 或 `error` 的 `actionResponse` 訊息。
    - **Client-to-Server 通訊**：
        - 在 `action` payload 中產生並包含 `actionId`。
        - 當 client 預期會收到回應時，於 action 上支援 `wantResponse: true`。
        - 若使用 A2A，傳送給 server 的每一則 A2A `Message` 都必須在其 `metadata` 欄位中包含 `a2uiClientCapabilities` 物件。
    - **能力**：
        - 在能力交換過程中，以 `surfaceProperties` 取代 `theme` 對外揭露。

---

## II. 標準元件目錄檢查清單

為了在各平台上維持一致的使用者體驗，A2UI 定義了一組標準元件。你的 client 應該把這些抽象定義對應到各自的原生 UI widget。

=== "v0.8"

    - **Text**：渲染文字。支援 `usageHint`（h1-h5、body、caption）。
    - **Image**：從 URL 渲染圖片。支援 `fit` 與 `usageHint`（avatar、hero 等）。
    - **Icon**：渲染系統圖示。
    - **Video**：渲染影片播放器。
    - **AudioPlayer**：渲染帶有描述的音訊播放器。
    - **Divider**：渲染水平／垂直分隔線。
    - **Row** / **Column**：水平／垂直排列子元件。支援 `distribution` 與 `alignment`。支援子元件的 `weight`。
    - **List**：渲染可捲動的清單。
    - **Card**：具有圓角與陰影的方塊版面。
    - **Tabs**：使用 `tabItems` 的分頁導覽。
    - **Modal**：由 `entryPointChild` 觸發、顯示 `contentChild` 的彈出視窗。
    - **Button**：觸發 `userAction` 的可點擊按鈕。支援 `primary` 樣式變化。
    - **CheckBox**：布林值核取方塊。
    - **TextField**：輸入欄位，支援 `label`、`textFieldType`（`shortText`、`longText` 等）與 `validationRegexp`。
    - **MultipleChoice**：支援 `options`、`maxAllowedSelections`，以及單選／多選數值。
    - **Slider**：使用 `minValue`、`maxValue` 支援數值範圍。

=== "v0.9.1 (Current)"

    - **Text**：渲染文字。支援 `variant`（取代 `usageHint`）。
    - **Image**：從 URL 渲染圖片。支援 `fit` 與 `variant`。
    - **Icon**：渲染系統圖示。
    - **Video**：渲染影片播放器。
    - **AudioPlayer**：渲染帶有描述的音訊播放器。
    - **Divider**：渲染水平／垂直分隔線。
    - **Row** / **Column**：水平／垂直排列子元件。支援 `justify` 與 `align`。支援子元件的 `weight`。
    - **List**：渲染可捲動的清單。
    - **Card**：具有圓角與陰影的方塊版面。
    - **Tabs**：使用 `tabs` 的分頁導覽。
    - **Modal**：由 `trigger` 觸發、顯示 `content` 的彈出視窗。
    - **Button**：觸發 `action` 的可點擊按鈕。支援 `variant`（primary、borderless）。
    - **CheckBox**：布林值核取方塊。
    - **TextField**：輸入欄位，支援 `label`、`value`（取代 `text`）、`variant`（`shortText`、`longText` 等）與 `checks`（驗證函式）。
    - **ChoicePicker**：（取代 MultipleChoice）支援 `options` 與 `variant`（`mutuallyExclusive`、`multipleSelection`）。
    - **Slider**：使用 `min`、`max`（取代 `minValue`、`maxValue`）支援數值範圍。

=== "v1.0 (Candidate)"

    包含 v0.9.1 的所有元件，並加上以下擴充：
    - **Video**：支援 `posterUrl` 屬性以顯示預覽圖片。
    - **TextField**：支援 `placeholder` 屬性。
