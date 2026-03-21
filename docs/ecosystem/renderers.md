# 生態系渲染器

由社群與第三方提供的 A2UI 渲染器實作。

!!! note
    這些渲染器由各自作者維護，而不是 A2UI 團隊。
    請自行確認各專案的相容性、版本支援情況與維護狀態。

## 社群渲染器

| 渲染器 | 平台 | v0.8 | v0.9 | 活躍度 | 連結 |
|----------|----------|------|------|----------|-------|
| **@a2ui-sdk/react** | React（Web） | ✅ | ❌ | ![Stars](https://img.shields.io/github/stars/easyops-cn/a2ui-sdk?style=flat-square&label=⭐) ![Last commit](https://img.shields.io/github/last-commit/easyops-cn/a2ui-sdk?style=flat-square&label=updated) | [GitHub](https://github.com/easyops-cn/a2ui-sdk) · [npm](https://www.npmjs.com/package/@a2ui-sdk/react) · [文件](https://a2ui-sdk.js.org/) |
| **A2UI-Android** | Android（Compose） | ✅ | ❌ | ![Stars](https://img.shields.io/github/stars/lmee/A2UI-Android?style=flat-square&label=⭐) ![Last commit](https://img.shields.io/github/last-commit/lmee/A2UI-Android?style=flat-square&label=updated) | [GitHub](https://github.com/lmee/A2UI-Android) |
| **a2ui-react-native** | React Native | ✅ | ❌ | ![Stars](https://img.shields.io/github/stars/sivamrudram-eng/a2ui-react-native?style=flat-square&label=⭐) ![Last commit](https://img.shields.io/github/last-commit/sivamrudram-eng/a2ui-react-native?style=flat-square&label=updated) | [GitHub](https://github.com/sivamrudram-eng/a2ui-react-native) |
| **@zhama/a2ui** | React（Web） | ✅ | ❌ | — | [npm](https://www.npmjs.com/package/@zhama/a2ui) |
| **A2UI-react** | React（Web） | ✅ | ❌ | ![Stars](https://img.shields.io/github/stars/jem-computer/A2UI-react?style=flat-square&label=⭐) ![Last commit](https://img.shields.io/github/last-commit/jem-computer/A2UI-react?style=flat-square&label=updated) | [GitHub](https://github.com/jem-computer/A2UI-react) |

### 值得一提

這些專案仍處於早期或實驗階段：

- **[@xpert-ai/a2ui-react](https://www.npmjs.com/package/@xpert-ai/a2ui-react)** — 基於 ShadCN UI 元件的 React 渲染器（v0.0.1，發布於 2026 年 1 月）
- **[a2ui-3d-renderer](https://github.com/josh-english-2k18/a2ui-3d-renderer)** — 面向 A2UI 的實驗性 Three.js/WebGL 3D 渲染器（約 2 星）
- **[ai-kit-a2ui](https://github.com/AINative-Studio/ai-kit-a2ui)** — 面向 AIKit 框架的 React + ShadCN 渲染器（約 2 星）

### 重點項目

**@a2ui-sdk/react** 目前是最成熟的社群 React 渲染器，已發布 11 個版本，提供 Radix UI primitives、Tailwind CSS 樣式，以及獨立的文件站點。它曾在 [A2UI 討論區](https://github.com/google/A2UI/discussions/489) 中發布。

**A2UI-Android** 補上了一個重要缺口：它是目前唯一的社群 Jetpack Compose 渲染器，支援 Android 5.0+，具備 20+ 個元件、資料繫結與無障礙支援。

**a2ui-react-native** 是目前唯一的 React Native 渲染器，讓 A2UI 可以透過單一程式碼庫同時支援 iOS 與 Android。

### Python / PyPI

截至 2026 年 3 月，在 PyPI 上尚未找到可信的 A2UI 渲染器套件。A2UI 渲染器本質上是客戶端（UI）函式庫，因此生態系自然更集中在 JavaScript / TypeScript 與原生行動框架上。

## 提交你的渲染器

你做了一個 A2UI 渲染器嗎？我們很樂意把它列在這裡。

### 如何提交

1. **Fork** [google/A2UI](https://github.com/google/A2UI) 倉庫
2. **編輯** 這個檔案（`docs/ecosystem/renderers.md`）—— 在社群渲染器表格中加入一列，包含你的渲染器名稱、平台、npm 套件（若有）、支援的版本，以及原始碼連結
3. **提交 PR** 到 `google/A2UI`，並附上簡短說明
4. **在 [GitHub Discussions](https://github.com/google/A2UI/discussions) 發文** —— 讓社群知道你做了什麼！一段簡短示範影片通常很有幫助。

需要靈感嗎？不妨瀏覽倉庫中的 **[community samples](https://github.com/google/A2UI/tree/main/samples)** —— 其中涵蓋 Angular、Lit 與基於 ADK 的智慧體，是很好的起點。

### 什麼樣的社群渲染器會更容易被採納？

如果你的渲染器具備以下特徵，就更有可能被接受並實際使用：

- 有 **公開可取得的原始碼**（最好是開源，MIT 或 Apache 2.0）
- 清楚說明 **支援哪個 A2UI 規範版本**（v0.8、v0.9，或兩者）
- 覆蓋 **核心元件**：文字、按鈕、輸入欄位與基本版面配置
- 附有 **README**，包含安裝說明與最小使用示例
- **持續維護中** —— 若不再維護，請明確標示為 archived

社群渲染器不需要達到可直接投入生產環境的成熟度才可被列出，實驗性或早期專案也很歡迎放在「值得一提」區塊中。
