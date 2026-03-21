# 元件與結構

A2UI 針對元件階層使用 **鄰接清單模型**。不採用巢狀 JSON 樹，而是以帶有 ID 參照的扁平清單來表示元件。

## 為什麼要用扁平清單？

**傳統的巢狀做法：**

- LLM 必須一次產生完全正確的巢狀結構
- 很難更新深度巢狀的元件
- 不容易逐步串流

**A2UI 鄰接清單：**

- ✅ 扁平結構，LLM 更容易產生
- ✅ 可以逐步送出元件
- ✅ 可以透過 ID 更新任何元件
- ✅ 清楚分離結構與資料

## 鄰接清單模型

=== "v0.8"

    ```json
    {
      "surfaceUpdate": {
        "components": [
          {
            "id": "root",
            "component": {
              "Column": {
                "children": { "explicitList": ["greeting", "buttons"] }
              }
            }
          },
          {
            "id": "greeting",
            "component": {
              "Text": { "text": { "literalString": "Hello" } }
            }
          },
          {
            "id": "buttons",
            "component": {
              "Row": {
                "children": { "explicitList": ["cancel-btn", "ok-btn"] }
              }
            }
          },
          {
            "id": "cancel-btn",
            "component": {
              "Button": {
                "child": "cancel-text",
                "action": { "name": "cancel" }
              }
            }
          },
          {
            "id": "cancel-text",
            "component": {
              "Text": { "text": { "literalString": "Cancel" } }
            }
          },
          {
            "id": "ok-btn",
            "component": {
              "Button": {
                "child": "ok-text",
                "action": { "name": "ok" }
              }
            }
          },
          {
            "id": "ok-text",
            "component": {
              "Text": { "text": { "literalString": "OK" } }
            }
          }
        ]
      }
    }
    ```

=== "v0.9"

    ```json
    {
      "version": "v0.9",
      "updateComponents": {
        "surfaceId": "main",
        "components": [
          {
            "id": "root",
            "component": "Column",
            "children": ["greeting", "buttons"]
          },
          {
            "id": "greeting",
            "component": "Text",
            "text": "Hello"
          },
          {
            "id": "buttons",
            "component": "Row",
            "children": ["cancel-btn", "ok-btn"]
          },
          {
            "id": "cancel-btn",
            "component": "Button",
            "child": "cancel-text",
            "action": { "event": { "name": "cancel" } }
          },
          {
            "id": "cancel-text",
            "component": "Text",
            "text": "Cancel"
          },
          {
            "id": "ok-btn",
            "component": "Button",
            "child": "ok-text",
            "action": { "event": { "name": "ok" } }
          },
          {
            "id": "ok-text",
            "component": "Text",
            "text": "OK"
          }
        ]
      }
    }
    ```

    v0.9 採用更扁平的元件格式：`"component": "Text"` 取代巢狀的 `{"Text": {...}}`，而 children 也改成單純陣列，不再使用 `{"explicitList": [...]}`。

元件透過 ID 參照子元件，而不是透過巢狀結構。

## 元件基礎

每個元件都包含：

1. **ID**：唯一識別碼（`"welcome"`）
2. **Type**：元件類型（`Text`、`Button`、`Card`）
3. **Properties**：該類型專屬的設定

=== "v0.8"

    ```json
    {
      "id": "welcome",
      "component": {
        "Text": {
          "text": { "literalString": "Hello" },
          "usageHint": "h1"
        }
      }
    }
    ```

=== "v0.9"

    ```json
    {
      "id": "welcome",
      "component": "Text",
      "text": "Hello",
      "variant": "h1"
    }
    ```

## 標準目錄

A2UI 定義了一組依用途組織的標準元件目錄：

- **Layout**：Row、Column、List - 用來排列其他元件
- **Display**：Text、Image、Icon、Video、Divider - 用來顯示資訊
- **Interactive**：Button、TextField、CheckBox、DateTimeInput、Slider - 用來收集使用者輸入
- **Container**：Card、Tabs、Modal - 用來分組與組織內容

完整的元件圖庫與範例請參閱 [元件參考](../reference/components.md)。

## 靜態與動態子元件

**靜態（`explicitList`）** - 固定的子元件 ID 清單：
```json
{
  "children": {
    "explicitList": ["back-btn", "title", "menu-btn"]
  }
}
```

**動態（`template`）** - 根據資料陣列產生子元件：
```json
{
  "children": {
    "template": {
      "dataBinding": "/items",
      "componentId": "item-template"
    }
  }
}
```

對 `/items` 中的每個項目，渲染 `item-template`。詳情請參閱 [資料繫結](data-binding.md)。

## 以值填充

元件的值有兩種取得方式：

- **Literal** - 固定值：`{"text": {"literalString": "Welcome"}}`
- **Data-bound** - 來自資料模型：`{"text": {"path": "/user/name"}}`

LLM 可以產生帶有字面值的元件，或將元件繫結到資料路徑，以顯示動態內容。

## 組合 Surfaces

元件可以組合成 **surfaces**（小工具）：

=== "v0.8"

    1. LLM 透過 `surfaceUpdate` 產生元件定義
    2. LLM 透過 `dataModelUpdate` 填入資料
    3. LLM 透過 `beginRendering` 發出開始渲染訊號
    4. Client 將所有元件渲染為原生 widget

=== "v0.9"

    1. LLM 透過 `createSurface` 建立 surface（並指定 catalog）
    2. LLM 透過 `updateComponents` 產生元件定義
    3. LLM 透過 `updateDataModel` 填入資料
    4. Client 將所有元件渲染為原生 widget

surface 是一個完整且一致的 UI（表單、儀表板、聊天介面等）。

## 增量更新

- **新增** - 傳送新的元件定義與新的 ID
- **更新** - 傳送具有既有 ID 與新屬性的元件定義
- **移除** - 更新父層的 `children` 清單，排除已移除的 ID

扁平結構讓所有更新都變成簡單的 ID 型操作。

## 自訂元件

除了標準目錄之外，client 也可以為特定領域需求定義自訂元件：

- **如何做**：在 renderer 中註冊自訂元件型別
- **內容**：圖表、地圖、自訂視覺化、專用 widget
- **安全性**：自訂元件仍然屬於 client 的受信任目錄

自訂元件會由 client 的 renderer 向 LLM **公開宣告**。LLM 之後就能在標準目錄之外一併使用它們。

實作細節請參閱 [自訂元件指南](../guides/custom-components.md)。

## 最佳實務

1. **具描述性的 ID**：使用 `"user-profile-card"`，不要用 `"c1"`
2. **淺層階層**：避免過深的巢狀結構
3. **分離結構與內容**：使用資料繫結，不要直接用字面值
4. **用模板重用**：一份模板，透過動態子元件產生多個實例
