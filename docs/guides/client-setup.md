# Client 設定指南

使用適合你平台的 renderer，將 A2UI 整合到你的應用程式中。

## 渲染器

| 渲染器 | 平台 | v0.8 | v0.9 | 狀態 |
| ------------------------ | ------------------ | ---- | ---- | ----------------- |
| **[React](https://github.com/google/A2UI/tree/main/renderers/react)** | Web | ✅ | ❌ | ✅ 穩定 |
| **[Lit (Web Components)](https://github.com/google/A2UI/tree/main/renderers/lit)** | Web | ✅ | ✅ | ✅ 穩定 |
| **[Angular](https://github.com/google/A2UI/tree/main/renderers/angular)** | Web | ✅ | ✅ | ✅ 穩定 |
| **[Flutter (GenUI SDK)](https://docs.flutter.dev/ai/genui)** | Mobile/Desktop/Web | ✅ | ✅ | ✅ 穩定 |
| **SwiftUI** | iOS/macOS | — | — | 🚧 預計 2026 年第二季 |
| **Jetpack Compose** | Android | — | — | 🚧 預計 2026 年第二季 |

## 元件目錄

元件目錄就是元件的集合 - 可以是標準元件、自訂元件，或共用函式庫。**重點是你的設計系統。** 你可以註冊任何元件與函式的集合，A2UI 都能與之運作。catalog 只是 Agent 與 renderer 之間的契約。

若要擴充或取代標準目錄，請參閱 [自訂元件](custom-components.md)。

## 共用 Web 函式庫

所有 Web renderer（Lit、Angular、React）都建立在共同基礎之上：**`@a2ui/web-lib`**。這個函式庫提供訊息處理器、狀態管理，以及每個 Web renderer 都需要的資料繫結邏輯。各框架專屬的 renderer 都是在其上再疊加，只負責該框架的渲染層。

這代表 Web 平台之間的核心協定處理是一致的，只有元件渲染部分不同。

## Web Components（Lit）

> ⚠️ **Attention**
>
> Lit client library 目前尚未發布到 NPM。請在接下來幾天再回來確認。

```bash
npm install @a2ui/web-lib lit @lit-labs/signals
```

Lit renderer 使用：

- **訊息處理器**：管理 A2UI 狀態並處理傳入訊息
- **`<a2ui-surface>` 元件**：在你的應用中渲染 surfaces
- **Lit Signals**：提供回應式狀態管理，讓 UI 可以自動更新

TODO：新增經驗證過的設定範例。

**可參考的運作範例：** [Lit shell sample](https://github.com/google/a2ui/tree/main/samples/client/lit/shell)

## Angular

> ⚠️ **Attention**
>
> Angular client library 目前尚未發布到 NPM。請在接下來幾天再回來確認。

```bash
npm install @a2ui/angular @a2ui/web_core
```

Angular renderer 提供：

- **`A2uiRendererService`**：管理 A2UI 訊息處理器與回應式模型的 service。
- **`a2ui-v09-component-host` 元件**：一個動態 component host，會從 surface 渲染 A2UI 元件。
- **`A2UI_RENDERER_CONFIG` token**：用來以 catalog 與 action handler 設定 renderer。

### 設定範例（v0.9）

A2UI 會針對各版本協定使用版本化 imports。對 v0.9 而言，請按如下方式設定應用程式 providers：

```typescript
import { ApplicationConfig } from '@angular/core';
import {
  A2UI_RENDERER_CONFIG,
  A2uiRendererService,
  minimalCatalog
} from '@a2ui/angular/v0_9';

export const appConfig: ApplicationConfig = {
  providers: [
    {
      provide: A2UI_RENDERER_CONFIG,
      useValue: {
        catalogs: [minimalCatalog],
        actionHandler: (action) => {
          console.log('Action dispatched:', action);
        }
      }
    },
    A2uiRendererService
  ]
};
```

**可參考的運作範例：** [Angular v0.9 Explorer](https://github.com/google/a2ui/tree/main/renderers/angular/a2ui_explorer)

## React

```bash
npm install @a2ui/react @a2ui/web-lib
```

React renderer 提供：

- **`<A2UISurface>` 元件**：在你的 React 應用程式中渲染 A2UI surfaces
- **`useA2UI()` hook**：讓任何元件都能存取訊息處理器
- **`MessageProcessor` 類別**：處理傳入的 A2UI 訊息（與其他 Web renderer 共用）

**可參考的運作範例：** [React shell](https://github.com/google/A2UI/tree/main/samples/client/react/shell)

## Flutter（GenUI SDK）

```bash
flutter pub add flutter_genui
```

Flutter 使用 GenUI SDK 提供原生 A2UI 渲染。

**文件：** [GenUI SDK](https://docs.flutter.dev/ai/genui) | [GitHub](https://github.com/flutter/genui) | [GenUI Flutter 套件中的 README](https://github.com/flutter/genui/blob/main/packages/genui/README.md#getting-started-with-genui)

## 連接到 Agents

你的 client 應用程式需要：

1. **從 agent 接收 A2UI 訊息**（透過 transport）
2. **使用 Message Processor 處理訊息**
3. **把使用者操作送回 agent**

常見的傳輸選項包括：

- **Server-Sent Events (SSE)**：從 server 到 client 的單向串流
- **WebSockets**：雙向即時通訊
- **A2A Protocol**：標準化的 agent-to-agent 通訊，並支援 A2UI

TODO：新增傳輸實作範例。

**請參考：** [Transports guide](../concepts/transports.md)

## 處理使用者操作

當使用者與 A2UI 元件互動時（例如點擊按鈕、提交表單等），client 會：

1. 從元件擷取 action 事件
2. 解析該 action 所需的資料 context
3. 將 action 傳送給 agent
4. 處理 agent 回傳的訊息

TODO：新增 action 處理範例。

## 錯誤處理

常見需要處理的錯誤包括：

- **無效的 Surface ID**：在收到 `beginRendering`（v0.8）或 `createSurface`（v0.9）之前就引用了 surface
- **無效的 Component ID**：component ID 在同一個 surface 內必須唯一
- **無效的資料路徑**：檢查 data model 結構與 JSON Pointer 語法
- **Schema 驗證失敗**：確認訊息格式符合 A2UI 規格

TODO：新增錯誤處理範例。

## 下一步

- **[Quickstart](../quickstart.md)**：試試看示範應用程式
- **[主題與樣式](theming.md)**：自訂外觀與風格
- **[自訂元件](custom-components.md)**：擴充元件目錄
- **[Agent Development](agent-development.md)**：建立會產生 A2UI 的 agent
- **[Reference Documentation](../reference/messages.md)**：深入了解協定
