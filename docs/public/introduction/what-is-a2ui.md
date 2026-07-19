# 什麼是 A2UI？

**A2UI（Agent to UI）是一個用於智慧體驅動介面的宣告式 UI 協議。** AI 智慧體可以生成豐富、可互動的 UI，並在不同平台（Web、行動端、桌面端）以原生方式渲染，而不需要執行任意程式碼。

## 問題是什麼

**只有文字的智慧體互動效率很低：**

```
User: "Book a table for 2 tomorrow at 7pm"
Agent: "Okay, for what day?"
User: "Tomorrow"
Agent: "What time?"
...
```

**更好的做法：** 讓智慧體生成一個帶有日期選擇器、時間選擇器與送出按鈕的表單。使用者與 UI 互動，而不是一直靠文字往返。

## 挑戰

在多智慧體系統中，智慧體通常是遠端執行的（位於不同伺服器、不同組織）。它們無法直接操作你的 UI，只能透過訊息來表達。

**傳統做法：** 在 iframe 中傳送 HTML / JavaScript

- 體積重、視覺體驗割裂
- 安全性複雜
- 難以融入應用本身的樣式系統

**真正需要的是：** 能像資料一樣安全、又像程式碼一樣具備表達力的 UI 傳輸方式。

## 解法

A2UI 透過 JSON 訊息描述 UI，並具備以下特性：

- 可由 LLM 作為結構化輸出生成
- 可透過任何 transport 傳遞（A2A、AG-UI、SSE、WebSockets）
- 由客戶端使用自己的原生元件進行渲染

**結果：** 安全與樣式由客戶端掌控，而智慧體生成的 UI 仍能保有原生體驗。

### 範例

=== "v0.8 (Legacy)"

    ```jsonl
    {
      "surfaceUpdate": {
        "surfaceId": "booking",
        "components": [
          {
            "id": "title",
            "component": {
              "Text": {
                "text": { "literalString": "Book Your Table" },
                "usageHint": "h1"
              }
            }
          },
          {
            "id": "datetime",
            "component": {
              "DateTimeInput": {
                "value": { "path": "/booking/date" },
                "enableDate": true
              }
            }
          },
          {
            "id": "submit-text",
            "component": {
              "Text": {
                "text": { "literalString": "Confirm" }
              }
            }
          },
          {
            "id": "submit-btn",
            "component": {
              "Button": {
                "child": "submit-text",
                "action": { "name": "confirm_booking" }
              }
            }
          }
        ]
      }
    }
    {
      "dataModelUpdate": {
        "surfaceId": "booking",
        "contents": [
          {
            "key": "booking",
            "valueMap": [
              { "key": "date", "valueString": "2025-12-16T19:00:00Z" }
            ]
          }
        ]
      }
    }
    {
      "beginRendering": {
        "surfaceId": "booking",
        "root": "title"
      }
    }
    ```

=== "v0.9 (Stable)"

    ```jsonl
    {
      "version": "v0.9.1",
      "createSurface": {
        "surfaceId": "booking",
        "catalogId": "https://a2ui.org/specification/v0_9_1/catalogs/basic/catalog.json"
      }
    }
    {
      "version": "v0.9.1",
      "updateComponents": {
        "surfaceId": "booking",
        "components": [
          {
            "id": "title",
            "component": "Text",
            "text": "Book Your Table",
            "variant": "h1"
          },
          {
            "id": "datetime",
            "component": "DateTimeInput",
            "value": { "path": "/booking/date" },
            "enableDate": true
          },
          {
            "id": "submit-text",
            "component": "Text",
            "text": "Confirm"
          },
          {
            "id": "submit-btn",
            "component": "Button",
            "child": "submit-text",
            "variant": "primary",
            "action": {
              "event": { "name": "confirm_booking" }
            }
          }
        ]
      }
    }
    {
      "version": "v0.9.1",
      "updateDataModel": {
        "surfaceId": "booking",
        "path": "/booking",
        "value": {
          "date": "2025-12-16T19:00:00Z"
        }
      }
    }
    ```

    v0.9 的主要差異在於：`createSurface` 取代了 `beginRendering`；元件採用更扁平的結構，用 `"component": "Text"` 取代巢狀物件；所有訊息都包含 `version` 欄位。

客戶端會將這些訊息渲染為原生元件（Angular、Flutter、React 等）。

## 核心價值

**1. 安全性：** 它是宣告式資料，不是程式碼。智慧體只能從客戶端受信任的 catalog 中請求元件，因此沒有執行程式碼的風險。

**2. 原生體驗：** 不需要 iframe。客戶端使用自己的 UI 框架進行渲染，因此能自然沿用應用的樣式、無障礙能力與效能特性。

**3. 可攜性：** 一份智慧體回應可在各處使用。同一份 JSON 可以在 Web（Lit / Angular / React）、行動端（Flutter / SwiftUI / Jetpack Compose）與桌面端渲染。

## 設計原則

**1. 對 LLM 友善：** 使用帶有 ID 參照的扁平元件列表，易於逐步生成、修正錯誤與進行串流。

**2. 與框架無關：** 智慧體送出抽象元件樹，由客戶端對映到原生 widgets（web / mobile / desktop）。

**3. 關注點分離：** 拆成三層——UI 結構、應用狀態、客戶端渲染。這讓資料繫結、反應式更新與清晰架構都更容易實現。

## A2UI 不是什麼

- 它不是一個框架（而是一個協議）
- 它不是 HTML 的替代品（它服務的是智慧體生成 UI，而不是靜態網站）
- 它不是一套完整的樣式系統（樣式主要由客戶端掌控，伺服端樣式能力有限）
- 它不只限於 Web（也支援行動端與桌面端）

## 關鍵概念

- **Surface**：元件承載區域（dialog、sidebar、主視圖等）
- **Component**：UI 元件（Button、TextField、Card 等）
- **Data Model**：應用狀態，元件會與它繫結
- **Catalog**：可用的元件類型集合
- **Message**：JSON 物件（`surfaceUpdate`、`dataModelUpdate`、`beginRendering` 等）

若想比較相關專案，請參閱 [Agent UI Ecosystem](agent-ui-ecosystem.md)。
