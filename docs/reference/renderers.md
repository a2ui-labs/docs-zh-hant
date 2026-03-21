# 渲染器（客戶端函式庫）

渲染器會把 A2UI JSON 訊息轉換成各平台的原生 UI 元件。

[智慧體](agents.md) 負責生成 A2UI 訊息，
[傳輸層](../concepts/transports.md) 負責把訊息送到客戶端。
客戶端渲染器函式庫必須緩衝並處理 A2UI 訊息、實作 A2UI 生命週期，並渲染 surface（元件）。

你有很大的彈性，可以把自訂元件帶進現有渲染器，也可以為自己的 UI 元件框架打造專屬渲染器。

## 維護中的渲染器

| 渲染器 | 平台 | v0.8 | v0.9 | 連結 |
|----------|----------|------|------|-------|
| **React** | Web | ✅ 穩定 | ❌ | [程式碼](https://github.com/google/A2UI/tree/main/renderers/react) |
| **Lit（Web Components）** | Web | ✅ 穩定 | ✅ 穩定 | [程式碼](https://github.com/google/A2UI/tree/main/renderers/lit) |
| **Angular** | Web | ✅ 穩定 | ✅ 穩定 | [程式碼](https://github.com/google/A2UI/tree/main/renderers/angular) |
| **Flutter（GenUI SDK）** | 行動端 / 桌面端 / Web | ✅ 穩定 | ✅ 穩定 | [文件](https://docs.flutter.dev/ai/genui) · [程式碼](https://github.com/flutter/genui) |
| **SwiftUI** | iOS / macOS | — | 🚧 預計 Q2 | — |
| **Jetpack Compose** | Android | — | 🚧 預計 Q2 | — |

更多資訊請參閱 [路線圖](../roadmap.md)。

## 生態系渲染器

社群也正在為更多平台打造 A2UI 渲染器：

- **[json-render](https://json-render.dev/docs/a2ui)** — Vercel 的 React 函式庫，透過 Zod schema 渲染 A2UI 目錄（[比較文章](https://dipjyotimetia.medium.com/vercels-json-render-vs-google-s-a2ui-the-head-to-head-6f213cf1a23b)）
- **[A2UI-Android](https://github.com/lmee/A2UI-Android)** — 社群版 Jetpack Compose 渲染器，提供 20+ 個元件（約 15 ⭐，v0.8）
- **[a2ui-react-native](https://github.com/sivamrudram-eng/a2ui-react-native)** — 面向 iOS/Android 的 React Native 渲染器（約 9 ⭐，v0.8）

更多社群專案與提交方式，請參閱 **[完整生態系渲染器清單](../ecosystem/renderers.md)**。

## 渲染器如何運作

```
A2UI JSON → 渲染器 → 原生元件 → 你的應用
```

1. **接收** 來自傳輸層的 A2UI 訊息
2. **解析** JSON，並依據 schema 進行驗證
3. **渲染** 成平台原生元件
4. **套用樣式**，以符合你的應用主題

## 使用渲染器

若要把 A2UI 整合進你的應用，可以依照你選擇的渲染器對應的安裝指南開始：

- **[React](../guides/client-setup.md#react)**
- **[Lit（Web Components）](../guides/client-setup.md#web-components-lit)**
- **[Angular](../guides/client-setup.md#angular)**
- **[Flutter（GenUI SDK）](../guides/client-setup.md#flutter-genui-sdk)**

## 建立自己的渲染器

想為你的平台打造一個渲染器嗎？

- 查看 [路線圖](../roadmap.md) 中規劃中的框架。
- 參考現有渲染器的實作模式。
- 閱讀 [渲染器開發指南](../guides/renderer-development.md) 以了解實作細節。

### 主要需求：

- 解析 A2UI JSON 訊息，特別是 adjacency list 格式
- 將 A2UI 元件對映到原生元件
- 處理資料繫結與生命週期事件
- 處理一串增量式 A2UI 訊息，以建立並更新 UI
- 支援由伺服端主動發起的更新
- 支援使用者操作

### 下一步

- **[客戶端設定指南](../guides/client-setup.md)**：整合說明
- **[快速開始](../quickstart.md)**：試試 Lit 渲染器
- **[元件參考](components.md)**：了解需要支援哪些元件
