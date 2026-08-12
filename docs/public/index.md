---
hide:
  - toc
---

<!-- markdownlint-disable MD041 -->
<!-- markdownlint-disable MD033 -->
<div style="text-align: center; margin: 2rem 0 3rem 0;" markdown>

<!-- Logo for Light Mode (shows dark logo on light background) -->
<img src="assets/A2UI_dark.svg" alt="A2UI Logo" width="120" class="light-mode-only" style="margin-bottom: 1rem;">
<!-- Logo for Dark Mode (shows light logo on dark background) -->
<img src="assets/A2UI_light.svg" alt="A2UI Logo" width="120" class="dark-mode-only" style="margin-bottom: 1rem;">

# 面向智慧體驅動介面的協議

<p style="font-size: 1.2rem; max-width: 800px; margin: 0 auto 1rem auto; opacity: 0.9; line-height: 1.6;">
A2UI 讓 AI 智慧體能夠生成豐富、可互動的使用者介面，並在 Web、行動端與桌面端以原生方式渲染，而不需要執行任意程式碼。
</p>

</div>

## 規範版本

| 版本 | 狀態 | 說明 |
|---------|--------|-------------|
| **[v1.0](specification/v1.0-a2ui.md)** | **候選版** | 發布候選版本。新增客戶端對伺服端的 RPC（`actionResponse`）、action ID，並將 theme 更名為 surfaceProperties。（草案階段時曾稱為 v0.10。）[查看演進指南 →](specification/v1.0-evolution-guide.md) |
| **[v0.9.1](specification/v0.9.1-a2ui.md)** | **目前版本** | 目前可用於生產的版本。針對 v0.9 進行小幅修訂，統一採用 `application/a2ui+json` MIME 類型，並放寬 surfaceId 的限制條件。[查看演進指南 →](specification/v0.9.1-evolution-guide.md) |
| **[v0.9](specification/v0.9-a2ui.md)** | **穩定版** | 前一個穩定版本。核心理念轉向以提示詞優先。引入 `createSurface`、客戶端函式、自訂 catalog、模組化 schema，以及驗證失敗的錯誤格式。[查看演進指南 →](specification/v0.9-evolution-guide.md) |
| **[v0.8](specification/v0.8-a2ui.md)** | **舊版** | 舊版本。以結構化輸出為優先。提供基礎的 surface、component、data binding 與 adjacency list 模型。 |

A2UI 採用 Apache 2.0 授權，
由 Google 發起，[CopilotKit](https://docs.copilotkit.ai/generative-ui/a2ui) 與開源社群共同參與貢獻，
並持續在 [GitHub](https://github.com/a2ui-project/a2ui) 上活躍開發中。

A2UI 要解決的問題是：**AI 智慧體如何在跨越信任邊界時，安全地傳送豐富的 UI？**

不同於只回傳文字，或直接執行高風險程式碼，A2UI 讓智慧體傳送**宣告式元件描述**，再由客戶端使用自己的原生元件進行渲染。這就像讓智慧體學會了一門通用的 UI 語言。

在這個倉庫中，你會找到
[A2UI 規範](specification/v0.9.1-a2ui.md)（v0.9.1 目前版本、v1.0 候選版），
客戶端側的 [renderers](reference/renderers.md) 實作（Angular、Flutter、Lit、Markdown 等），
以及用來在智慧體與客戶端之間傳遞 A2UI 訊息的 [transports](concepts/transports.md)（如 A2A）。

<div class="grid cards" markdown>

- :material-shield-check: **從設計上確保安全**

    ---

    它是宣告式資料格式，而不是可執行程式碼。智慧體只能使用你在 catalog 中預先核準的元件，從根源避免 UI 注入攻擊。

- :material-rocket-launch: **對 LLM 友善**

    ---

    扁平、可串流傳輸的 JSON 結構，專為易生成而設計。LLM 不需要一次就產生完美 JSON，也能逐步建構 UI。

- :material-devices: **與框架無關**

    ---

    一份智慧體回應可在多端重用。你可以在 Angular、Flutter、React 或原生行動端，以自己的樣式元件渲染同一份 UI。

- :material-chart-timeline: **漸進式渲染**

    ---

    UI 更新一生成就能立刻串流傳送。使用者不必等待完整回應，就能即時看到介面逐步建構出來。

</div>

## 5 分鐘快速上手

<div class="grid cards" markdown>

- :material-clock-fast:{ .lg .middle } **[快速開始指南](quickstart.md)**

    ---

    執行餐廳查找示例，親自體驗由 Gemini 驅動的智慧體如何使用 A2UI。

    [:octicons-arrow-right-24: 立即開始](quickstart.md)

- :material-book-open-variant:{ .lg .middle } **[核心概念](concepts/overview.md)**

    ---

    了解 surface、component、data binding 與 adjacency list 模型。

    [:octicons-arrow-right-24: 學習概念](concepts/overview.md)

- :material-code-braces:{ .lg .middle } **[開發指南](guides/client-setup.md)**

    ---

    將 A2UI renderer 整合進你的應用，或建構能夠生成 UI 的智慧體。

    [:octicons-arrow-right-24: 開始建構](guides/client-setup.md)

- :material-file-document:{ .lg .middle } **協議規範**

    ---

    深入閱讀完整技術規範：[v0.8（舊版）](specification/v0.8-a2ui.md) · [v0.9（穩定版）](specification/v0.9-a2ui.md) · [v0.9.1（目前版本）](specification/v0.9.1-a2ui.md) · [v1.0（候選版）](specification/v1.0-a2ui.md)

    [:octicons-arrow-right-24: 閱讀 v0.9.1 規範](specification/v0.9.1-a2ui.md)

</div>

## 工作原理

1. **使用者傳送訊息** 給 AI 智慧體
2. **智慧體生成 A2UI 訊息**，描述 UI（結構 + 資料）
3. **訊息被串流傳送** 到客戶端應用
4. **客戶端使用原生元件渲染**（Angular、Flutter、React 等）
5. **使用者與 UI 互動**，再把操作傳回給智慧體
6. **智慧體回應**，並送出更新後的 A2UI 訊息

![End-to-End Data Flow](assets/end-to-end-data-flow.png)

## A2UI 實際效果

### 景觀設計師示例

<div style="margin: 2rem 0;">
  <div style="border-radius: .8rem; overflow: hidden; box-shadow: var(--md-shadow-z2);">
    <video width="100%" height="auto" controls playsinline style="display: block; aspect-ratio: 16/9; object-fit: cover;">
      <source src="assets/landscape-architect-demo.mp4" type="video/mp4">
      你的瀏覽器不支援 video 標籤。
    </video>
  </div>
  <p style="text-align: center; margin-top: 1rem; opacity: 0.8;">
    觀看智慧體為景觀設計應用動態生成完整介面。使用者上傳照片後，智慧體會使用 Gemini 理解影像內容，並生成符合景觀需求的客製表單。
  </p>
</div>

### 自訂元件：互動式圖表與地圖

<div style="margin: 2rem 0;">
  <div style="border-radius: .8rem; overflow: hidden; box-shadow: var(--md-shadow-z2);">
    <video width="100%" height="auto" controls playsinline style="display: block; aspect-ratio: 16/9; object-fit: cover;">
      <source src="assets/a2ui-custom-component.mp4" type="video/mp4">
      你的瀏覽器不支援 video 標籤。
    </video>
  </div>
  <p style="text-align: center; margin-top: 1rem; opacity: 0.8;">
    觀看智慧體如何在回答數值摘要問題時選擇圖表元件，在回答地理位置問題時選擇 Google Map 元件。這兩個元件都由客戶端以自訂元件形式提供。
  </p>
</div>

### A2UI Composer

我們提供了一個公開可用的 [A2UI Composer](https://a2ui-project.github.io/composer/)。

[A2UI Composer 文件](./composer/index.md)
