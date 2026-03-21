# A2UI Renderer 實作指南

本文說明 A2UI 協定之新 renderer 實作所需的功能。這份文件是提供給正在建立新 renderer 的開發者，例如 React、Flutter、iOS 等。

!!! info "版本說明"
    本指南主要描述 v0.8 的訊息流程。v0.9 重新命名了幾個訊息（`surfaceUpdate` → `updateComponents`、`dataModelUpdate` → `updateDataModel`、`beginRendering` → `createSurface`），並採用更扁平的元件格式。詳情請參閱 [v0.9 specification](../specification/v0.9-a2ui.md)。

## Web 渲染器：使用 `@a2ui/web-lib`（`web_core`）

如果你正在為 Web（React、Vue、Svelte 等）建立 renderer，就不需要從零實作訊息處理、狀態管理或 schema 驗證。**[`@a2ui/web-lib`](https://github.com/google/A2UI/tree/main/renderers/web_core)** 套件（`web_core`）提供了所有維護中 Lit、Angular 與 React renderer 共用的框架無關邏輯。

### `web_core` 提供什麼

| 模組 | 功能 |
|--------|-------------|
| **`MessageProcessor`** | 處理 A2UI JSONL 串流、派送訊息、管理 surface 生命週期 |
| **`SurfaceModel` / `SurfaceGroupModel`** | 管理 surfaces、元件與 data model 的狀態 |
| **`DataModel` / `DataContext`** | 資料繫結解析、依路徑查找、模板清單渲染 |
| **`ComponentModel`** | 元件樹狀態、鄰接清單 → 樹狀結構的解析 |
| **`Types & Schemas`** | 所有 A2UI 元件、primitive、顏色、樣式與 JSON schema 驗證的 TypeScript 型別 |
| **Expression parser** | client 端函式求值（v0.9） |

### 維護中的 renderer 如何使用它

這三個 Web renderer 都遵循同樣的模式 - `web_core` 處理協定，renderer 處理 UI：

```typescript
// Types — shared across all renderers
import type * as Types from '@a2ui/web_core/types/types';
import type * as Primitives from '@a2ui/web_core/types/primitives';

// v0.8: Message processing and state
import { A2uiMessageProcessor } from '@a2ui/web_core/data/model-processor';

// v0.9: Message processing, surfaces, catalogs
import { MessageProcessor } from '@a2ui/web_core/v0_9';
import { SurfaceModel } from '@a2ui/web_core/v0_9';

// Styles and layout helpers
import * as Styles from '@a2ui/web_core/styles/index';
```

你的 renderer 只需要：

1. **把 A2UI 元件型別對應到你的框架元件**（例如 `Text` → `<p>`、`Button` → `<button>`）
2. **訂閱 `web_core` 的狀態變更**，並重新渲染
3. **透過 `MessageProcessor` 把使用者操作往回傳送**

請參考 [React renderer](https://github.com/google/A2UI/tree/main/renderers/react)、[Lit renderer](https://github.com/google/A2UI/tree/main/renderers/lit) 與 [Angular renderer](https://github.com/google/A2UI/tree/main/renderers/angular) 來看這個模式的實作範例。

### 版本支援

`web_core` 同時輸出 v0.8 與 v0.9 API：

- `@a2ui/web_core/v0_8` 或 `@a2ui/web_core`（預設）— 穩定的 v0.8
- `@a2ui/web_core/v0_9` — 支援 `createSurface`、自訂 catalog、client 端函式的 v0.9
- `@a2ui/web_core/v0_9/basic_catalog` — v0.9 基礎 catalog 的表達式解析器與內建函式

!!! tip "從 `web_core` 開始"
    如果不使用 `web_core` 就建立 Web renderer，等於要重新實作約 3,000 行的訊息處理、狀態管理與 schema 驗證。除非你有明確理由要偏離，否則請直接使用它。

---

## I. 核心協定實作檢查清單

這一節詳細說明 A2UI 協定的基本機制。要讓 renderer 符合規範，就必須實作這些系統，才能成功解析 server 串流、管理狀態並處理使用者互動。

### 訊息處理與狀態管理

- **JSONL 串流解析**：實作一個能逐行讀取串流回應的 parser，將每一行解碼成獨立的 JSON 物件。
- **訊息分派器**：建立分派器來辨識訊息類型（`beginRendering`、`surfaceUpdate`、`dataModelUpdate`、`deleteSurface`），並將它路由到正確的 handler。
- **Surface 管理**：
  - 實作一個資料結構來管理多個 UI surface，每個都以其 `surfaceId` 為鍵。
  - 處理 `surfaceUpdate`：將元件新增或更新到指定 surface 的元件緩衝區。
  - 處理 `deleteSurface`：移除指定 surface 及其所有相關資料與元件。
- **元件緩衝（鄰接清單）**：
  - 對每個 surface 維護一個元件緩衝區（例如 `Map<String, Component>`），用來依 `id` 儲存所有元件定義。
  - 在渲染時，能透過解析容器元件中的 `id` 參照（`children.explicitList`、`child`、`contentChild` 等）重建 UI 樹。
- **資料模型儲存**：
  - 對每個 surface 維護一個獨立的 data model 儲存區（例如 JSON 物件或 `Map<String, any>`）。
  - 處理 `dataModelUpdate`：更新指定 `path` 的 data model。`contents` 會是鄰接清單格式（例如 `[{ "key": "name", "valueString": "Bob" }]`）。

### 渲染邏輯

- **漸進式渲染控制**：
  - 將所有進來的 `surfaceUpdate` 與 `dataModelUpdate` 訊息先緩衝起來，不要立刻渲染。
  - 處理 `beginRendering`：這則訊息是明確的初次渲染信號，並設定 root component ID。
    - 從指定的 `root` component ID 開始渲染。
    - 如果有提供 `catalogId`，要確保使用對應的 component catalog（若未提供則預設使用標準 catalog）。
    - 套用這則訊息中提供的任何全域 `styles`（例如 `font`、`primaryColor`）。
- **資料繫結解析**：
  - 為元件屬性中找到的 `BoundValue` 物件實作解析器。
  - 如果只提供 `literal*` 值（`literalString`、`literalNumber` 等），就直接使用它。
  - 如果只提供 `path`，就把它解析到 surface 的 data model。
  - 如果同時提供 `path` 與 `literal*`，先用字面值更新 `path` 對應的 data model，然後把元件屬性繫結到該 `path`。
- **動態清單渲染**：
  - 對於具有 `children.template` 的容器，迭代 `template.dataBinding` 找到的資料清單（會解析成 data model 中的一個 list）。
  - 對資料清單中的每個項目，渲染 `template.componentId` 指定的元件，並讓該項目的資料可在 template 內進行相對資料繫結。

### Client-to-Server 通訊

- **事件處理**：
  - 當使用者與具有 `action` 定義的元件互動時，組裝一個 `userAction` payload。
  - 將 `action.context` 中的所有資料繫結，根據 data model 進行解析。
  - 將完整的 `userAction` 物件傳送到 server 的事件處理端點。
- **Client 能力回報**：
  - 在送往 server 的**每一則** A2A 訊息中（作為 metadata 的一部分），都要包含 `a2uiClientCapabilities` 物件。
  - 這個物件應宣告 client 支援的 component catalog，透過 `supportedCatalogIds` 表示（例如包含標準 0.8 catalog 的 URI）。
  - 如果 server 支援，也可以選擇提供 `inlineCatalogs`，用於臨時的自訂元件定義。
- **錯誤回報**：實作一個機制，把 `error` 訊息送到 server，以回報任何 client 端錯誤（例如資料繫結失敗、未知元件型別）。

## II. 標準元件目錄檢查清單

為了在各平台上維持一致的使用者體驗，A2UI 定義了一組標準元件。你的 client 應該把這些抽象定義對應到各自的原生 UI widget。

### 基本內容

- **Text**：渲染文字內容。必須支援 `text` 的資料繫結，以及用於樣式的 `usageHint`（h1-h5、body、caption）。
- **Image**：從 URL 渲染圖片。必須支援 `fit`（cover、contain 等）與 `usageHint`（avatar、hero 等）屬性。
- **Icon**：從 catalog 中指定的標準圖示集合渲染預定義圖示。
- **Video**：為指定 URL 渲染影片播放器。
- **AudioPlayer**：為指定 URL 渲染音訊播放器，並可選擇附帶描述。
- **Divider**：渲染視覺分隔線，支援 `horizontal` 與 `vertical` 兩種軸向。

### 版面配置與容器

- **Row**：水平排列子元件。必須支援 `distribution`（justify-content）與 `alignment`（align-items）。子元件可以有 `weight` 屬性來控制 flex-grow 行為。
- **Column**：垂直排列子元件。必須支援 `distribution` 與 `alignment`。子元件可以有 `weight` 屬性來控制 flex-grow 行為。
- **List**：渲染可捲動的項目清單。必須支援 `direction`（`horizontal`/`vertical`）與 `alignment`。
- **Card**：一個會在視覺上把子內容分組的容器，通常帶有邊框、圓角和/或陰影。只有一個 `child`。
- **Tabs**：顯示一組分頁的容器。包含 `tabItems`，每個項目都有 `title` 與 `child`。
- **Modal**：出現在主要內容上方的對話框。它由 `entryPointChild`（例如按鈕）觸發，啟用時會顯示 `contentChild`。

### 互動與輸入元件

- **Button**：可點擊元件，用來觸發 `userAction`。必須能包含一個 `child` 元件（通常是 Text 或 Icon），並可依 `primary` 布林值調整樣式。
- **CheckBox**：可切換的核取方塊，反映布林值。
- **TextField**：文字輸入欄位。必須支援 `label`、`text`（值）、`textFieldType`（`shortText`、`longText`、`number`、`obscured`、`date`）與 `validationRegexp`。
- **DateTimeInput**：專門用來選擇日期和/或時間的輸入元件。必須支援 `enableDate` 與 `enableTime`。
- **MultipleChoice**：用來從清單（`options`）中選擇一個或多個選項的元件。必須支援 `maxAllowedSelections`，並將 `selections` 繫結到清單或單一值。
- **Slider**：用來從定義好的範圍（`minValue`、`maxValue`）中選擇數值（`value`）的滑桿。
