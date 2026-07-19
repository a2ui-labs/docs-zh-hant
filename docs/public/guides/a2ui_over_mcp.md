# 透過 Model Context Protocol (MCP) 使用 A2UI

本指南說明如何使用 **A2UI** 的宣告式語法，透過 **Model Context Protocol (MCP)** 上的 Tools 與 Resources 建立豐富且可互動的介面。

範例請見 [MCP Samples](../../samples/agent/mcp)。

## 目錄協商

在 server 能向 client 傳送 A2UI 之前，雙方必須先建立對該協定的共同支援，並確定可用的 catalog。依照你的系統架構，這項能力協商可以透過兩種方式完成：在初始連線握手期間，或在每一則訊息的基礎上處理。

### 選項 A：在 MCP 初始化期間進行 catalog 握手

由於 MCP 是有狀態的 session 協定，最有效率的作法是在建立連線時只宣告一次能力。client 會在標準 `initialize` 請求的 `capabilities` 物件中宣告其 A2UI 支援情況（通常位於 experimental 或自訂 key 底下）。server 會在整個 session 期間保存這個狀態。

初始化請求範例：

```json
{
  "jsonrpc": "2.0",
  "method": "initialize",
  "id": "init-123",
  "params": {
    "protocolVersion": "2025-11-25",
    "clientInfo": {
      "name": "a2ui-enabled-client",
      "version": "1.0.0"
    },
    "capabilities": {
      "a2ui": {
        "clientCapabilities": {
          "v0.10": {
            "supportedCatalogIds": [
              "https://a2ui.org/specification/v0_10/basic_catalog.json"
            ]
          }
        }
      }
    }
  }
}
```

### 選項 B：在每一則 MCP 訊息上進行 catalog 握手（適用於無狀態 server）

如果你的架構要求 MCP Server 完全保持無狀態，client 可以在每次 tool call 請求的 `_meta` 欄位中傳入自己的 A2UI 版本與 catalog 支援資訊。server 會即時讀取這些中繼資料，決定回應 UI 要使用哪個 catalog。

呼叫請求中繼資料範例：

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "id": "id-123",
  "params": {
    "name": "generate_report",
    "arguments": { "date": "2026-03-01" },
    "_meta": {
      "a2ui": {
        "clientCapabilities": {
          "v0.10": {
            "supportedCatalogIds": [
              "https://a2ui.org/specification/v0_10/basic_catalog.json"
            ],
            "inlineCatalogs": []
          }
        }
      }
    }
  }
}
```

## 將 A2UI 內容作為嵌入式資源回傳

Embedded Resources 能讓 Tool 直接回傳與該次回應綁定的 UI 版面，而不需要 server 端儲存或追蹤。

- **URI**：必須使用 `a2ui://` 前綴，並搭配具描述性的名稱識別碼（例如 `a2ui://training-plan-page`）。
- **MIME Type**：必須使用 `application/a2ui+json`。這可確保 MCP client 會把 payload 導向 A2UI renderer，而不是直接把原始 JSON 顯示給使用者。

#### Python 實作範例

```python
import mcp.types as types

@self.tool()
def get_hello_world_ui():
    a2ui_payload = [
        {
            "version": "v0.10",
            "createSurface": {
                "surfaceId": "default",
                "catalogId": "https://a2ui.org/specification/v0_10/basic_catalog.json"
            }
        },
        {
            "version": "v0.10",
            "updateComponents": {
                "surfaceId": "default",
                "components": [
                    {
                        "id": "root",
                        "component": "Text",
                        "text": "Hello World!"
                    }
                ]
            }
        }
    ]

    # Wrap A2UI as an Embedded Resource
    a2ui_resource = types.EmbeddedResource(
        type="resource",
        resource=types.TextResourceContents(
            uri="a2ui://training-plan-page",
            mimeType="application/a2ui+json",
            text=json.dumps(a2ui_payload),
        )
    )

    text_content = types.TextContent(
        type="text",
        text="Here is your generated training plan summary..."
    )

    return types.CallToolResult(content=[text_content, a2ui_resource])
```

## 處理使用者動作

互動式元件（例如 `Button`）允許把 `actions` 傳回 server。

#### 1. 帶有 Action 的 A2UI JSON

```json
{
  "id": "confirm-button",
  "component": {
    "Button": {
      "child": "confirm-button-text",
      "action": {
        "event": {
          "name": "confirm_booking",
          "context": {
            "start": "/dates/start",
            "end": "/dates/end"
          }
        }
      }
    }
  }
}
```

#### 2. A2UI Action 的 MCP Payload

當按鈕被點擊時，client 會根據 surface 的繫結狀態，解析任何絕對或相對路徑模型（例如 `/dates/start` 或 `/dates/end`），並將其轉換為 MCP tool call 引數。

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "id": "id-456",
  "params": {
    "name": "a2ui_action",
    "arguments": {
      "name": "confirm_booking",
      "context": {
        "start": "2026-03-20",
        "end": "2026-03-25"
      }
    }
  }
}
```

#### 3. Action Handler MCP Server Tool

MCP server 會接收這個 tool call，並執行對應的 handler。

```python
@self.tool()
async def a2ui_action(action_payload: Dict[str, Any]) -> Dict[str, Any]:
    if action_payload["name"] == "confirm_booking":
        return {"response": f"Booking confirmed for {action_payload['context']['start']} to {action_payload['context']['end']}."}
    raise ValueError(f"Unknown action: {action_payload['name']}")
```

## 錯誤處理

和處理使用者互動類似，MCP server 也可以接收來自 client 的錯誤。

#### 1. A2UI 錯誤 MCP Payload

當 client 在 A2UI payload 上遇到錯誤時，可以送出一個 error MCP payload 給 server。

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "id": "id-789",
  "params": {
    "name": "a2ui_error",
    "arguments": {
      "code": "INVALID_JSON",
      "message": "Failed to parse A2UI payload.",
      "surfaceId": "default",
    }
  }
}
```

#### 2. Error Handler MCP Server Tool

MCP server 會接收這個 tool call，並執行對應的 handler。

```python
@self.tool()
async def a2ui_error(error_payload: Dict[str, Any]) -> Dict[str, Any]:
    return {"response": f"Received A2UI error: {error_payload['error']}."}
```

## 口語化與可見性控制

你可以透過 MCP 的 **Resource Annotations** 來控制後續 assistant 回合是否可以「閱讀」或解讀後端 payload。

```python
a2ui_resource = types.EmbeddedResource(
    type="resource",
    resource=types.TextResourceContents(
        uri="a2ui://training-plan-page",
        mimeType="application/a2ui+json",
        text=json.dumps(a2ui_payload)
    ),
    # Hide the raw JSON from the LLM, but show the UI to the user
    annotations=types.Annotations(audience=["user"])
)
```

- **空白 Audience**：元件同時對使用者與 LLM 模型可見。
- **Audience `user`**：在視圖畫面中顯示項目時需要。
- **Audience `assistant`**：允許內容口語化，並在連續回合中觸發提示輸入。停用 assistant 會限制 Agent 的上下文解析，但仍保留離散且安全的資料洩漏控制。
