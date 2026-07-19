# 訊息類型

本參考文件提供所有 A2UI 訊息類型的詳細說明。

## 訊息格式

所有 A2UI 訊息都是以 JSON Lines（JSONL）形式傳送的 JSON 物件。每一行只包含一則訊息。

=== "v0.8 訊息類型"

    - `beginRendering` — 通知客戶端開始渲染 surface
    - `surfaceUpdate` — 新增或更新元件
    - `dataModelUpdate` — 更新應用狀態
    - `deleteSurface` — 移除 surface

=== "v0.9 訊息類型"

    - `createSurface` — 建立 surface 並指定其 catalog
    - `updateComponents` — 新增或更新元件
    - `updateDataModel` — 更新應用狀態
    - `deleteSurface` — 移除 surface

    所有 v0.9 訊息都包含 `"version": "v0.9"` 欄位。

---

## beginRendering（v0.8）/ createSurface（v0.9）

通知客戶端初始化並渲染一個 surface。

=== "v0.8 — `beginRendering`"

    ### 架構

    ```typescript
    {
      beginRendering: {
        surfaceId: string;      // Required: Unique surface identifier
        root: string;           // Required: The ID of the root component to render
        catalogId?: string;     // Optional: URL of component catalog
        styles?: object;        // Optional: Styling information
      }
    }
    ```

    ### 屬性

    | 屬性        | 類型   | 必填     | 說明                                                                                     |
    | ----------- | ------ | -------- | --------------------------------------------------------------------------------------- |
    | `surfaceId` | string | ✅        | 此 surface 的唯一識別符。                                                               |
    | `root`      | string | ✅        | 此 surface 的 UI tree 根元件 `id`。                                                     |
    | `catalogId` | string | ❌        | 元件 catalog 的識別符。若省略，預設為 v0.8 標準 catalog。                               |
    | `styles`    | object | ❌        | 依 catalog 定義的 UI 樣式資訊。                                                         |

    ### 範例

    ```json
    {
      "beginRendering": {
        "surfaceId": "main",
        "root": "root-component"
      }
    }
    ```

    **使用自訂 catalog：**

    ```json
    {
      "beginRendering": {
        "surfaceId": "custom-ui",
        "root": "root-custom",
        "catalogId": "https://my-company.com/a2ui/v0.8/my_custom_catalog.json"
      }
    }
    ```

    必須在元件定義之後送出。客戶端會緩衝 `surfaceUpdate` 與 `dataModelUpdate` 訊息，直到收到 `beginRendering` 為止。

=== "v0.9 — `createSurface`"

    ### 架構

    ```typescript
    {
      version: "v0.9";
      createSurface: {
        surfaceId: string;      // Required: Unique surface identifier
        catalogId: string;      // Required: URL of component catalog
        theme?: object;         // Optional: Theme configuration
        sendDataModel?: boolean; // Optional: Request client to send data model updates
      }
    }
    ```

    ### 屬性

    | 屬性            | 類型    | 必填     | 說明                                                             |
    | --------------- | ------- | -------- | --------------------------------------------------------------- |
    | `surfaceId`     | string  | ✅        | 此 surface 的唯一識別符。                                       |
    | `catalogId`     | string  | ✅        | 元件 catalog 的識別符。                                         |
    | `theme`         | object  | ❌        | 主題設定（例如 `primaryColor`）。                               |
    | `sendDataModel` | boolean | ❌        | 若為 true，客戶端會把資料模型變更回傳給伺服器。                 |

    ### 範例

    ```json
    {
      "version": "v0.9",
      "createSurface": {
        "surfaceId": "main",
        "catalogId": "https://a2ui.org/specification/v0_9/basic_catalog.json"
      }
    }
    ```

    在 v0.9 中，`createSurface` 取代了 `beginRendering`。根元件由慣例決定：`updateComponents` 中必須有一個元件的 `"id"` 為 `"root"`。`catalogId` 為必填。

---

## surfaceUpdate（v0.8）/ updateComponents（v0.9）

在一個 surface 中新增或更新元件。

=== "v0.8 — `surfaceUpdate`"

    ### 架構

    ```typescript
    {
      surfaceUpdate: {
        surfaceId: string;        // Required: Target surface
        components: Array<{       // Required: List of components
          id: string;             // Required: Component ID
          component: {            // Required: Wrapper for component data
            [ComponentType]: {    // Required: Exactly one component type
              ...properties       // Component-specific properties
            }
          }
        }>
      }
    }
    ```

    ### 屬性

    | 屬性         | 類型   | 必填     | 說明                         |
    | ------------ | ------ | -------- | ------------------------------ |
    | `surfaceId`  | string | ✅        | 要更新的 surface ID。          |
    | `components` | array  | ✅        | 元件定義陣列。                 |

    ### 元件物件

    `components` 陣列中的每個物件都必須包含：

    - `id`（string，必填）：在此 surface 內的唯一識別符
    - `component`（object，必填）：包裝物件，且只包含一個鍵，也就是元件類型（例如 `Text`、`Button`）。

    ### 範例

    **單一元件：**

    ```json
    {
      "surfaceUpdate": {
        "surfaceId": "main",
        "components": [
          {
            "id": "greeting",
            "component": {
              "Text": {
                "text": {"literalString": "Hello, World!"},
                "usageHint": "h1"
              }
            }
          }
        ]
      }
    }
    ```

    **多個元件（adjacency list）：**

    ```json
    {
      "surfaceUpdate": {
        "surfaceId": "main",
        "components": [
          {
            "id": "root",
            "component": {
              "Column": {
                "children": {"explicitList": ["header", "body"]}
              }
            }
          },
          {
            "id": "header",
            "component": {
              "Text": {
                "text": {"literalString": "Welcome"}
              }
            }
          },
          {
            "id": "body",
            "component": {
              "Card": {
                "child": "content"
              }
            }
          },
          {
            "id": "content",
            "component": {
              "Text": {
                "text": {"path": "/message"}
              }
            }
          }
        ]
      }
    }
    ```

    **更新既有元件：**

    ```json
    {
      "surfaceUpdate": {
        "surfaceId": "main",
        "components": [
          {
            "id": "greeting",
            "component": {
              "Text": {
                "text": {"literalString": "Hello, Alice!"},
                "usageHint": "h1"
              }
            }
          }
        ]
      }
    }
    ```

    `id: "greeting"` 的元件會被更新，而不會重複建立。

    ### 使用注意事項

    - 必須在 `beginRendering` 訊息中指定一個元件作為 `root`，用來當作樹根。
    - 元件以 adjacency list 形式組織，也就是帶有 ID 參照的扁平結構。
    - 傳送一個已存在 ID 的元件會更新該元件。
    - 子元件以 ID 參照。
    - 元件可以逐步新增（串流式）。

=== "v0.9 — `updateComponents`"

    ### 架構

    ```typescript
    {
      version: "v0.9";
      updateComponents: {
        surfaceId: string;        // Required: Target surface
        components: Array<{       // Required: List of components
          id: string;             // Required: Component ID
          component: string;      // Required: Component type name
          ...properties           // Component-specific properties (flat)
        }>
      }
    }
    ```

    ### 屬性

    | 屬性         | 類型   | 必填     | 說明                         |
    | ------------ | ------ | -------- | ------------------------------ |
    | `surfaceId`  | string | ✅        | ID of the surface to update    |
    | `components` | array  | ✅        | Array of component definitions |

    ### 元件物件

    在 v0.9 中，元件結構更扁平：

    - `id`（string，必填）：在此 surface 內的唯一識別符
    - `component`（string，必填）：元件類型名稱（例如 `"Text"`、`"Button"`）
    - 其他所有屬性都位於元件物件的頂層。

    ### 範例

    **單一元件：**

    ```json
    {
      "version": "v0.9",
      "updateComponents": {
        "surfaceId": "main",
        "components": [
          {
            "id": "greeting",
            "component": "Text",
            "text": "Hello, World!",
            "variant": "h1"
          }
        ]
      }
    }
    ```

    **多個元件：**

    ```json
    {
      "version": "v0.9",
      "updateComponents": {
        "surfaceId": "main",
        "components": [
          {
            "id": "root",
            "component": "Column",
            "children": ["header", "body"]
          },
          {
            "id": "header",
            "component": "Text",
            "text": "Welcome"
          },
          {
            "id": "body",
            "component": "Card",
            "child": "content"
          },
          {
            "id": "content",
            "component": "Text",
            "text": {"path": "/message"}
          }
        ]
      }
    }
    ```

    **更新既有元件：**

    ```json
    {
      "version": "v0.9",
      "updateComponents": {
        "surfaceId": "main",
        "components": [
          {
            "id": "greeting",
            "component": "Text",
            "text": "Hello, Alice!",
            "variant": "h1"
          }
        ]
      }
    }
    ```

    ### 使用注意事項

    - 必須有一個元件的 `"id": "root"` 作為樹根（這是慣例，不是獨立訊息欄位）。
    - 元件類型是字串（`"component": "Text"`），而不是包裝物件。
    - 屬性會直接平鋪在元件物件上（不再包在 type 鍵之下）。
    - 資料繫結使用 `{"path": "/pointer"}`（JSON Pointer）——鍵名與 v0.8 相同，但採用標準 JSON Pointer 路徑。
    - 元件可以逐步新增（串流式）。

### 錯誤

| Error                  | Cause                                  | Solution                                                                                                               |
| ---------------------- | -------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Surface 已存在         | `surfaceId` 已被使用                   | 確保 `surfaceId` 在該 renderer 的生命週期內全域唯一。若使用具有子智慧體的 orchestrator，orchestrator 有責任視需要管理 surface ID，以避免衝突。 |
| 找不到 Surface         | `surfaceId` 不存在                     | 確保 `surfaceId` 與已建立的 surface 相符。在 v0.8 中，surface 是隱式建立的，但 v0.9 以上版本則需要 `createSurface`。   |
| 無效元件類型           | 不存在的元件類型                       | 檢查該元件類型是否存在於協商後的 catalog 中。                                                                          |
| 無效屬性               | 這個類型沒有該屬性                     | 依 catalog schema 驗證。                                                                                               |
| 循環參照               | 元件把自己當成子元件                   | 修正元件階層。                                                                                                         |

---

## dataModelUpdate（v0.8）/ updateDataModel（v0.9）

更新元件所繫結的資料模型。

=== "v0.8 — `dataModelUpdate`"

    ### 架構

    ```typescript
    {
      dataModelUpdate: {
        surfaceId: string;      // Required: Target surface
        path?: string;          // Optional: Path to a location in the model
        contents: Array<{       // Required: Data entries
          key: string;
          valueString?: string;
          valueNumber?: number;
          valueBoolean?: boolean;
          valueMap?: Array<{...}>;
        }>
      }
    }
    ```

    ### 屬性

    | 屬性        | 類型   | 必填     | 說明                                                                                                  |
    | ----------- | ------ | -------- | ---------------------------------------------------------------------------------------------------- |
    | `surfaceId` | string | ✅        | 要更新的 surface ID。                                                                                |
    | `path`      | string | ❌        | 資料模型內某個位置的路徑（例如 `user`）。若省略，更新會套用到 root。                                   |
    | `contents`  | array  | ✅        | 以 adjacency list 表示的資料項目陣列。每個項目都有一個 `key` 與一個具型別的 `value*` 屬性。          |

    ### `contents` adjacency list

    `contents` 陣列是一組 key-value 配對。陣列中的每個物件都必須有一個 `key`，以及且僅有一個 `value*` 屬性（`valueString`、`valueNumber`、`valueBoolean` 或 `valueMap`）。這種結構對 LLM 友善，也避免了從通用 `value` 欄位推斷型別的問題。

    ### 範例

    **Initialize entire model:**

    ```json
    {
      "dataModelUpdate": {
        "surfaceId": "main",
        "contents": [
          {
            "key": "user",
            "valueMap": [
              { "key": "name", "valueString": "Alice" },
              { "key": "email", "valueString": "alice@example.com" }
            ]
          },
          { "key": "items", "valueMap": [] }
        ]
      }
    }
    ```

    **Update nested property:**

    ```json
    {
      "dataModelUpdate": {
        "surfaceId": "main",
        "path": "user",
        "contents": [
          { "key": "email", "valueString": "alice@newdomain.com" }
        ]
      }
    }
    ```

    這會變更 `/user/email`，但不會影響 `/user/name`。

    ### 使用注意事項

    - 資料模型是以 surface 為單位。
    - 當繫結資料變更時，元件會自動重新渲染。
    - 優先更新特定路徑的局部內容，而不是整個模型。
    - 使用具型別的 value 欄位（`valueString`、`valueNumber`、`valueBoolean`、`valueMap`）——對 LLM 友善，不需要推斷型別。
    - 任何資料轉換（例如日期格式化）都必須在送出訊息前由伺服器完成。

=== "v0.9 — `updateDataModel`"

    ### 架構

    ```typescript
    {
      version: "v0.9";
      updateDataModel: {
        surfaceId: string;      // Required: Target surface
        path?: string;          // Optional: JSON Pointer path (defaults to "/")
        value?: any;            // Optional: Value to set (omit to delete)
      }
    }
    ```

    ### 屬性

    | 屬性        | 類型   | 必填     | 說明                                                   |
    | ----------- | ------ | -------- | ----------------------------------------------------- |
    | `surfaceId` | string | ✅        | 要更新的 surface ID。                                 |
    | `path`      | string | ❌        | JSON Pointer 路徑（例如 `/user/email`）。若省略，預設為 `/`（root）。 |
    | `value`     | any    | ❌        | 要設定的值。若省略，則會刪除 `path` 對應的鍵。       |

    ### 範例

    **Initialize entire model:**

    ```json
    {
      "version": "v0.9",
      "updateDataModel": {
        "surfaceId": "main",
        "path": "/",
        "value": {
          "user": {
            "name": "Alice",
            "email": "alice@example.com"
          },
          "items": []
        }
      }
    }
    ```

    **Update nested property:**

    ```json
    {
      "version": "v0.9",
      "updateDataModel": {
        "surfaceId": "main",
        "path": "/user/email",
        "value": "alice@newdomain.com"
      }
    }
    ```

    ### 使用注意事項

    - v0.9 使用標準 JSON Pointer 路徑與純 JSON 值，不再有型別包裝。
    - 若省略 `path`，預設為 `"/"`（root）。
    - `value` 可以是任何 JSON 型別（字串、數字、布林、物件、陣列、null）。若省略，則刪除。
    - 比 v0.8 的 `contents` adjacency list 更簡潔，也更接近標準 JSON Patch 語意。
    - 參照 `{"path": "/user/email"}` 的元件，會在該路徑變更時自動更新。

---

## deleteSurface

移除一個 surface 及其所有元件與資料。

=== "v0.8 — `deleteSurface`"

    ### 架構

    ```typescript
    {
      deleteSurface: {
        surfaceId: string;        // Required: Surface to delete
      }
    }
    ```

    ### 範例

    ```json
    {
      "deleteSurface": {
        "surfaceId": "modal"
      }
    }
    ```

=== "v0.9 — `deleteSurface`"

    ### 架構

    ```typescript
    {
      version: "v0.9";
      deleteSurface: {
        surfaceId: string;        // Required: Surface to delete
      }
    }
    ```

    ### 範例

    ```json
    {
      "version": "v0.9",
      "deleteSurface": {
        "surfaceId": "modal"
      }
    }
    ```

### 屬性

| 屬性 | 類型 | 必填 | 說明 |
| ----------- | ------ | -------- | --------------------------- |
| `surfaceId` | string | ✅        | 要刪除的 surface ID。       |

### 使用注意事項

- 會移除與該 surface 相關的所有元件
- 會清除該 surface 的資料模型
- 客戶端應從 UI 中移除該 surface
- 刪除不存在的 surface 是安全的（no-op）
- 可在關閉 modal、dialog 或離開頁面時使用
- 兩個版本的結構相同（v0.9 只多了 `version` 欄位）

---

## 訊息順序

### 要求

1. `beginRendering` 必須在該 surface 的初始 `surfaceUpdate` 訊息之後送出。
2. `surfaceUpdate` 可以出現在 `dataModelUpdate` 之前或之後。
3. 不同 surface 的訊息彼此獨立。
4. 多則訊息可以逐步更新同一個 surface。

### 建議順序

=== "v0.8"

    ```jsonl
    { "surfaceUpdate":    { "surfaceId": "main", "components": [...] } }
    { "dataModelUpdate":  { "surfaceId": "main", "contents": {...} } }
    { "beginRendering":   { "surfaceId": "main", "root": "root-id" } }
    ```

=== "v0.9"

    ```jsonl
    { "version": "v0.9", "createSurface":    { "surfaceId": "main", "catalogId": "..." } }
    { "version": "v0.9", "updateComponents": { "surfaceId": "main", "components": [...] } }
    { "version": "v0.9", "updateDataModel":  { "surfaceId": "main", "path": "/", "value": {...} } }
    ```

### 漸進式建構

=== "v0.8"

    ```jsonl
    { "surfaceUpdate":   { "surfaceId": "main", "components": [...] } }  // Header
    { "surfaceUpdate":   { "surfaceId": "main", "components": [...] } }  // Body
    { "beginRendering":  { "surfaceId": "main", "root": "root-id" } }   // Render
    { "surfaceUpdate":   { "surfaceId": "main", "components": [...] } }  // Footer
    { "dataModelUpdate": { "surfaceId": "main", "contents": {...} } }    // Data
    ```

=== "v0.9"

    ```jsonl
    { "version": "v0.9", "createSurface":    { "surfaceId": "main", "catalogId": "..." } }
    { "version": "v0.9", "updateComponents": { "surfaceId": "main", "components": [...] } }  // Header
    { "version": "v0.9", "updateComponents": { "surfaceId": "main", "components": [...] } }  // Body + Footer
    { "version": "v0.9", "updateDataModel":  { "surfaceId": "main", "path": "/", "value": {...} } }
    ```

## 驗證

=== "v0.8"

    依下列內容驗證：

    - **[server_to_client.json](../../../specification/v0_8/json/server_to_client.json)**: 訊息信封架構
    - **[standard_catalog_definition.json](../../../specification/v0_8/json/standard_catalog_definition.json)**: 元件架構

=== "v0.9"

    依下列內容驗證：

    - **[server_to_client.json](../../../specification/v0_9/json/server_to_client.json)**: 訊息信封架構
    - **[catalogs/basic/catalog.json](../../../specification/v0_9/catalogs/basic/catalog.json)**: 元件架構

## 延伸閱讀

- **[元件畫廊](components.md)**：所有可用的元件類型
- **[資料繫結指南](../concepts/data-binding.md)**：資料繫結如何運作
- **[Agent 開發指南](../guides/agent-development.md)**：生成有效訊息
