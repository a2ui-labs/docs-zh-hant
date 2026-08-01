# Client 設定指南

使用適合你平台的 renderer，將 A2UI 整合到你的應用程式中。

## 渲染器

| 渲染器                                                        | 平台                | v0.8 | v0.9 | 狀態               |
| -------------------------------------------------------------- | ------------------- | ---- | ---- | ------------------ |
| **[React](../../../renderers/react)**                          | Web                 | ✅   | ✅   | ✅ 穩定            |
| **[Lit (Web Components)](../../../renderers/lit)**             | Web                 | ✅   | ✅   | ✅ 穩定            |
| **[Angular](../../../renderers/angular)**                      | Web                 | ✅   | ✅   | ✅ 穩定            |
| **[Flutter (GenUI SDK)](https://docs.flutter.dev/ai/genui)**   | Mobile/Desktop/Web  | ✅   | ✅   | ✅ 穩定            |
| **Jetpack Compose**                                             | Android             | —    | —    | 🚧 預計 2026 年第二季 |

更多資訊請見完整的 [A2UI Renderers](../reference/renderers.md) 與 [社群 A2UI Renderers](../ecosystem/renderers.md)。

## 元件目錄

元件目錄（component catalog）就是任意元件的集合。A2UI 提供了一份「Basic Catalog」，但我們預期你會加入自己的元件、共用函式庫，或完全用自己的元件取代基礎元件。

**重點是你的設計系統。** 你可以註冊任何元件與函式的集合，A2UI 都能與之運作。catalog 只是你的 agent 與 renderer 之間的契約。

若要了解如何定義符合你設計系統的 catalog，請參閱 [定義你自己的 Catalog](defining-your-own-catalog.md)。

## 共用 Web 函式庫

所有 Web renderer（Lit、Angular、React）都建立在共同基礎之上：**`@a2ui/web_core`**。這個函式庫提供訊息處理器、狀態管理，以及每個 Web renderer 都需要的資料繫結邏輯。各框架專屬的 renderer 都是在其上再疊加，只負責該框架的渲染層。

這代表 Web 平台之間的核心協定處理是一致的，只有元件渲染部分不同。

共用的 `web_core` 函式庫提供：

- **Message Processor**：管理 A2UI 狀態並處理傳入訊息。

## Web Components（Lit）

```bash
npm install @a2ui/lit @a2ui/web_core
```

安裝完成後，你就可以在應用程式中使用這個 renderer。Lit renderer 使用：

- **Message Processor**：包裝 A2UI 訊息處理器。
- **`<a2ui-surface>` 元件**：在你的應用中渲染 surfaces。
- **Lit Signals**：提供回應式狀態管理，讓 UI 可以自動更新。

**可參考的運作範例：** [Lit shell sample](../../../samples/client/lit/shell) —— 詳細執行說明請參閱其 README。

## Angular

```bash
npm install @a2ui/angular @a2ui/web_core
```

安裝完成後，你就可以在應用程式中使用這個 renderer。Angular renderer 提供：

- **`A2uiRendererService`**：管理 A2UI 訊息處理器與回應式模型的 service。
- **`a2ui-v09-component-host` 元件**：一個動態 component host，會從 surface 渲染 A2UI 元件。
- **`A2UI_RENDERER_CONFIG` token**：用來以 catalog 與 action handler 設定 renderer。

### 設定範例（v0.9）

A2UI 會針對各版本協定使用版本化 imports。對 v0.9 而言，請使用 `provideA2Ui` 來設定應用程式 providers：

```typescript
import {ApplicationConfig} from '@angular/core';
import {provideA2Ui, BasicCatalog} from '@a2ui/angular/v0_9';

export const appConfig: ApplicationConfig = {
  providers: [
    provideA2Ui({
      catalogs: [new BasicCatalog()],
      actionHandler: action => {
        console.log('Action dispatched:', action);
      },
    }),
  ],
};
```

#### 在 Action Handler 中做依賴注入

如果你的 `actionHandler` 需要注入其他依賴（例如在 action 被派發時呼叫某個 service），你可以傳一個工廠函式給 `provideA2Ui`。在這個工廠函式中，你可以使用 Angular 的 `inject()` 函式：

```typescript
import {ApplicationConfig, inject} from '@angular/core';
import {provideA2Ui, BasicCatalog} from '@a2ui/angular/v0_9';
import {MyActionDispatcherService} from './my-action-dispatcher.service';

export const appConfig: ApplicationConfig = {
  providers: [
    provideA2Ui(() => {
      const dispatcher = inject(MyActionDispatcherService);
      return {
        catalogs: [new BasicCatalog()],
        actionHandler: action => dispatcher.dispatch(action),
      };
    }),
  ],
};
```

**可參考的運作範例：** [Angular samples](../../../samples/client/angular)

### 串流

Angular client 預設使用串流 API。若要停用串流，請在啟動應用程式前，將 `ENABLE_STREAMING` 環境變數設為 `false`：

```bash
export ENABLE_STREAMING=false
yarn start restaurant
```

> [!NOTE]
> **套件管理工具說明：** 上面的 `yarn start` 指令僅適用於在 A2UI monorepo 儲存庫內執行範例應用程式。若是你自己一般的日常使用或此儲存庫之外的獨立專案，請使用你偏好的套件管理工具（例如 npm、pnpm）。

## React

```bash
npm install @a2ui/react @a2ui/web_core
```

React renderer 提供：

- **`MessageProcessor` 類別**：管理 A2UI 訊息處理器與回應式模型的類別。
- **`<A2UISurface>` 元件**：在你的 React 應用程式中渲染 A2UI surfaces
- **`useA2UI()` hook**：讓任何元件都能存取訊息處理器

**可參考的運作範例：** [React shell](../../../samples/client/react/shell)

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

使用 A2A 協定 client 的範例，請見 [samples/client/lit/shell/client.ts](../../../samples/client/lit/shell/client.ts)。

**請參考：** [Transports guide](../concepts/transports.md)

## 處理使用者操作

當使用者與 A2UI 元件互動時（例如點擊按鈕、提交表單等），client 會：

1. 從元件擷取 action 事件
2. 解析該 action 所需的資料 context
3. 將 action 傳送給 agent
4. 處理 agent 回傳的訊息

處理按鈕點擊與表單提交的範例，請參閱 [samples/client/lit/shell/app.ts](../../../samples/client/lit/shell/app.ts) 中 `#maybeRenderData` 裡的 `@a2uiaction` 事件處理器。

## 錯誤處理

常見需要處理的錯誤包括：

- **無效的 Surface ID**：在收到 `beginRendering`（v0.8）或 `createSurface`（v0.9）之前就引用了 surface
- **無效的 Component ID**：component ID 在同一個 surface 內必須唯一
- **無效的資料路徑**：檢查 data model 結構與 JSON Pointer 語法
- **Schema 驗證失敗**：確認訊息格式符合 A2UI 規格

處理通訊錯誤的範例，請參閱 [samples/client/lit/shell/app.ts](../../../samples/client/lit/shell/app.ts) 中 `#sendMessage` 裡的 `try...catch` 區塊。

## 下一步

- **[Quickstart](../quickstart.md)**：試試看示範應用程式
- **[主題與樣式](theming.md)**：自訂外觀與風格
- **[定義你自己的 Catalog](defining-your-own-catalog.md)**：擴充元件目錄
- **[Agent Development](agent-development.md)**：建立會產生 A2UI 的 agent
- **[Reference Documentation](../reference/messages.md)**：深入了解協定
