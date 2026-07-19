# 資料繫結

資料繫結會使用 JSON Pointer 路徑 ([RFC 6901](https://tools.ietf.org/html/rfc6901)) 將 UI 元件連接到應用程式狀態。這也是 A2UI 能夠有效率地為大型資料陣列定義版面配置，或在不重新從頭產生內容的情況下顯示更新後內容的原因。

## 結構與狀態

A2UI 會將下列兩者分開：

1. **UI 結構**（Components）：介面看起來是什麼樣子
2. **應用程式狀態**（Data Model）：介面顯示哪些資料

這樣可以支援：回應式更新、以資料驅動的 UI、可重用模板，以及雙向繫結。

## 資料模型

每個 surface 都有一個保存狀態的 JSON 物件：

```json
{
  "user": {"name": "Alice", "email": "alice@example.com"},
  "cart": {
    "items": [{"name": "Widget", "price": 9.99, "quantity": 2}],
    "total": 19.98
  }
}
```

## JSON Pointer 路徑

**語法：**

- `/user/name` - 物件屬性
- `/cart/items/0` - 陣列索引（從 0 開始）
- `/cart/items/0/price` - 巢狀路徑

**範例：**

```json
{"user": {"name": "Alice"}, "items": ["Apple", "Banana"]}
```

- `/user/name` → `"Alice"`
- `/items/0` → `"Apple"`

## 字面值與路徑值

=== "v0.8"

    **Literal（固定）：**
    ```json
    {
      "id": "title",
      "component": {
        "Text": {
          "text": { "literalString": "Welcome" }
        }
      }
    }
    ```

    **Data-bound（回應式）：**
    ```json
    {
      "id": "username",
      "component": {
        "Text": {
          "text": { "path": "/user/name" }
        }
      }
    }
    ```

=== "v0.9 and later"

    **Literal（固定）：**
    ```json
    {
      "id": "title",
      "component": "Text",
      "text": "Welcome"
    }
    ```

    **Data-bound（回應式）：**
    ```json
    {
      "id": "username",
      "component": "Text",
      "text": { "path": "/user/name" }
    }
    ```

當 `/user/name` 從 "Alice" 變成 "Bob" 時，文字會 **自動更新** 為 "Bob"。

## 回應式更新

繫結到資料路徑的元件會在資料變更時自動更新：

=== "v0.8"

    ```json
    {
      "id": "status",
      "component": {
        "Text": {
          "text": {"path": "/order/status"}
        }
      }
    }
    ```

=== "v0.9 and later"

    ```json
    {
      "id": "status",
      "component": "Text",
      "text": {"path": "/order/status"}
    }
    ```

- **初始值：** `/order/status` = "Processing..." → 顯示 "Processing..."
- **更新：** 送出一個 data model 更新，將 `status` 設為 `"Shipped"` → 顯示 "Shipped"

不需要更新元件本身，只要更新資料即可。

## 動態清單

使用 template 來渲染陣列：

=== "v0.8"

    ```json
    {
      "id": "product-list",
      "component": {
        "Column": {
          "children": {
            "template": {
              "dataBinding": "/products",
              "componentId": "product-card"
            }
          }
        }
      }
    }
    ```

=== "v0.9 and later"

    ```json
    {
      "id": "product-list",
      "component": "Column",
      "children": {
        "path": "/products",
        "componentId": "product-card"
      }
    }
    ```

**資料：**
```json
{
  "products": [
    { "name": "Widget", "price": 9.99 },
    { "name": "Gadget", "price": 19.99 }
  ]
}
```

**結果：** 會渲染兩張卡片，每個商品各一張。

### 作用域路徑

在 template 內，路徑的作用域會限制在陣列項目本身：

=== "v0.8"

    ```json
    {
      "id": "product-name",
      "component": {
        "Text": {
          "text": {"path": "/name"}
        }
      }
    }
    ```

    - 對 `/products/0` 而言，`/name` 會解析為 `/products/0/name` → "Widget"
    - 對 `/products/1` 而言，`/name` 會解析為 `/products/1/name` → "Gadget"

=== "v0.9 and later"

    ```json
    {
      "id": "product-name",
      "component": "Text",
      "text": {"path": "name"}
    }
    ```

    - 對 `/products/0` 而言，`name` 會解析為 `/products/0/name` → "Widget"
    - 對 `/products/1` 而言，`name` 會解析為 `/products/1/name` → "Gadget"

新增或移除項目時，系統會自動更新已渲染的元件。

## 輸入繫結

互動式元件會以雙向方式更新資料模型：

=== "v0.8"

    | Component | Example | User Action | Data Update |
    |-----------|---------|-------------|-------------|
    | **TextField** | `{"text": {"path": "/form/name"}}` | 輸入 "Alice" | `/form/name` = "Alice" |
    | **CheckBox** | `{"value": {"path": "/form/agreed"}}` | 勾選方塊 | `/form/agreed` = true |
    | **MultipleChoice** | `{"selections": {"path": "/form/country"}}` | 選取 "Canada" | `/form/country` = ["ca"] |

=== "v0.9 and later"

    | Component | Example | User Action | Data Update |
    |-----------|---------|-------------|-------------|
    | **TextField** | `{"value": {"path": "/form/name"}}` | 輸入 "Alice" | `/form/name` = "Alice" |
    | **CheckBox** | `{"value": {"path": "/form/agreed"}}` | 勾選方塊 | `/form/agreed` = true |
    | **ChoicePicker** | `{"value": {"path": "/form/country"}}` | 選取 "Canada" | `/form/country` = ["ca"] |

## 最佳實務

**1. 使用細粒度更新** - 只更新已變更的路徑：

=== "v0.8"

    ```json
    {
      "dataModelUpdate": {
        "path": "/user",
        "contents": [{"key": "name", "valueString": "Alice"}]
      }
    }
    ```

=== "v0.9 and later"

    ```json
    {
      "version": "v0.9.1",
      "updateDataModel": {
        "surfaceId": "user_profile",
        "path": "/user/name",
        "value": "Alice"
      }
    }
    ```

**2. 依領域組織** - 將相關資料分組：
```json
{"user": {...}, "cart": {...}, "ui": {...}}
```

**3. 預先計算顯示值** - Agent 在送出前先格式化資料（貨幣、日期）：
```json
{"price": "$19.99"}  // Not: {"price": 19.99}
```
