# 智慧體（伺服端）

智慧體是部署在伺服端的程式，會根據使用者請求生成 A2UI 訊息。

實際的元件渲染由 [渲染器](renderers.md) 完成，
訊息會先被 [傳輸層](../concepts/transports.md) 傳到客戶端。
智慧體的責任只在於生成 A2UI 訊息。

## 智慧體如何運作

```
使用者輸入 → 智慧體邏輯 → LLM → A2UI JSON → 傳送到客戶端
```

1. **接收** 使用者訊息
2. **用 LLM 處理**（Gemini、GPT、Claude 等）
3. **生成** 結構化輸出的 A2UI JSON 訊息
4. **透過傳輸層傳送** 到客戶端

來自客戶端的使用者互動，也可以視為新的使用者輸入。

## 示例智慧體

A2UI 倉庫中提供了幾個可以參考的示例智慧體：

- [Restaurant Finder](https://github.com/a2ui-project/a2ui/tree/main/samples/agent/adk/restaurant_finder)
    - 使用表單完成訂位
    - 以 ADK 撰寫
- [Rizzcharts](https://github.com/a2ui-project/a2ui/tree/main/samples/community/agent/adk/rizzcharts/python)
    - A2UI 自訂元件示例
    - 以 ADK 撰寫
- [Orchestrator](https://github.com/a2ui-project/a2ui/tree/main/samples/community/agent/adk/orchestrator)
    - 轉送來自遠端子智慧體的 A2UI 訊息
    - 以 ADK 撰寫

## 你會在 A2A 中搭配使用的智慧體類型

### 1. 使用者面向的智慧體（獨立）

這類智慧體會直接與使用者互動。

### 2. 作為遠端智慧體宿主的使用者面向智慧體

這是一種常見模式：由使用者面向的智慧體擔任一個或多個遠端智慧體的宿主。使用者面向智慧體會呼叫遠端智慧體，而遠端智慧體會生成 A2UI 訊息。在 [A2A](https://a2a-protocol.org) 中，這通常表現為客戶端智慧體呼叫伺服端智慧體。

- 使用者面向智慧體可以直接「透傳」A2UI 訊息而不做修改
- 使用者面向智慧體也可以在送到客戶端之前改寫 A2UI 訊息

### 3. 遠端智慧體

遠端智慧體並不是使用者介面本身的直接一部分。它會被註冊為遠端智慧體，並由使用者面向智慧體視需要呼叫。這也是 [A2A](https://a2a-protocol.org) 中客戶端智慧體呼叫伺服端智慧體的常見模式。
