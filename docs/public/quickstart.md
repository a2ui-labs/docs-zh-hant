# 快速開始：5 分鐘跑通 A2UI

透過執行餐廳查找示例，親手體驗 A2UI。本指南會讓你在 5 分鐘內看到由智慧體生成的 UI。

## 你將建構什麼

完成本快速開始後，你將獲得：

- ✅ 一個正在執行、使用 A2UI Lit renderer 的 Web 應用
- ✅ 一個由 Gemini 驅動、可以生成動態 UI 的智慧體
- ✅ 一個支援表單生成、時間選擇與確認流程的互動式餐廳查找示例
- ✅ 對 A2UI 訊息如何從智慧體流向 UI 的基本理解

## 前置條件

開始之前，請確認你已具備：

- **Node.js**（v18 或更新版本，並已啟用 [Corepack](https://nodejs.org/api/corepack.html)）— [下載連結](https://nodejs.org/)
- **uv**（Python 套件管理器）— [安裝指南](https://docs.astral.sh/uv/getting-started/installation/)（用於執行 Python 智慧體後端）
- **Gemini API Key** — [可從 Google AI Studio 免費取得](https://aistudio.google.com/apikey)

> ⚠️ **安全提醒**
>
>
> 這個示例會執行一個使用 Gemini 生成 A2UI 回應的 A2A 智慧體。該智慧體可以存取你的 API Key，並向 Google 的 Gemini API 發出請求。在生產環境執行之前，請務必先審閱智慧體程式碼。

## 第 1 步：複製倉庫

```bash
git clone https://github.com/a2ui-project/a2ui.git
cd a2ui
```

## 第 2 步：設定 API Key

將你的 Gemini API Key 匯出為環境變數：

```bash
export GEMINI_API_KEY="your_gemini_api_key_here"
```

## 第 3 步：切換到 Lit 客戶端目錄

```bash
cd samples/client/lit
```

## 第 4 步：安裝並執行

執行示例啟動程式（請確保已啟用 Corepack，讓 Node 能自動抓取正確版本的 Yarn）：

```bash
# 啟用 Corepack（macOS Homebrew 使用者請參閱下方提示）
corepack enable

yarn install
yarn demo:restaurant
```

> [!TIP]
> **macOS Homebrew 使用者：** 如果你已安裝獨立的套件管理工具，請先解除連結衝突的工具，再安裝 Corepack，讓 Corepack 能依專案管理版本：
>
> ```bash
> brew unlink yarn pnpm
> brew install corepack
> corepack enable
> ```

這個命令會：

1. 安裝所有相依套件
2. 建置 A2UI renderer
3. 啟動 A2A 餐廳查找智慧體（Python 後端）
4. 啟動開發伺服器
5. 在瀏覽器中開啟 `http://localhost:5173`

Restaurant Finder 智慧體的原始碼位於 [`samples/agent/adk/restaurant_finder`](../../samples/agent/adk/restaurant_finder)。

> [!NOTE]
> **套件管理工具的使用：** 在 A2UI 倉庫內執行快速開始示例應用程式時，需要使用 Yarn，因為這是透過 Corepack workspaces 設定的。若是在此倉庫之外的一般日常使用與獨立專案中，可以自由選擇你偏好的套件管理工具（例如 npm、pnpm）。

### 手動執行（替代方案）

如果你想在不同的終端機視窗中分別執行智慧體與客戶端，或需要進行疑難排解：

**1. 執行智慧體：**

```bash
cd samples/agent/adk/restaurant_finder
uv run .
```

**2. 執行客戶端：**

```bash
cd samples/client/lit/shell
yarn dev
```

> ✅ **示例已啟動**
>
> 如果一切順利，你會在瀏覽器中看到 Web 應用，而智慧體現在已準備好開始生成 UI。

## 第 5 步：開始體驗

在 Web 應用中，你可以嘗試這些提示詞：

1. **"Book a table for 2"** - 觀察智慧體生成預約表單
2. **"Find Italian restaurants near me"** - 查看動態搜尋結果
3. **"What are your hours?"** - 體驗針對不同意圖生成的不同 UI 版型

### 背後發生了什麼

```
┌─────────────┐         ┌──────────────┐         ┌────────────────┐
│   You Type  │────────>│ A2A Agent    │────────>│  Gemini API    │
│  a Message  │         │  (Python)    │         │  (LLM)         │
└─────────────┘         └──────────────┘         └────────────────┘
                               │                         │
                               │ Generates A2UI JSON     │
                               │<────────────────────────┘
                               │
                               │ Streams JSONL messages
                               v
                        ┌──────────────┐
                        │   Web App    │
                        │ (A2UI Lit    │
                        │  Renderer)   │
                        └──────────────┘
                               │
                               │ Renders native components
                               v
                        ┌──────────────┐
                        │   Your UI    │
                        └──────────────┘
```

1. **你透過 Web UI 傳送訊息**
2. **A2A 智慧體** 接收訊息並把對話送給 Gemini
3. **Gemini 生成** 描述 UI 的 A2UI JSON 訊息
4. **A2A 智慧體將這些訊息串流回傳** 給 Web 應用
5. **A2UI renderer** 將它們轉成原生 Web 元件
6. **你在瀏覽器中看到 UI** 被渲染出來

## A2UI 訊息結構

下面來看一下智慧體送出的內容。這裡提供一個簡化版 JSON 訊息示例：

=== "v0.8 (Legacy)"

    **定義 UI：**

    ```json
    {"surfaceUpdate": {"surfaceId": "main", "components": [
      {"id": "header", "component": {"Text": {"text": {"literalString": "Book Your Table"}, "usageHint": "h1"}}},
      {"id": "date-picker", "component": {"DateTimeInput": {"label": {"literalString": "Select Date"}, "value": {"path": "/reservation/date"}, "enableDate": true}}},
      {"id": "submit-text", "component": {"Text": {"text": {"literalString": "Confirm Reservation"}}}},
      {"id": "submit-btn", "component": {"Button": {"child": "submit-text", "action": {"name": "confirm_booking"}}}}
    ]}}
    ```

    **填入資料：**

    ```json
    {"dataModelUpdate": {"surfaceId": "main", "contents": [
      {"key": "reservation", "valueMap": [
        {"key": "date", "valueString": "2025-12-15"},
        {"key": "time", "valueString": "19:00"},
        {"key": "guests", "valueInt": 2}
      ]}
    ]}}
    ```

    **發出渲染訊號：**

    ```json
    {"beginRendering": {"surfaceId": "main", "root": "header"}}
    ```

=== "v0.9 (Stable)"

    **建立 surface：**

    ```json
    {"version": "v0.9.1", "createSurface": {"surfaceId": "main", "catalogId": "https://a2ui.org/specification/v0_9_1/catalogs/basic/catalog.json"}}
    ```

    **定義 UI：**

    ```json
    {"version": "v0.9.1", "updateComponents": {"surfaceId": "main", "components": [
      {"id": "header", "component": "Text", "text": "# Book Your Table", "variant": "h1"},
      {"id": "date-picker", "component": "DateTimeInput", "label": "Select Date", "value": {"path": "/reservation/date"}, "enableDate": true},
      {"id": "submit-text", "component": "Text", "text": "Confirm Reservation"},
      {"id": "submit-btn", "component": "Button", "child": "submit-text", "variant": "primary", "action": {"event": {"name": "confirm_booking"}}}
    ]}}
    ```

    **填入資料：**

    ```json
    {"version": "v0.9.1", "updateDataModel": {"surfaceId": "main", "path": "/reservation", "value": {"date": "2025-12-15", "time": "19:00", "guests": 2}}}
    ```

    注意：在 v0.9 中，`createSurface` 取代了 `beginRendering`；元件格式更扁平；資料模型也改為普通 JSON 值，而不是帶型別的 adjacency list。

> 💡 **它本質上就是 JSON**
>
> 你會發現它清楚且結構化。LLM 很容易生成這種格式，而且在傳輸與渲染時都很安全，不需要執行程式碼。

## 體驗其他示例

倉庫中還包含其他幾個示例：

### 元件畫廊（不需要智慧體）

查看所有可用的 A2UI 元件：

```bash
yarn start gallery
```

這會啟動一個純客戶端示例，展示所有標準元件（Card、Button、TextField、Timeline 等）的即時示例與程式碼片段。

### 其他語言與框架

雖然本指南以 Lit 客戶端作為範例，但 A2UI 也在 `samples/client` 目錄中提供了其他熱門框架的示例：

- **Angular**：`samples/client/angular`
- **React**：`samples/client/react`

探索 [samples/client](../../samples/client) 目錄，查看所有可用的客戶端實作。

## 下一步

現在你已經看過 A2UI 的實際效果，接下來可以繼續：

- **[學習核心概念](concepts/overview.md)**：理解 surface、component 與 data binding
- **[建立自己的客戶端](guides/client-setup.md)**：把 A2UI 整合進你的應用
- **[建構智慧體](guides/agent-development.md)**：建立能生成 A2UI 回應的智慧體
- **[使用既有的智慧體應用](guides/a2ui-with-any-agent-framework.md)**：透過 CopilotKit + AG-UI，為 ADK、LangGraph、CrewAI、Mastra 或自訂服務加入 A2UI
- **[閱讀協議細節](reference/messages.md)**：深入了解技術規範

## 疑難排解

### 連接埠已被佔用

如果 5173 連接埠已被佔用，開發伺服器會自動嘗試下一個可用連接埠。請查看終端機輸出中的實際 URL。

### API Key 問題

如果你看到缺少 API Key 的錯誤：

1. 確認該變數已匯出：`echo $GEMINI_API_KEY`
2. 確認它是從 [Google AI Studio](https://aistudio.google.com/apikey) 取得的有效 Gemini API Key
3. 嘗試重新匯出：`export GEMINI_API_KEY="your_key"`

### 啟動時連線錯誤

如果瀏覽器開啟時看到 `ERR_CONNECTION_REFUSED`，**不用擔心**。這是一個已知的競態問題（[#587](https://github.com/a2ui-project/a2ui/issues/587)）。Web 應用啟動速度比 Python 智慧體後端更快。等待幾秒後重新整理頁面即可。

### Python / uv 問題

這些示例智慧體需要 [uv](https://docs.astral.sh/uv/) 才能執行。如果你看到 `uv: command not found`：

```bash
# 安裝 uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# 驗證
uv --version
```

如果你遇到其他 Python 錯誤：

```bash
# 確認 Python 3.10+ 可用
python3 --version

# 嘗試手動執行智慧體
cd samples/agent/adk/restaurant_finder
uv run .
```

### 還有問題？

- 查看 [GitHub Issues](https://github.com/a2ui-project/a2ui/issues)
- 閱讀 [samples/client/lit/README.md](https://github.com/a2ui-project/a2ui/tree/main/samples/client/lit)
- 參與社群討論

## 理解示例程式碼

如果你想看它是怎麼實作的，可以查看：

- **Agent 程式碼**：`samples/agent/adk/restaurant_finder/` — Python A2A 智慧體
- **客戶端程式碼**：`samples/client/lit/` — 整合了 A2UI renderer 的 Lit Web 客戶端
- **A2UI Renderers**：`renderers/lit/`（Lit）與 `renderers/web_core/`（框架無關核心）

每個目錄下都有自己的 README，提供更詳細的說明。

---

**恭喜！** 你已成功執行第一個 A2UI 應用。你已經看到，AI 智慧體如何透過安全、宣告式的 JSON 訊息，在 Web 應用中生成豐富且可互動的原生 UI。
