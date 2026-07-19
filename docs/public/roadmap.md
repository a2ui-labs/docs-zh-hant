# 路線圖

這份路線圖概述了 A2UI 專案目前的狀態與未來規劃。專案仍在積極開發中，優先順序可能會根據社群回饋與新興使用場景而調整。

## 目前狀態

### 協議

| 版本 | 狀態 | 說明 |
|---------|--------|-------|
| **v0.8** | 🆗 先前版本 | 初次公開發布，僅提供最基本的支援 |
| **v0.9** | 🆗 先前版本 | 功能完整，屬於舊版支援 |
| **v0.9.1** | 🆗 目前版本 | 穩定發布版本 |
| **v1.0** | 📋 候選版 | 發布候選版本 |

主要特性：

- ✅ 串流式 JSONL 訊息格式
- ✅ 四種核心訊息類型（`surfaceUpdate`、`dataModelUpdate`、`beginRendering`、`deleteSurface`）
- ✅ Adjacency list 元件模型
- ✅ 以 JSON Pointer 為基礎的資料繫結
- ✅ 結構與狀態分離

### 渲染器

| 客戶端函式庫 | 狀態 | 平台 | 說明 |
|-----------------|--------|----------|-------|
| **Web Components (Lit)** | ✅ 穩定版 | Web | 與框架無關，幾乎可在任何地方使用 |
| **Angular** | ✅ 穩定版 | Web | 完整 Angular 整合 |
| **Flutter (GenUI SDK)** | ✅ 穩定版 | 多平台 | 支援行動端、Web 與桌面端 |
| **React** | 🚧 開發中 | Web | 預計 2026 Q1 |
| [**Lynx**](https://lynxjs.org/next/react/genui/a2ui.html) | ✅ 穩定版 | 行動端、Web、桌面端 | 適用於 A2UI v0.9 的 ReactLynx 渲染器 |
| **SwiftUI** | 📋 規劃中 | iOS / macOS | 預計 2026 Q2 |
| **Jetpack Compose** | 📋 規劃中 | Android | 預計 2026 Q2 |
| **Vue** | 💡 提議中 | Web | 社群有興趣 |
| [**Svelte/Kit**](https://svelte.dev/docs/kit/introduction) | 💡 提議中 | Web | [社群有興趣](https://news.ycombinator.com/item?id=46287728) |
| **ShadCN (React)** | 💡 提議中 | Web | 社群有興趣 |

### 傳輸層

| 傳輸方式 | 狀態 | 說明 |
|-------------|--------|-------|
| **A2A Protocol** | ✅ 完成 | 原生 A2A 傳輸 |
| **AG-UI** | ✅ 完成 | 從第一天起即相容 |
| **REST API** | 📋 規劃中 | 雙向通訊 |
| **WebSockets** | 💡 提議中 | 雙向通訊 |
| **SSE (Server-Sent Events)** | 💡 提議中 | Web 串流 |
| **MCP (Model Context Protocol)** | 💡 提議中 | 社群有興趣 |

### 智慧體 UI 工具包

| Agent UI 工具包 | 狀態 | 說明 |
|-------------|--------|-------|
| **CopilotKit** | ✅ 完成 | 透過 AG-UI 從第一天起即相容 |
| **Open AI ChatKit** | 💡 提議中 | 社群有興趣 |
| **Vecel AI SDK UI** | 💡 提議中 | 社群有興趣 |

### 智慧體框架

| 整合項目 | 狀態 | 說明 |
|-------------|--------|-------|
| **任何支援 A2A 的智慧體** | ✅ 完成 | 因 A2A 協議而具備從第一天起的相容性 |
| **ADK** | 📋 規劃中 | 仍在設計開發者體驗，可參考 [samples](https://github.com/a2ui-project/a2ui/tree/main/samples/agent/adk) |
| **Genkit** | 💡 提議中 | 社群有興趣 |
| **LangGraph** | 💡 提議中 | 社群有興趣 |
| **CrewAI** | 💡 提議中 | 社群有興趣 |
| **AG2** | 💡 提議中 | 社群有興趣 |
| **Claude Agent SDK** | 💡 提議中 | 社群有興趣 |
| **OpenAI Agent SDK** | 💡 提議中 | 社群有興趣 |
| **Microsoft Agent Framework** | 💡 提議中 | 社群有興趣 |
| **AWS Strands Agent SDK** | 💡 提議中 | 社群有興趣 |

## 近期里程碑

### 2025 Q2

Google 多個團隊展開了許多研究專案，包括整合至內部產品與智慧體中。

### 2025 Q4

- 發布 v0.8.0 規範
- 推出 A2A 擴充（感謝 Google A2A 團隊！曾於 [a2asummit.ai](https://a2asummit.ai) 預告）
- 推出 Flutter 渲染器（感謝 Flutter 團隊！）
- 推出 Angular 渲染器（感謝 Angular 團隊！）
- 推出 Web Components（Lit）渲染器（感謝 Opal 團隊與朋友們！）
- AG-UI / CopilotKit 整合（感謝 CopilotKit 團隊！）
- GitHub 公開發布（Apache 2.0）

## 即將到來的里程碑

### 2026 Q1

#### A2UI v0.9

- 發布 0.9 規範候選版本
- 改善渲染器的主題支援（完整）
- 改善智慧體的伺服端主題支援（最小可用）
- 改善開發者體驗

#### React 渲染器

提供一個原生 React 渲染器，具備基於 hooks 的 API 與完整 TypeScript 支援。

- React 對常用元件的支援
- React 對自訂元件的支援
- 用於訊息處理的 `useA2UI` hook
- React 的主題支援

### 2026 Q2

#### 原生行動端渲染器

為 iOS 與 Android 平台打造原生渲染器。

**SwiftUI 渲染器（iOS / macOS）：**

- 原生 SwiftUI 元件
- 支援 iOS 設計語言
- 支援 macOS

**Jetpack Compose 渲染器（Android）：**

- 原生 Compose UI 元件
- 支援 Material Design 3
- 與 Android 平台整合

#### 效能優化

- 渲染器效能基準測試
- 大型元件樹的延遲載入
- 列表的虛擬捲動
- 元件記憶化策略

### 2026 Q4

#### 協議 v1.0

完成 v1.0 協議，包含：

- 穩定性保證
- 從 v0.9 升級的遷移路徑
- 完整測試套件
- 渲染器認證計畫

## 長期願景

### 多智慧體協作

更完善地支援多個智慧體共同貢獻同一個 UI：

- 建議的智慧體組合模式
- 衝突解決策略
- 共用 surface 管理

### 無障礙能力

提供一流的無障礙支援：

- 產生 ARIA 屬性
- 螢幕閱讀器最佳化
- 鍵盤操作標準
- 對比與色彩指引

### 進階 UI 模式

支援更複雜的 UI 互動：

- 拖放
- 手勢與動畫
- 3D 渲染
- AR / VR 介面（探索中）

### 生態系成長

- 更多框架整合
- 第三方元件函式庫
- 智慧體市集整合
- 企業級功能與支援

## 社群需求

以下是社群提出的功能需求（順序不分先後）：

- **更多渲染器整合**：把你的客戶端函式庫對映到 A2UI
- **更多智慧體框架**：把你的智慧體框架對映到 A2UI
- **更多傳輸層**：把你的傳輸方式對映到 A2UI
- **社群元件函式庫**：與社群分享自訂元件
- **社群示例**：與社群分享自訂示例
- **社群評估**：生成式 UI 評估場景與標註資料集
- **開發者體驗**：如果你能打造更好的 A2UI 體驗，歡迎與社群分享

## 如何影響路線圖

我們歡迎社群對優先順序提出意見：

1. **為 Issues 投票**：對你在意的 GitHub issues 按 👍
2. **提議功能**：在 GitHub 上開啟討論（先搜尋是否已有相關討論）
3. **提交 PR**：直接實作你需要的功能（先搜尋是否已有相關 PR）
4. **加入討論**：分享你的使用場景與需求（先搜尋是否已有相關討論）

## 發布週期

- **主要版本**（1.0、2.0）：每年一次，或在需要重大破壞性變更時發布
- **次要版本**（1.1、1.2）：按季發布新功能
- **修補版本**（1.1.1、1.1.2）：視 bug 修復需求而定

## 版本策略

A2UI 採用 [Semantic Versioning](https://semver.org/)：

- **MAJOR**：不相容的協議變更
- **MINOR**：向後相容的新功能
- **PATCH**：向後相容的 bug 修復

## 參與方式

想對路線圖做出貢獻嗎？

- 在 [GitHub Discussions](https://github.com/a2ui-project/a2ui/discussions) 中 **提出功能建議**
- **建立原型** 並與社群分享
- 在 GitHub Issues 中 **加入討論**

## 持續關注

- Watch [GitHub 倉庫](https://github.com/a2ui-project/a2ui) 以接收更新
- Star 倉庫以表達支持
- 追蹤 releases 以在新版本發布時收到通知

---

**最後更新：** 2026 年 6 月

對路線圖有問題嗎？歡迎在 [GitHub](https://github.com/a2ui-project/a2ui/discussions) 發起討論。
