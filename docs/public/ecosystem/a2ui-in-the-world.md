# A2UI 在真實世界中的應用

A2UI 正被 Google 與合作夥伴組織中的多個團隊採用，用來打造下一代智慧體驅動應用。以下是真實世界中 A2UI 產生影響的一些案例。

## 生產環境部署案例

### Google Opal：讓每個人都能建立 AI Mini-Apps

[Opal](http://opal.google) 讓數十萬人可以透過自然語言建立、編輯與分享 AI mini-apps，不需要寫程式。

**Opal 如何使用 A2UI：**

Google 的 Opal 團隊從一開始就是 **A2UI 的核心貢獻者**。他們利用 A2UI 建構支撐 Opal AI mini-apps 的動態生成式 UI 系統。

- **快速原型設計**：快速建立並測試新的 UI 模式
- **使用者生成應用**：任何人都能建立具有自訂 UI 的應用
- **動態介面**：UI 會自動適應不同使用情境

> 「A2UI 是我們工作的基礎。它讓我們能以全新的方式讓 AI 主導使用者體驗，而不會被固定的前端所限制。它的宣告式特性與對安全性的重視，使我們能夠快速且安全地進行實驗。」
>
> **— Dimitri Glazkov**, Principal Engineer, Opal Team

**進一步了解：** [opal.google](http://opal.google)

---

### Gemini Enterprise：企業用自訂智慧體

Gemini Enterprise 讓企業可以打造強大的自訂 AI 智慧體，以符合特定工作流程與資料需求。

**Gemini Enterprise 如何使用 A2UI：**

A2UI 正被整合進 Gemini Enterprise，讓企業智慧體能夠在商務應用中渲染 **豐富且可互動的 UI**，不再只停留在簡單文字回應，而是能引導員工完成複雜工作流程。

- **資料輸入表單**：由 AI 生成、用於結構化資料蒐集的表單
- **審批儀表板**：用於審查與核準流程的動態 UI
- **工作流程自動化**：為複雜任務生成逐步介面
- **企業客製 UI**：符合特定產業需求的客製介面

> 「我們的客戶不只需要智慧體回答問題；他們還需要智慧體引導員工完成複雜的工作流程。A2UI 將讓基於 Gemini Enterprise 開發的開發者，能讓智慧體為任何任務生成所需的動態自訂 UI，從資料輸入表單到審批儀表板，進而大幅加速工作流程自動化。」
>
> **— Fred Jabbour**, Product Manager, Gemini Enterprise

> **備註：** A2UI 在 Gemini Enterprise 中的渲染，目前僅支援透過 Agent-to-Agent（A2A）路徑（`a2aAgentDefinition`）註冊的自助式（self-serve）智慧體。部署在 Vertex AI Agent Engine 上、並透過 `adkAgentDefinition` 路徑註冊的受管理智慧體，目前尚不支援 A2UI 渲染。

**進一步了解：** [Gemini Enterprise](https://cloud.google.com/gemini)

---

### Flutter GenUI SDK：為行動端打造生成式 UI

[Flutter GenUI SDK](https://docs.flutter.dev/ai/genui) 將動態、由 AI 生成的 UI 帶進 Flutter 應用，覆蓋行動端、桌面端與 Web。

**GenUI 如何使用 A2UI：**

GenUI SDK 使用 **A2UI 作為底層協議**，負責在伺服端智慧體與 Flutter 應用之間傳遞訊息。換句話說，當你使用 GenUI 時，其實底層就是 A2UI。

- **跨平台支援**：同一個智慧體可運行在 iOS、Android、Web 與桌面端
- **原生效能**：Flutter widgets 在各平台上以原生方式渲染
- **品牌一致性**：UI 可匹配你的應用設計系統
- **伺服端驅動 UI**：智慧體可控制顯示內容，而不必更新應用

> 「開發者選擇 Flutter，是因為它能讓他們快速打造富有表達力、品牌感強烈且高度客製的設計系統，並在每個平台上都保有出色體驗。A2UI 非常適合 Flutter 的 GenUI SDK，因為它能確保每一位使用者、在每一個平台上，都獲得高品質、原生感強烈的體驗。」
>
> **— Vijay Menon**, Engineering Director, Dart & Flutter

**試試看：**

- [GenUI Documentation](https://docs.flutter.dev/ai/genui)
- [Getting Started Video](https://www.youtube.com/watch?v=nWr6eZKM6no)
- [Verdure Example](https://github.com/flutter/genui/tree/main/examples/verdure) (client-server A2UI sample)

---

### Google ADK：Agent Development Kit

[Agent Development Kit](https://google.github.io/adk-docs/)（ADK）是 Google 的開源框架，用來建立與部署 AI 智慧體。它內建的開發者 UI，[ADK Web](https://github.com/google/adk-web)，已包含原生的 A2UI 渲染能力。

**ADK 如何使用 A2UI：**

ADK 整合了 A2UI v0.8 standard catalog，能在聊天介面中直接把符合規範的 agent parts 渲染為原生 UI 元件。ADK 也處理 A2UI ↔ A2A 的訊息轉換，因此以 ADK 建立的智慧體可以把豐富 UI 傳送到任何支援 A2UI 的客戶端。

- **內建渲染**：ADK Web 可在開發 UI 中原生渲染 A2UI 元件
- **A2A 整合**：A2UI 訊息可在 A2A DataPart metadata 與 ADK events 之間轉換
- **Agent SDK**：[A2UI Python agent SDK](../../../agent_sdks/python) 提供 ADK 擴充，讓智慧體能生成 A2UI

**試試看：**
- [ADK 文件](https://google.github.io/adk-docs/)
- [ADK Web](https://github.com/google/adk-web)（支援 A2UI 的開發者 UI）
- [智慧體開發指南](../guides/agent-development.md)（使用 ADK 建立 A2UI 智慧體）

---

## 合作夥伴整合

### AG-UI / CopilotKit：全端智慧體框架

[AG-UI](https://ag-ui.com/) 提供協議，而 [CopilotKit](https://www.copilotkit.ai/) 則提供主要的全端框架，用來建立智慧體應用，並且 **從第一天起就相容 A2UI**。

**它們如何協同運作：**

AG-UI 擅長在自訂前端與專屬智慧體之間建立高頻寬連接。再加上 A2UI 支援之後，開發者就能同時擁有兩者的優勢：

- **狀態同步**：AG-UI 處理應用狀態與聊天歷史
- **A2UI 渲染**：渲染來自第三方智慧體的動態 UI
- **多智慧體支援**：協調來自多個智慧體的 UI
- **框架整合**：透過 CopilotKit 支援 React、Vue、Angular 等其他應用介面

> 「AG-UI 擅長在自訂前端與其專屬智慧體之間建立高頻寬連接。加入 A2UI 支援之後，我們讓開發者可以同時享有兩者的優勢。他們現在可以建立既豐富、又能同步狀態的應用，同時也能透過 A2UI 渲染來自第三方智慧體的動態 UI。這與多智慧體世界非常契合。」
>
> **— Atai Barkai**, Founder of CopilotKit and AG-UI

**進一步了解：**

- [AG-UI](https://ag-ui.com/)
- [CopilotKit](https://www.copilotkit.ai/)

---

### Google 的 AI 驅動產品

隨著 Google 在整個公司範圍內導入 AI，A2UI 提供了一種 **讓 AI 智慧體交換使用者介面的標準化方式**，而不只是交換文字。

**內部智慧體採用情況：**

- **多智慧體工作流程**：多個智慧體可共同為同一個 surface 貢獻內容
- **遠端智慧體支援**：運行於不同服務上的智慧體也能提供 UI
- **標準化**：跨團隊使用共同協議，可降低整合成本
- **對外暴露**：內部智慧體可以輕鬆對外提供（例如 Gemini Enterprise）

> 「就像 A2A 讓任何智慧體不受平台限制地彼此溝通一樣，A2UI 將使用者介面層標準化，並透過 orchestrator 支援遠端智慧體場景。這對內部團隊非常有力，使他們能快速開發出以豐富使用者介面為常態而非例外的智慧體。隨著 Google 持續投入生成式 UI，A2UI 也成為一個理想的平台，讓伺服端驅動的 UI 能在任何客戶端上渲染。」
>
> **— James Wren**, Senior Staff Engineer, AI Powered Google

---

## 社群專案

A2UI 社群正在建立許多令人期待的專案：

### 開源示例

- **Restaurant Finder** ([samples/agent/adk/restaurant_finder](../../../samples/agent/adk/restaurant_finder))
    - 具備動態表單的餐桌預約流程
    - 由 Gemini 驅動的智慧體
    - 提供完整原始碼

- **Component Gallery** ([samples/client/angular - gallery mode](../../../samples/client/angular))
    - 所有元件的互動式展示
    - 附帶程式碼的即時示例
    - 很適合作為學習材料

### 第三方整合

- **[json-render](https://json-render.dev/docs/a2ui)** — Vercel 的 React 函式庫，可透過 Zod schema 渲染 A2UI 元件目錄。另可參考 [json-render vs. A2UI 比較](https://dipjyotimetia.medium.com/vercels-json-render-vs-google-s-a2ui-the-head-to-head-6f213cf1a23b)。
- **[OpenClaw Canvas](https://docs.openclaw.ai/platforms/mac/canvas)** — OpenClaw 透過其 canvas 系統使用 A2UI，在已連接裝置上渲染由智慧體生成的 UI。另可參考 [架構概覽](https://ppaolo.substack.com/p/openclaw-system-architecture-overview)。

### 社群貢獻

你用 A2UI 做了什麼嗎？[歡迎與社群分享！](community.md)

---

## 下一步

- [快速開始指南](../quickstart.md) - 試試示例
- [智慧體開發](../guides/agent-development.md) - 建立智慧體
- [客戶端設定](../guides/client-setup.md) - 整合渲染器
- [社群](community.md) - 加入社群

---

**你正在生產環境中使用 A2UI 嗎？** 歡迎到 [GitHub Discussions](https://github.com/a2ui-project/a2ui/discussions) 分享你的故事。
