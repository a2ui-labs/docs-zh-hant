# 處理使用者 Actions

本指南說明 A2UI 如何處理使用者互動。元件使用 `action` 屬性來觸發本機 **Functions**（在 renderer 上執行）或 **Events**（派發給 agent）。此外，**Data Model Synchronization** 可確保 agent 始終能夠存取完整 UI 狀態，從而支援語音命令等順暢的多模態互動。這個設計在維持安全、受限環境的同時，也能實作高回應性的介面。

## Action 架構

Action 允許 UI 元件觸發 [`common_types.json`](../../specification/v0_9/json/common_types.json#L271-L313) 中 [`Action`](../../specification/v0_9/json/common_types.json#L271-L313) schema 定義的行為。Action 可以觸發：

1.  **Events**：派發給 Agent 處理（在 Agent 上執行，例如點擊“Submit”）。
2.  **Functions**：使用 [`FunctionCall`](../../specification/v0_9/json/common_types.json#L200-L242) 完全在 renderer 上執行（在 Renderer 上執行，例如開啟 URL）。

### 1. Functions（本機）

Function 會在 renderer 上立即執行行為，不需要網路往返。Agent 不會收到本機 function call 的通知。它們使用 `functionCall` 關鍵字。

```json
{
  "id": "help-btn",
  "component": "Button",
  "child": "help-text",
  "action": {
    "functionCall": {
      "call": "openUrl",
      "args": {"url": "https://a2ui.org/help"}
    }
  }
}
```

Function 的常見用途包括：

- **導航**：開啟 URL 或切換標籤頁。
- **驗證**：提交前檢查輸入（見下方 Checks）。

### 2. Events（Agent）

Event 會把資料送出給 agent 處理。它們使用 `event` 關鍵字。

`Button` 這樣的元件會暴露 `action` 屬性。下面是一個 Event 的連接方式：

```json
{
  "id": "submit-btn",
  "component": "Button",
  "child": "btn-text",
  "action": {
    "event": {
      "name": "submit_reservation",
      "context": {
        "time": {"path": "/reservationTime"},
        "size": {"path": "/partySize"}
      }
    }
  }
}
```

- **`name`**：供 agent 分支處理的穩定標識符。
- **`context`**：鍵值對對映。值可以是字面量，也可以用 `path` 從 data model 目前狀態中取值。

NOTE: **Context 與 Data Model**：Data Model 表示某個 surface 的完整狀態樹，而 action 中的 `context` 實際上是從該狀態中手動挑選出來的 **“視圖”** 或子集。這能為特定 event 精確提供所需值，讓 Agent 不必遍歷可能很大且複雜的 data model。

### Basic Catalog Function Validation（Checks）

Basic catalog 定義了一組可在 renderer 上執行的有限 checks。互動元件可以定義 `checks` 清單（使用 `common_types.json` 中的 [`Checkable`](../../specification/v0_9/json/common_types.json#L258-L270) schema）。對於 `Button`，如果任一 check 失敗，按鈕會在 renderer 上 **自動停用**。

- **UX 重點**：Validation check 用來管理 **UI State（使用者體驗）**，在無效互動發生前阻止它們。它們不能替代 **Data Integrity** check；後者仍必須在 agent 上執行。

這樣 UI 就能在使用者嘗試提交之前強制滿足要求，例如欄位不能為空。

```json
{
  "id": "submit-button",
  "component": "Button",
  "child": "submit-text",
  "checks": [
    {
      "condition": {
        "call": "required",
        "args": {"value": {"path": "/partySize"}}
      },
      "message": "Party size is required"
    }
  ],
  "action": {"event": {"name": "submit_booking"}}
}
```

## 本機狀態更新與 “Write” 契約

在 Event 被派發之前，renderer 就已經在本機管理 UI 狀態。A2UI 為所有輸入元件（如 `TextField`、`CheckBox` 或 `Slider`）定義了 **Read/Write Contract**。

1.  **Read（Model → View）**：元件渲染時，從 Data Model 中繫結的 `path` 拉取值。
2.  **Write（View → Model）**：使用者一發生互動（例如輸入字符或點擊複選框），renderer 就會 **立即** 將新值寫入本機 Data Model。

這意味著本機 model **始終** 是 UI 目前狀態的事實來源。這種 “View-to-Model” 同步完全在 renderer 上發生。Data model 只會在 event 發生時（例如點擊 Button）送出給 agent。

IMPORTANT: **同步更新**：本機 model 更新是 **同步** 的。這保證在任何 Event 解析其 `context` path 或封裝 `DataModelSync` payload 前，Data Model 都已經完全更新。輸入和點擊之間不存在競態條件；“Write” 總是先提交。

這種本機優先方式帶來顯著的 **性能收益**。因為同步即時且本機完成，開發者不需要在使用者輸入 `TextField` 時實作網路防抖，也不用擔心延遲抖動。網路會完全避免 “UI noise”（如逐個按鍵），直到使用者準備好派發正式 Event。

### 表單提交模式

這種分離支援穩健的表單提交模式：

- **Binding**：`TextField` 繫結到 `/reservationTime`。
- **Interaction**：使用者輸入 “7:00 PM”。本機 model 中的 `/reservationTime` 會立即更新。
- **Submission**：使用者點擊 “Book” 按鈕。按鈕的 Event 會從本機 model 解析 `path: "/reservationTime"`，並把目前值送出給 agent。

## 使用者互動流程

當使用者與元件互動時（例如點擊按鈕）：

1.  **Resolve**：renderer 根據本機 **Data Model** 解析 `context` 中的所有 `path` 引用。
2.  **Construct**：renderer 建構符合 [`client_to_server.json`](../../specification/v0_9/json/client_to_server.json) 的 `action` payload。
3.  **Dispatch**：payload 透過所選傳輸層送出（例如 A2A、WebSockets）。

### 範例：Action Payload（v0.9）

如果使用者點擊上面的按鈕，而 data model 包含 `{"reservationTime": "7:00 PM", "partySize": 4}`，renderer 會用 `action` key 送出訊息：

```json
{
  "version": "v0.9",
  "action": {
    "name": "submit_reservation",
    "surfaceId": "booking-surface",
    "sourceComponentId": "submit-btn",
    "timestamp": "2026-02-25T10:40:00Z",
    "context": {
      "time": "7:00 PM",
      "size": 4
    }
  }
}
```

IMPORTANT: **版本說明（v0.8 vs v0.9）**：在 v0.8 中，頂層 payload key 是 `userAction`（例如 `{"userAction": {...}}`）。v0.9 轉為上面更簡潔的 `action` key。標準協議 parser 會期待與 payload 宣告版本對應的 key。

## Agent 處理

Agent（或 Orchestrator）收到該 event 並采取行動。在 agentic 系統中，agent 通常會把 event 轉換為給 LLM 的隱藏使用者查詢。

**Agent 處理範例（Python）：**

```python
if action_name == "submit_reservation":
    time = context.get("time")
    size = context.get("size")
    # Feed this to the LLM
    query = f"User submitted a reservation for {size} people at {time}."
    response = await llm.generate(query)
```

## Renderer 到 Agent 的錯誤報告

除了使用者觸發的 Event 之外，renderer 還可以使用 [`client_to_server.json`](../../specification/v0_9/json/client_to_server.json) 中定義的 `error` payload，將系統級錯誤報告給 agent。

### 驗證失敗

如果 agent 送出的 A2UI JSON 违反 catalog schema 或協議規則，renderer 會送出 `VALIDATION_FAILED` 錯誤。這是 agentic 系統中的關鍵反饋回路：

```json
{
  "version": "v0.9",
  "error": {
    "code": "VALIDATION_FAILED",
    "surfaceId": "booking-surface",
    "path": "/components/0/children",
    "message": "Expected array of strings, got null."
  }
}
```

Agent 可以捕獲該錯誤、道歉（或在內部自我修正），然後重新送出修正後的 UI。

## Data Model Sync（v0.9）

A2UI v0.9 引入了強大的“無狀態”同步功能。它允許 renderer 自動把某個 surface 的 **完整 data model** 包含到它送出給 agent 的每條訊息的 metadata 中。

### 啟用同步

同步由 agent 在 surface 初始化期間請求。透過在 `createSurface` 訊息中設定 `sendDataModel: true`，agent 會指示 renderer 啟動同步循環。

```json
{
  "version": "v0.9",
  "createSurface": {
    "surfaceId": "booking-surface",
    "catalogId": "https://a2ui.org/catalogs/v1/basic.json",
    "sendDataModel": true
  }
}
```

### 線上傳輸中的同步

啟用同步後，renderer 不會把 data model 作為獨立訊息送出。相反，它會把 data model 作為 **metadata** 附加到外發的傳輸 envelope（例如 A2A message）上。

在 A2A（Agent-to-Agent）繫結中，data model 會放在 envelope `metadata` 欄位內的 `a2uiClientDataModel` 物件中。

**帶同步的 A2A Envelope 範例：**

```json
{
  "parts": [{"text": "Submit the reservation"}],
  "metadata": {
    "a2uiClientDataModel": {
      "version": "v0.9",
      "surfaces": {
        "booking-surface": {
          "reservationTime": "7:00 PM",
          "partySize": 4,
          "notes": "Window seat preferred"
        }
      }
    }
  }
}
```

### 為什么使用 Data Model Sync？

- **連接更簡單**：無需手動把每個輸入欄位對映到按鈕的 `context` 屬性。Agent 可以直接檢查 metadata，看到所有欄位的目前狀態。
- **無狀態 Agent**：Agent 不需要為每個使用者 session 維護本機狀態；它在每次互動中都會收到完整的目前上下文。
- **語音快捷方式**：允許使用者透過語音或文本觸發 Event（例如“好的，提交”），即使沒有點擊特定按鈕也可以。由於 agent 會隨文本訊息一起收到更新後的 data model，因此可以立即處理請求。

## Renderer Metadata 與 Capabilities

Agent 在安全送出 UI 之前，renderer 必須宣告它支援哪些 component catalog。這透過 `a2uiClientCapabilities` 物件處理。

### 宣告能力

Renderer 會在發給 agent 的訊息 **metadata** 中包含 `a2uiClientCapabilities` 物件（例如 A2A envelope 的 `metadata` 欄位）。

```json
{
  "v0.9": {
    "supportedCatalogIds": [
      "https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json",
      "https://my-company.com/catalogs/v1/custom.json"
    ],
    "inlineCatalogs": []
  }
}
```

- **`supportedCatalogIds`**：renderer 可以渲染的 catalog URI 數組。
- **`inlineCatalogs`**：（可選）用於開發或專門環境，允許內聯送出完整 catalog schema。

沒有這個握手，agent 無法確定 renderer 是否能處理要送出的特定元件。

## 傳輸與編碼

A2UI 與具體傳輸層無關，但最常透過 **A2A（Agent-to-Agent）** 或 WebSockets 使用。理解 payload 如何被包裹對實作至關重要。

### A2A 編碼

在標準 A2A 繫結中，A2UI 訊息編碼為 A2A **DataPart**。為了將其識別為 A2UI payload，part 必須用特定 metadata 包裹：

- **mimeType**：`application/json+a2ui`

`DataPart` 的 `data` 欄位包含 A2UI 訊息 **清單**。這允許在單一網路包中送出多個更新（例如 `createSurface` 後接 `updateComponents`）。

NOTE: **A2A 版本說明**：`data` 欄位中使用 **清單** 是在 **A2A v1.0** 中引入的。更早版本的 A2A 協議期望 `data` 欄位包含單一 JSON object。

```json
{
  "kind": "data",
  "metadata": {
    "mimeType": "application/json+a2ui"
  },
  "data": [
    {
      "version": "v0.9",
      "action": { ... }
    }
  ]
}
```

## 安全注意事項

A2UI 把安全、沙箱化通訊作為核心原則。由於協議依賴透過網路傳遞使用者狀態和互動觸發，它會對資料可見性和執行施加嚴格邊界。

### 沙箱化執行

A2UI 的一個核心賣點是透過限制獲得安全性。透過禁止 agent 執行任意程式碼（例如注入原始 JavaScript），A2UI 確保 agent 只能觸發預註冊行為。`functionCall` 機制提供了一種安全、沙箱化的方式，讓 agent 與 renderer 環境互動，而不會讓使用者暴露於惡意腳本。

### Data Model 隔離與 Orchestrator 路由

啟用 `sendDataModel: true` 後，renderer 會在外發訊息中包含 surface 的完整 data model。開發者必須理解這些資料的可見范圍：

- **點對點可見性**：只有接收傳輸 envelope 的後端（建立該 surface 的 Agent，或中間 Orchestrator）可以讀取此 payload。
- **Orchestrator 的責任**：在多 agent 架構中，中央 Orchestrator 通常會把使用者意圖路由到專門 sub-agent。Orchestrator 必須強制執行 **資料隔離**。它負責解析 `a2uiClientDataModel`、識別 `surfaceId`，並確保 data model 只傳給擁有該 surface 的特定 sub-agent。一個 agent surface 的資料絕不能泄漏給另一個 agent。

## 編排與路由

在多 agent 系統中，中央 **Orchestrator** 通常管理使用者與多個專門 sub-agent 之間的互動。一個關鍵挑戰是確保來自 renderer 的 `action` 訊息被路由回生成該 UI surface 的特定 sub-agent。

### Surface 所有權模式

為處理這個問題，orchestrator 必須維護 `surfaceId` 到其所屬 sub-agent 的對映。這通常儲存在 **Session State** 中。

#### 1. 對映所有權

當 sub-agent 發出 `createSurface` 訊息時，orchestrator 會攔截它並記錄所有權。

```python
# Simplified Orchestrator Logic: Record Ownership
def on_surface_created(surface_id, agent_name, session):
    # Store the mapping in the orchestrator's session state
    session.state.update({f"owner_of_{surface_id}": agent_name})
```

#### 2. 路由 Event

當 renderer 把 `action` 發回 orchestrator 時，orchestrator 會查找 `surfaceId`，並把請求轉交給正確的 sub-agent。

```python
# Simplified Orchestrator Logic: Route Event
async def handle_incoming_action(payload, session):
    action = payload.get("action")
    surface_id = action.get("surfaceId")

    # Lookup the owning agent
    target_agent = session.state.get(f"owner_of_{surface_id}")

    if target_agent:
        # Programmatically route the request to the sub-agent
        return transfer_to(target_agent)
```

該模式確保即使在複雜的多 agent 環境中，雙向通訊循環仍能為每個功能區域維持完整且有狀態。

### 透過 Metadata Stripping 防止資料泄漏

在多 agent 環境中，`a2uiClientDataModel` 可能包含多個由不同 agent 擁有的 surface 狀態。為防止敏感資料泄漏，orchestrator 必須 **strip** data model metadata，只保留被呼叫的特定 sub-agent 所擁有的 surface。

這最好在 outbound metadata interceptor 中實作：

```python
# Simplified Orchestrator Interceptor: Strip Data Model
async def intercept(self, request_payload, target_agent, session):
    message = request_payload["params"]["message"]
    data_model = message.get("metadata", {}).get("a2uiClientDataModel")

    if data_model:
        # Filter surfaces to only those owned by the target_agent
        filtered_surfaces = {
            surface_id: state for surface_id, state in data_model["surfaces"].items()
            if session.state.get(f"owner_of_{surface_id}") == target_agent.name
        }

        # Replace with the stripped data model
        message["metadata"]["a2uiClientDataModel"]["surfaces"] = filtered_surfaces

    return request_payload
```

透過剥離 metadata，orchestrator 可以確保 sub-agent 只收到自己有權查看的 data model 部分。

CAUTION: **安全風險：狀態抓取**：如果 Orchestrator 未剥離 `a2uiClientDataModel`，惡意或遭入侵的 sub-agent 可能會“抓取”其他 active surface 的狀態。例如，如果 orchestrator 泄漏了整個多 surface data model，一個天氣 sub-agent 就可能讀取银行 surface 的敏感資料。剥離是多 agent 系統中的強制安全要求。

---

## 綜合範例

### 1. Button Submit（顯式 Context）

此範例展示一個按鈕如何顯式收集需要送出的資料。

**元件定義：**

```json
{
  "id": "submit-button",
  "component": "Button",
  "child": "submit-text",
  "action": {
    "event": {
      "name": "submit_booking",
      "context": {
        "partySize": {"path": "/partySize"},
        "reservationTime": {"path": "/reservationTime"}
      }
    }
  }
}
```

**生成的 Action Payload：**
Agent 會收到一個 `action` 物件，其中 `partySize` 和 `reservationTime` 直接位於 `context` 欄位中。

### 2. 語音提交（Data Model Sync）

在這個場景中，使用者沒有點擊按鈕，而是說“Okay, submit the form.”

**初始化：**
Agent 用 `sendDataModel: true` 建立了 surface：

```json
{
  "version": "v0.9",
  "createSurface": {
    "surfaceId": "booking-surface",
    "catalogId": "...",
    "sendDataModel": true
  }
}
```

**Renderer 傳輸：**
Renderer 送出一條 A2A 訊息，其中包含使用者文本以及 metadata 中的 data model：

```json
{
  "parts": [{"text": "Okay, submit the form"}],
  "metadata": {
    "a2uiClientDataModel": {
      "version": "v0.9",
      "surfaces": {
        "booking-surface": {
          "partySize": 4,
          "reservationTime": "7:00 PM"
        }
      }
    }
  }
}
```

**Agent 處理：**
Agent 看到使用者意圖（“submit”），並查看 `metadata` 找到 `partySize` 和 `reservationTime` 的目前值，從而無需進一步澄清即可完成任務。
