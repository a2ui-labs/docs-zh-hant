# 自訂元件 Catalogs

透過定義 **custom catalogs**，你可以把自己的元件與標準 A2UI 元件一起納入，進一步擴充 A2UI。

## 為什麼需要 Custom Catalogs？

A2UI Standard Catalog 提供了常見的 UI 元件（buttons、text fields 等），但你的應用可能需要更專門的元件：

- **特定領域 widgets**：股票行情、醫療圖表、CAD 檢視器
- **第三方整合**：Google Maps、付款表單、聊天 widgets
- **品牌專屬元件**：自訂日期選擇器、商品卡片、儀表板

**Custom catalogs** 是一組元件集合，可以包含：
- 標準 A2UI 元件（Text、Button、TextField 等）
- 你的自訂元件（GoogleMap、StockTicker 等）
- 第三方元件

你註冊到客戶端應用的是整個 catalog，而不是單一元件。這樣能讓 agents 與 clients 針對一套共享、可擴充的元件集達成共識，同時保有安全性與型別安全。

## Custom Catalogs 如何運作

1. **Client 定義 Catalog**：你建立一份 catalog 定義，列出標準元件與自訂元件。
2. **Client 註冊 Catalog**：你把 catalog（以及其元件實作）註冊到客戶端應用中。
3. **Client 宣告支援能力**：客戶端告訴 agent 它支援哪些 catalogs。
4. **Agent 選擇 Catalog**：agent 為特定 UI surface 選擇要使用的 catalog。
5. **Agent 生成 UI**：agent 使用該 catalog 中的元件名稱來生成元件訊息（v0.8 的 `surfaceUpdate`、v0.9 的 `updateComponents`）。

## 定義 Custom Catalogs

TODO：補充各平台如何定義 custom catalogs 的詳細指南。

**Web（Lit / Angular）：**

- 如何定義同時包含標準元件與自訂元件的 catalog
- 如何在 A2UI client 中註冊 catalog
- 如何實作自訂元件 class

**Flutter：**

- 如何透過 GenUI 定義 custom catalogs
- 如何註冊 custom component renderers

**參考可運作的示例：**

- [Lit samples](https://github.com/google/a2ui/tree/main/samples/client/lit)
- [Angular samples](https://github.com/google/a2ui/tree/main/samples/client/angular)
- [Flutter GenUI docs](https://docs.flutter.dev/ai/genui)

## Agent 端：使用來自 Custom Catalog 的元件

當 catalog 已在 client 上註冊後，agents 就可以在 `surfaceUpdate` 訊息中使用其中的元件。

agent 會透過 `beginRendering` 訊息中的 `catalogId` 指定要使用哪個 catalog。

TODO：補充以下範例：

- agents 如何選擇 catalogs
- agents 如何從 catalogs 參照自訂元件
- catalog 版本管理如何運作

## 資料繫結與 Actions

Custom components 與標準元件一樣，支援相同的資料繫結與 action 機制：

- **資料繫結**：自訂元件可以透過 JSON Pointer 語法，把屬性繫結到 data model 路徑上
- **Actions**：自訂元件可以發出 actions，讓 agent 接收並處理

## 安全考量

在建立 custom catalogs 與 components 時：

1. **Allowlist 元件**：只把你信任的元件註冊進 catalogs
2. **驗證屬性**：永遠要驗證來自 agent 訊息的元件屬性
3. **清理使用者輸入**：若元件接受使用者輸入，請先清理再處理
4. **限制 API 存取**：不要把敏感 API 或憑證暴露給 custom components

TODO：補充詳細的安全最佳實務與程式碼範例。

## 下一步

- **[主題與樣式](theming.md)**：自訂元件的外觀與風格
- **[Component Reference](../reference/components.md)**：查看所有標準元件
- **[Agent Development](agent-development.md)**：建立能使用自訂元件的 agents
