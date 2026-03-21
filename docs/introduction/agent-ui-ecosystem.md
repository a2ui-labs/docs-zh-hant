# A2UI 與其他方案相比如何？

智慧體 UI 領域發展很快。以下說明 A2UI 與其他主要做法之間的關係。

## 一覽

| | **A2UI** | **MCP Apps** | **AG UI** |
|---|---|---|---|
| **做法** | 宣告式元件藍圖 | 透過 `ui://` URI 提供預先建好的 HTML | 連接後端與前端的高頻寬協議 |
| **渲染** | 原生元件（Angular、Flutter、Lit 等） | 沙箱化 `iframe` | 由開發者決定（任何框架） |
| **樣式** | 由宿主應用控制，沿用設計系統 | 彼此隔離，由遠端伺服器控制外觀 | 由開發者控制，屬於宿主應用的一部分 |
| **安全性** | 宣告式資料，不執行程式碼 | 透過沙箱 iframe 隔離 | 在你自己的應用中執行受信任程式碼 |
| **多智慧體** | ✅ 可跨越信任邊界 | ✅ 可連接多個 MCP 伺服器 | ⚠️ 主要偏向單一智慧體 |
| **跨平台** | ✅ Web、行動端、桌面、原生 | ⚠️ 主要偏向 Web（iframe） | ✅ 協議本身與框架無關 |
| **LLM 生成** | ✅ 為串流輸出而設計 | ❌ 由伺服器預先建好 | ✅ 可透過 A2UI 整合 |
| **規範** | 開放協議（Apache 2.0） | [MCP extension](https://modelcontextprotocol.io/docs/extensions/apps)（SEP-1865） | 開源（由 CopilotKit 維護） |

## A2UI vs MCP Apps

[MCP Apps](https://blog.modelcontextprotocol.io/posts/2025-11-21-mcp-apps/) 把 UI 視為一種 **resource**：伺服器透過 `ui://` URI 提供預先建好的 HTML，再放進沙箱 iframe 中渲染。遠端整合方掌控全部內容與外觀，而設定則透過工具呼叫完成。A2UI 則採取 **宣告式 UI** 做法：智慧體送出元件藍圖，但樣式、主題，以及元件如何配置與渲染，仍由宿主應用控制。若你希望由伺服器完全掌控 UI 體驗，請選 MCP Apps；若你想要能自然融入自己應用的動態、跨平台 UI，請選 A2UI。

## A2UI vs AG UI / CopilotKit

[AG UI](https://ag-ui.com/) 是一種 **傳輸協議**，用來把智慧體後端與前端以即時狀態同步的方式連接起來。A2UI 則是一種 **UI 格式**，也就是描述要渲染什麼內容的 payload。兩者是互補關係：把 AG UI 當成管道，把 A2UI 當成內容即可。AG UI 是 [CopilotKit](https://copilotkit.ai) 團隊的專案，他們同時也貢獻了 [A2UI Composer](../composer.md)。AG UI 從第一天起就支援 A2UI。

## A2UI vs ChatKit（OpenAI）

[ChatKit](https://platform.openai.com/docs/guides/chatkit) 在 OpenAI 生態系中提供高度整合的體驗。A2UI 與 ChatKit 共享部分設計理念，兩者都定義了一組基礎元件，並使用可設定、宣告式的抽象層。A2UI 的重點在於 **平台無關**，它是為了那些要在 Web、行動端與桌面端自行打造智慧體介面的開發者而設計，也適用於需要讓智慧體跨越信任邊界渲染 UI 的多智慧體系統。

## 搭配使用

這些做法是互補的，不是互相競爭的：

- **A2UI + AG UI** — 以 AG UI 作為 transport，以 A2UI 作為生成式 UI 格式
- **A2UI + A2A** — 在多智慧體系統中，透過 [A2A protocol](../concepts/transports.md) 傳送 A2UI 訊息
- **A2UI + MCP** — 即將推出的橋接層，會讓 MCP 伺服器除了 HTML resource 之外，也能提供 A2UI 藍圖

## 延伸閱讀

- [什麼是 A2UI？](what-is-a2ui.md) — 協議總覽
- [Transports](../concepts/transports.md) — A2UI 訊息如何在智慧體與客戶端之間傳遞
- [A2UI 在哪裡被使用？](../ecosystem/a2ui-in-the-world.md) — 案例研究與採用情況
