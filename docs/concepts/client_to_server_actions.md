# A2UI Client-to-Server 操作

A2UI 的互動性仰賴雙向通訊迴圈。Agent 透過串流元件與資料更新來驅動 UI，而 Client 則透過 **Actions** 與 **Data Model Synchronization** 把使用者意圖回傳給 Agent。

## 操作架構

Actions 讓 UI 元件可以觸發行為。它們定義在 [`common_types.json`](../specification/v0_9/json/common_types.json) 中的 `Action` schema 內，分成兩種：

1. **Server Events**：送到 Agent 端處理，例如點擊「Submit」。
2. **Local Function Calls**：完全在 client 端執行，例如開啟 URL。

### Schema 中的操作連接方式

像 `Button` 這類元件會暴露 `action` 屬性。以下是 server event 的連接方式：

```json
{
  "id": "submit-btn",
  "component": "Button",
  "child": "btn-text",
  "action": {
    "event": {
      "name": "submit_reservation",
      "context": {
        "time": { "path": "/reservationTime" },
        "size": { "path": "/partySize" }
      }
    }
  }
}
```

- **`name`**：給 agent 用來分派處理的穩定識別碼。
- **`context`**：鍵值對映射。值可以是字面值，也可以透過 `path` 從目前的資料模型狀態取值。

> [!NOTE]
> **Context 與 Data Model**：Data Model 表示某個 surface 的整個狀態樹，而 action 中的 `context` 實際上是從該狀態中挑選出的 **「視圖」** 或子集合。這能讓 Agent 只拿到特定事件真正需要的值，而不必遍歷可能很大、很複雜的資料模型。

### 本地操作與 Server Events

雖然 Server Event 是與 agent 互動的主要方式，但 **Local Actions** 可以讓 client 直接在本機端做出反應，而不需要網路來回。這對回應速度敏感的 UI 模式非常重要。

```json
{
  "id": "help-btn",
  "component": "Button",
  "child": "help-text",
  "action": {
    "functionCall": {
      "call": "openUrl",
      "args": { "url": "https://a2ui.org/help" }
    }
  }
}
```

Local Actions 常見用途包括：

- **驗證**：在表單送出前先驗證輸入。
- **格式化**：使用 `formatString` 格式化本機顯示值。

### 基礎目錄操作驗證（Checks）

基礎目錄定義了一組可在 client 端執行的有限檢查。互動元件可以定義一個 `checks` 清單（使用 [`common_types.json`](../specification/v0_9/json/common_types.json) 中的 `Checkable` schema）。對於 `Button` 而言，只要有任一檢查失敗，按鈕就會在 client 端 **自動停用**。

- **UX 重點**：操作檢查是為了管理 **UI 狀態（使用者體驗）**，在無效互動發生前先阻止它們。它們不能取代 **資料完整性** 檢查，後者仍然必須在 server 端執行。

這讓 UI 可以先強制要求條件成立，例如欄位不能為空，然後使用者才真正送出。

```json
{
  "id": "submit-button",
  "component": "Button",
  "child": "submit-text",
  "checks": [
    {
      "condition": {
        "call": "required",
        "args": { "value": { "path": "/partySize" } }
      },
      "message": "Party size is required"
    }
  ],
  "action": { "event": { "name": "submit_booking" } }
}
```

## 本地狀態更新與 "Write" 合約

在 action 還沒送出之前，client 就已經在本機管理 UI 狀態了。A2UI 為所有輸入元件（例如 `TextField`、`CheckBox` 或 `Slider`）定義了一個 **Read/Write Contract**。

1. **Read（Model → View）**：元件渲染時，會從 Data Model 中綁定的 `path` 讀取值。
2. **Write（View → Model）**：只要使用者互動發生，例如輸入字元或點擊核取方塊，client 就會 **立即** 把新值寫回本機 Data Model。

這代表本機模型永遠是 UI 目前狀態的唯一真實來源。這種「View-to-Model」同步完全在 client 端完成。只有在使用者動作（例如點擊 Button）被觸發時，這個狀態才會再同步回 server。

> [!IMPORTANT]
> **同步更新**：本機模型更新是 **同步** 的。這可確保在任何 action 解析 `context` path 或 `DataModelSync` payload 被封裝之前，Data Model 已經完整更新。輸入與點擊之間不會產生競態條件；`Write` 一定會先提交。

這種以本機為優先的做法帶來明顯的 **效能優勢**。因為同步是即時且本機完成的，開發者不需要在使用者輸入 `TextField` 時實作網路去抖動，或擔心延遲造成的抖動。直到使用者準備正式送出 Action 為止，網路都不會被「UI 雜訊」（例如逐字鍵入）污染。

### 表單送出模式

這種分離讓表單送出模式變得很穩健：

- **綁定**：`TextField` 綁定到 `/reservationTime`。
- **互動**：使用者輸入「7:00 PM」。本機模型中 `/reservationTime` 會立即更新。
- **送出**：使用者點擊「Book」按鈕。按鈕的 action 會從本機模型解析 `path: "/reservationTime"`，並把目前值送到 server。

## 使用者互動流程

當使用者與元件互動時，例如點擊按鈕：

1. **解析**：client 會根據本機 **Data Model** 解析 `context` 中所有 `path` 參照。
2. **建構**：client 建立符合 [`client_to_server.json`](../specification/v0_9/json/client_to_server.json) 的 `action` payload。
3. **派送**：payload 透過所選傳輸方式送出，例如 A2A 或 WebSockets。

### 範例：Action Payload（v0.9）

如果使用者點擊上面的按鈕，而資料模型中包含 `{"reservationTime": "7:00 PM", "partySize": 4}`，client 會使用 `action` key 送出訊息：

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

> [!IMPORTANT]
> **版本說明（v0.8 vs v0.9）**：在 v0.8 中，頂層 payload key 是 `userAction`（例如 `{"userAction": {...}}`）。v0.9 改成更簡潔的 `action` key，如上所示。標準協議 parser 會依照 payload 中宣告的版本來期待對應的 key。

## Agent 處理

Agent（或 Orchestrator）收到這個事件後會加以處理。在 agentic 系統中，這通常代表把事件轉換成一個隱藏的使用者查詢，再交給 LLM。

**Agent 處理範例（Python）：**

```python
if action_name == "submit_reservation":
    time = context.get("time")
    size = context.get("size")
    # 將這些資訊送給 LLM
    query = f"User submitted a reservation for {size} people at {time}."
    response = await llm.generate(query)
```

## Client-to-Server 錯誤回報

除了使用者動作之外，client 也可以透過 [`client_to_server.json`](../specification/v0_9/json/client_to_server.json) 中定義的 `error` payload，把系統層級的錯誤回報給 server。

### 驗證失敗

如果 agent 送出的 A2UI JSON 違反目錄 schema 或協議規則，client 會送出 `VALIDATION_FAILED` 錯誤。這對 agentic 系統來說是關鍵的回饋迴圈：

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

agent 可以捕捉這個錯誤、致歉（或在內部自行修正），然後重新送出修正後的 UI。

## Data Model 同步（v0.9）

在 A2UI v0.9 中，我們引入了一個強大的「stateless」同步功能。這讓 client 可以自動把某個 surface 的 **完整 data model** 附加到它每次送往 server 的訊息 metadata 中。

### 啟用同步

同步會在 surface 初始化時由 agent 要求。只要在 `createSurface` 訊息中設定 `sendDataModel: true`，agent 就會指示 client 開始同步迴圈。

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

### 同步如何在網路上傳遞

當同步啟用時，client 不會把 data model 當成獨立訊息送出。相反地，它會把它作為 **metadata** 附加到外送的 transport envelope 上，例如 A2A 訊息。

在 A2A（Agent-to-Agent）綁定中，data model 會放在 envelope 的 `metadata` 欄位中的 `a2uiClientDataModel` 物件裡。

**A2A 同步封包範例：**

```json
{
  "parts": [{ "text": "Submit the reservation" }],
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

### 為什麼要使用 Data Model 同步？

- **連接更簡單**：你不需要手動把每個輸入欄位映射到按鈕的 `context` 屬性。agent 只要檢查 metadata，就能看到所有欄位的目前狀態。
- **Stateless Agents**：agent 不需要為每個使用者 session 維護本地狀態；每次互動都會收到完整的即時 context。
- **口語捷徑**：即使沒有點擊特定按鈕，使用者也能用語音或文字觸發操作，例如「好，送出」。由於 agent 會和文字訊息一起收到更新後的 data model，所以能立刻處理請求。

## Client 中繼資料與能力

在 agent 安全地送出 UI 之前，client 必須先宣告自己支援哪些元件目錄。這是透過 `a2uiClientCapabilities` 物件完成的。

### 宣告能力

client 會在送給 server 的訊息 **metadata** 中包含 `a2uiClientCapabilities` 物件，例如放在 A2A envelope 的 `metadata` 欄位裡。

```json
{
  "v0.9": {
    "supportedCatalogIds": [
      "https://a2ui.org/specification/v0_9/basic_catalog.json",
      "https://my-company.com/catalogs/v1/custom.json"
    ],
    "inlineCatalogs": []
  }
}
```

- **`supportedCatalogIds`**：client 可以渲染的目錄 URI 陣列。
- **`inlineCatalogs`**：可選，用於開發或特殊環境，允許直接內嵌完整目錄 schema。

若沒有這個握手，agent 就無法確定 renderer 是否能處理它送出的特定元件。

## 傳輸與編碼

A2UI 不綁定特定傳輸方式，但最常透過 **A2A（Agent-to-Agent）** 或 WebSockets 使用。理解 payload 如何被包裝，對實作非常重要。

### A2A 編碼

在標準 A2A 綁定中，A2UI 訊息會被編碼為 A2A **DataPart**。為了識別它是 A2UI payload，該 part 必須包上特定 metadata：

- **mimeType**：`application/json+a2ui`

`DataPart` 的 `data` 欄位包含一個 **A2UI 訊息列表**。這樣就可以在單一網路封包中送出多個更新，例如先 `createSurface` 再 `updateComponents`。

> [!NOTE]
> **A2A 版本**：`data` 欄位中使用 **list** 的做法是在 **A2A v1.0** 才加入。較早版本的 A2A 協議預期 `data` 欄位只會包含單一 JSON object。

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

## 安全考量

A2UI 的核心原則之一，就是安全且受 sandbox 保護的通訊。由於協議需要在網路上傳遞使用者狀態與互動觸發，因此它對資料可見性與執行都有嚴格邊界。

### Sandbox 執行

A2UI 的主要賣點之一，是透過限制來達到安全。因為它禁止 agent 執行任意程式碼（例如注入原始 JavaScript），所以 agent 只能觸發預先註冊的 client-side 行為。`functionCall` 機制提供了一種安全、受 sandbox 保護的方式，讓 agent 能與 client 環境互動，而不會暴露使用者給惡意腳本。

### Data Model 隔離與 Orchestrator 路由

當啟用 `sendDataModel: true` 時，client 會把 surface 的完整 data model 包含在外送訊息中。開發者必須理解這些資料的可見性：

- **點對點可見性**：只有接收 transport envelope 的後端（建立該 surface 的 Agent，或中介的 Orchestrator）可以讀取這個 payload。
- **Orchestrator 的責任**：在多 agent 架構中，中央 Orchestrator 常會把使用者意圖轉送給專門的 sub-agent。Orchestrator 必須強制執行 **資料隔離**。它有責任解析 `a2uiClientDataModel`、識別 `surfaceId`，並確保 data model 只會傳給擁有該 surface 的特定 sub-agent。某個 agent surface 的資料絕不能洩漏到另一個 agent。

## 協調與路由

在多 agent 系統中，中央 **Orchestrator** 常常負責協調使用者與多個專門 sub-agent 之間的互動。關鍵挑戰在於，必須確保 client 發出的 `action` 訊息能被路由回產生該 UI surface 的特定 sub-agent。

### Surface 擁有權模式

為了處理這件事，orchestrator 必須維護 `surfaceId` 到其所屬 sub-agent 的對應。這通常會存放在 **Session State** 中。

#### 1. 建立擁有權映射

當 sub-agent 發出 `createSurface` 訊息時，orchestrator 會攔截它並記錄擁有權。

```python
# 簡化的 Orchestrator 邏輯：記錄擁有權
def on_surface_created(surface_id, agent_name, session):
    # 將映射存入 orchestrator 的 session state
    session.state.update({f"owner_of_{surface_id}": agent_name})
```

#### 2. 路由使用者操作

當 client 把 `action` 送回 orchestrator 時，orchestrator 會查找 `surfaceId`，並把請求轉交給正確的 sub-agent。

```python
# 簡化的 Orchestrator 邏輯：路由 action
async def handle_incoming_action(payload, session):
    action = payload.get("action")
    surface_id = action.get("surfaceId")

    # 查找擁有該 surface 的 agent
    target_agent = session.state.get(f"owner_of_{surface_id}")

    if target_agent:
        # 以程式方式把請求路由到 sub-agent
        return transfer_to(target_agent)
```

這種模式可確保即使在複雜的多 agent 環境中，雙向通訊迴圈仍能維持完整，且每個功能區塊都保有狀態。

### 透過 metadata 剝離避免資料外洩

在多 agent 環境中，`a2uiClientDataModel` 可能包含多個不同 agent 擁有的 surface 狀態。為了避免敏感資料外洩，orchestrator 必須 **剝離** data model metadata，只保留被呼叫的特定 sub-agent 所擁有的 surface。

最好的做法是透過 outbound metadata interceptor：

```python
# 簡化的 Orchestrator interceptor：剝離 data model
async def intercept(self, request_payload, target_agent, session):
    message = request_payload["params"]["message"]
    data_model = message.get("metadata", {}).get("a2uiClientDataModel")

    if data_model:
        # 只保留目標 agent 擁有的 surfaces
        filtered_surfaces = {
            surface_id: state for surface_id, state in data_model["surfaces"].items()
            if session.state.get(f"owner_of_{surface_id}") == target_agent.name
        }

        # 以剝離後的 data model 取代原內容
        message["metadata"]["a2uiClientDataModel"]["surfaces"] = filtered_surfaces

    return request_payload
```

透過剝離 metadata，orchestrator 能確保 sub-agent 只會收到它有權限查看的那一部分 data model。

> [!CAUTION]
> **安全風險：狀態竊取**：如果 Orchestrator 沒有剝離 `a2uiClientDataModel`，惡意或遭入侵的 sub-agent 可能會「竊取」其他 active surface 的狀態。例如，一個天氣 sub-agent 可能讀到銀行 surface 的敏感資料，只因為 orchestrator 洩漏了整個多 surface data model。剝離 metadata 是多 agent 系統中的必要安全要求。

---

## 完整範例

### 1. Button Submit（明確 context）

這個範例展示了按鈕如何明確收集它需要送出的資料。

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
        "partySize": { "path": "/partySize" },
        "reservationTime": { "path": "/reservationTime" }
      }
    }
  }
}
```

**產生的 Action Payload：**
agent 會收到一個 `action` 物件，而 `partySize` 和 `reservationTime` 會直接出現在 `context` 欄位中。

### 2. 口語送出（Data Model 同步）

在這個情境中，使用者沒有點按鈕，而是直接說「好，送出表單。」

**初始化：**
agent 在建立 surface 時已設置 `sendDataModel: true`：

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

**Client 傳輸：**
client 送出一則 A2A 訊息，包含使用者文字與 metadata 中的 data model：

```json
{
  "parts": [{ "text": "Okay, submit the form" }],
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

**Agent 動作：**
agent 看到使用者意圖（「submit」），並查看 `metadata` 找出 `partySize` 與 `reservationTime` 的目前值，因而能在不需要進一步確認的情況下完成操作。
