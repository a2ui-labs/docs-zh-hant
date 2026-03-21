# 傳輸層（訊息傳遞）

傳輸層負責把 A2UI 訊息從智慧體送到客戶端。A2UI 不綁定特定傳輸方式，只要能夠傳送 JSON，就可以使用。

實際的元件渲染由 [renderer](../reference/renderers.md) 完成，
[agents](../reference/agents.md) 負責生成 A2UI 訊息，
而把訊息從智慧體送到客戶端，就是 transport 的工作。

## 運作方式

```
Agent → Transport → Client Renderer
```

A2UI 定義了一串 JSON 訊息序列。傳輸層負責把這串訊息從智慧體送到客戶端。常見做法是使用 JSON Lines（JSONL）這類串流格式，其中每一行都是一則獨立的 A2UI 訊息。

## 可用的傳輸方式

| 傳輸方式 | 狀態 | 使用場景 |
|-----------|--------|----------|
| **A2A Protocol** | ✅ 穩定 | 多智慧體系統、企業級網格 |
| **AG UI** | ✅ 穩定 | 全端 React 應用 |
| **REST API** | 📋 規劃中 | 簡單 HTTP 端點 |
| **WebSockets** | 💡 提議中 | 即時雙向通訊 |
| **SSE (Server-Sent Events)** | 💡 提議中 | Web 串流 |

## A2A Protocol

[Agent2Agent (A2A) protocol](https://a2a-protocol.org) 提供安全且標準化的智慧體通訊能力。A2UI 透過 A2A 擴充，可以相對容易地整合進這個協議。

**優點：**

- 內建安全與驗證機制
- 支援多種訊息格式、驗證方式與傳輸協議
- 關注點分離清楚

如果你已經在使用 A2A，這部分整合通常幾乎是自動完成的。

TODO：補充詳細指南。

**另見：** [A2A 擴充規範](../specification/v0.8-a2a-extension.md)

## AG UI

[AG UI](https://ag-ui.com/) 會把 A2UI 訊息轉成 AG UI 訊息，並自動處理傳輸與狀態同步。

如果你正在使用 AG UI，這部分通常也會自動完成。

TODO：補充詳細指南。

## 自訂傳輸方式

任何能夠傳送 JSON 的傳輸方式都可以使用：

**HTTP/REST：**

```javascript
// TODO: Add an example
```

**WebSockets：**

```javascript
// TODO: Add an example
```

**Server-Sent Events：**

```javascript
// TODO: Add an example
```

## 下一步

- **[A2A Protocol 文件](https://a2a-protocol.org)**：了解 A2A
- **[A2A 擴充規範](../specification/v0.8-a2a-extension.md)**：查看 A2UI 與 A2A 的結合細節
