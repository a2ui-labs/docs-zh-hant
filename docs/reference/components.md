# 元件畫廊

本頁展示所有 A2UI 元件，並附上範例與使用模式。

!!! abstract "Schema 檔案"

    === "v0.8"

        [:material-code-json: 標準 Catalog 定義（JSON Schema）](https://a2ui.org/specification/v0_8/standard_catalog_definition.json)

    === "v0.9"

        [:material-code-json: 基本 Catalog 定義（JSON Schema）](https://a2ui.org/specification/v0_9/basic_catalog.json)

---

## 佈局元件

### Row

水平佈局容器。子元件由左至右排列。

=== "v0.8"

    **屬性：** `children`（`explicitList` 或 `template`）、`distribution`、`alignment`

    ```json
    {
      "id": "toolbar",
      "component": {
        "Row": {
          "children": { "explicitList": ["btn1", "btn2", "btn3"] },
          "distribution": "spaceBetween",
          "alignment": "center"
        }
      }
    }
    ```

=== "v0.9"

    **屬性：** `children`（陣列或 template）、`justify`、`align`

    ```json
    {
      "id": "toolbar",
      "component": "Row",
      "children": ["btn1", "btn2", "btn3"],
      "justify": "spaceBetween",
      "align": "center"
    }
    ```

### Column

垂直佈局容器。子元件由上至下排列。

=== "v0.8"

    **屬性：** `children`（`explicitList` 或 `template`）、`distribution`、`alignment`

    ```json
    {
      "id": "content",
      "component": {
        "Column": {
          "children": { "explicitList": ["header", "body", "footer"] },
          "distribution": "start",
          "alignment": "stretch"
        }
      }
    }
    ```

=== "v0.9"

    **屬性：** `children`（陣列或 template）、`justify`、`align`

    ```json
    {
      "id": "content",
      "component": "Column",
      "children": ["header", "body", "footer"],
      "justify": "start",
      "align": "stretch"
    }
    ```

### List

可捲動的項目清單。支援靜態子元件與動態模板。

=== "v0.8"

    **屬性：** `children`（`explicitList` 或 `template`）、`direction`、`alignment`

    ```json
    {
      "id": "message-list",
      "component": {
        "List": {
          "children": {
            "template": {
              "dataBinding": "/messages",
              "componentId": "message-item"
            }
          },
          "direction": "vertical"
        }
      }
    }
    ```

=== "v0.9"

    **屬性：** `children`（陣列或 template）、`direction`、`align`

    ```json
    {
      "id": "message-list",
      "component": "List",
      "children": {
        "componentId": "message-item",
        "path": "/messages"
      },
      "direction": "vertical"
    }
    ```

---

## 顯示元件

### Text

顯示文字內容，並提供樣式提示。

=== "v0.8"

    **屬性：** `text`（BoundValue）、`usageHint`

    `usageHint` 值：`h1`、`h2`、`h3`、`h4`、`h5`、`caption`、`body`

    ```json
    {
      "id": "title",
      "component": {
        "Text": {
          "text": { "literalString": "Welcome to A2UI" },
          "usageHint": "h1"
        }
      }
    }
    ```

=== "v0.9"

    **屬性：** `text`（字串或 DataBinding）、`variant`

    `variant` 值：`h1`、`h2`、`h3`、`h4`、`h5`、`caption`、`body`

    ```json
    {
      "id": "title",
      "component": "Text",
      "text": "Welcome to A2UI",
      "variant": "h1"
    }
    ```

### Image

顯示來自 URL 的圖片。

=== "v0.8"

    **屬性：** `url`（BoundValue）、`fit`、`usageHint`

    ```json
    {
      "id": "hero",
      "component": {
        "Image": {
          "url": { "literalString": "https://example.com/hero.png" },
          "fit": "cover",
          "usageHint": "hero"
        }
      }
    }
    ```

=== "v0.9"

    **屬性：** `url`（字串或 DataBinding）、`fit`、`variant`

    ```json
    {
      "id": "hero",
      "component": "Image",
      "url": "https://example.com/hero.png",
      "fit": "cover",
      "variant": "hero"
    }
    ```

### Icon

顯示 catalog 中定義的標準圖示。

=== "v0.8"

    **屬性：** `name`（BoundValue）

    ```json
    {
      "id": "check-icon",
      "component": {
        "Icon": {
          "name": { "literalString": "check" }
        }
      }
    }
    ```

=== "v0.9"

    **屬性：** `name`（字串或 DataBinding）

    ```json
    {
      "id": "check-icon",
      "component": "Icon",
      "name": "check"
    }
    ```

### Divider

視覺分隔線。

=== "v0.8"

    **屬性：** `axis`

    ```json
    {
      "id": "separator",
      "component": {
        "Divider": {
          "axis": "horizontal"
        }
      }
    }
    ```

=== "v0.9"

    **屬性：** `axis`

    ```json
    {
      "id": "separator",
      "component": "Divider",
      "axis": "horizontal"
    }
    ```

---

## 互動元件

### Button

可點擊的按鈕，用來觸發動作。

=== "v0.8"

    **屬性：** `child`（元件 ID）、`primary`（boolean）、`action`

    ```json
    {
      "id": "submit-btn",
      "component": {
        "Button": {
          "child": "submit-text",
          "primary": true,
          "action": {
            "name": "submit_form"
          }
        }
      }
    }
    ```

=== "v0.9"

    **屬性：** `child`（元件 ID）、`variant`、`action`

    ```json
    {
      "id": "submit-btn",
      "component": "Button",
      "child": "submit-text",
      "variant": "primary",
      "action": {
        "event": {
          "name": "submit_form"
        }
      }
    }
    ```

### TextField

文字輸入欄位，可選擇搭配驗證。

=== "v0.8"

    **屬性：** `label`（BoundValue）、`text`（BoundValue）、`textFieldType`、`validationRegexp`

    `textFieldType` 值：`shortText`、`longText`、`number`、`obscured`、`date`

    ```json
    {
      "id": "email-input",
      "component": {
        "TextField": {
          "label": { "literalString": "Email Address" },
          "text": { "path": "/user/email" },
          "textFieldType": "shortText"
        }
      }
    }
    ```

=== "v0.9"

    **屬性：** `label`（字串）、`value`（字串或 DataBinding）、`textFieldType`、`validationRegexp`

    `textFieldType` 值：`shortText`、`longText`、`number`、`obscured`、`date`

    ```json
    {
      "id": "email-input",
      "component": "TextField",
      "label": "Email Address",
      "value": { "path": "/user/email" },
      "textFieldType": "shortText"
    }
    ```

### CheckBox

布林切換元件。

=== "v0.8"

    **屬性：** `label`（BoundValue）、`value`（BoundValue，boolean）

    ```json
    {
      "id": "terms-checkbox",
      "component": {
        "CheckBox": {
          "label": { "literalString": "I agree to the terms" },
          "value": { "path": "/form/agreedToTerms" }
        }
      }
    }
    ```

=== "v0.9"

    **屬性：** `label`（字串）、`value`（DataBinding，boolean）

    ```json
    {
      "id": "terms-checkbox",
      "component": "CheckBox",
      "label": "I agree to the terms",
      "value": { "path": "/form/agreedToTerms" }
    }
    ```

### Slider

數值範圍輸入元件。

=== "v0.8"

    **屬性：** `value`（BoundValue）、`minValue`、`maxValue`

    ```json
    {
      "id": "volume",
      "component": {
        "Slider": {
          "value": { "path": "/settings/volume" },
          "minValue": 0,
          "maxValue": 100
        }
      }
    }
    ```

=== "v0.9"

    **屬性：** `value`（DataBinding）、`minValue`、`maxValue`

    ```json
    {
      "id": "volume",
      "component": "Slider",
      "value": { "path": "/settings/volume" },
      "minValue": 0,
      "maxValue": 100
    }
    ```

### DateTimeInput

日期和/或時間選擇器。

=== "v0.8"

    **屬性：** `value`（BoundValue）、`enableDate`、`enableTime`

    ```json
    {
      "id": "date-picker",
      "component": {
        "DateTimeInput": {
          "value": { "path": "/booking/date" },
          "enableDate": true,
          "enableTime": false
        }
      }
    }
    ```

=== "v0.9"

    **屬性：** `value`（DataBinding）、`enableDate`、`enableTime`

    ```json
    {
      "id": "date-picker",
      "component": "DateTimeInput",
      "value": { "path": "/booking/date" },
      "enableDate": true,
      "enableTime": false
    }
    ```

### MultipleChoice (v0.8) / ChoicePicker (v0.9)

從清單中選擇一個或多個選項。

=== "v0.8"

    **屬性：** `options`（陣列）、`selections`（BoundValue）、`maxAllowedSelections`

    ```json
    {
      "id": "country-select",
      "component": {
        "MultipleChoice": {
          "options": [
            { "label": { "literalString": "USA" }, "value": "us" },
            { "label": { "literalString": "Canada" }, "value": "ca" }
          ],
          "selections": { "path": "/form/country" },
          "maxAllowedSelections": 1
        }
      }
    }
    ```

=== "v0.9"

    **屬性：** `options`（陣列）、`selections`（DataBinding）、`maxAllowedSelections`

    ```json
    {
      "id": "country-select",
      "component": "ChoicePicker",
      "options": [
        { "label": "USA", "value": "us" },
        { "label": "Canada", "value": "ca" }
      ],
      "selections": { "path": "/form/country" },
      "maxAllowedSelections": 1
    }
    ```

---

## 容器元件

### Card

具有陰影/邊框與內距的容器。

=== "v0.8"

    **屬性：** `child`（元件 ID）

    ```json
    {
      "id": "info-card",
      "component": {
        "Card": {
          "child": "card-content"
        }
      }
    }
    ```

=== "v0.9"

    **屬性：** `child`（元件 ID）

    ```json
    {
      "id": "info-card",
      "component": "Card",
      "child": "card-content"
    }
    ```

### Modal

由入口元件觸發的覆蓋式對話框。

=== "v0.8"

    **屬性：** `entryPointChild`（元件 ID）、`contentChild`（元件 ID）

    ```json
    {
      "id": "confirmation-modal",
      "component": {
        "Modal": {
          "entryPointChild": "open-modal-btn",
          "contentChild": "modal-content"
        }
      }
    }
    ```

=== "v0.9"

    **屬性：** `entryPointChild`（元件 ID）、`contentChild`（元件 ID）

    ```json
    {
      "id": "confirmation-modal",
      "component": "Modal",
      "entryPointChild": "open-modal-btn",
      "contentChild": "modal-content"
    }
    ```

### Tabs

以分頁面板方式組織內容的切換式介面。

=== "v0.8"

    **屬性：** `tabItems`（`{ title, child }` 陣列）

    ```json
    {
      "id": "settings-tabs",
      "component": {
        "Tabs": {
          "tabItems": [
            { "title": { "literalString": "General" }, "child": "general-tab" },
            { "title": { "literalString": "Privacy" }, "child": "privacy-tab" }
          ]
        }
      }
    }
    ```

=== "v0.9"

    **屬性：** `tabItems`（`{ title, child }` 陣列）

    ```json
    {
      "id": "settings-tabs",
      "component": "Tabs",
      "tabItems": [
        { "title": "General", "child": "general-tab" },
        { "title": "Privacy", "child": "privacy-tab" }
      ]
    }
    ```

---

## 共通屬性

所有元件都共用：

- `id`（必填）：在 surface 內的唯一識別符
- `accessibility`：無障礙屬性（label、role）
- `weight`：位於 Row 或 Column 中時的 flex-grow 值

## 版本差異摘要

元件名稱與屬性在各版本之間大致相同。結構上的差異如下：

| 面向 | v0.8 | v0.9 |
|--------|------|------|
| 元件包裝 | `"component": { "Text": { ... } }` | `"component": "Text", ...props` |
| 字串值 | `{ "literalString": "Hello" }` | `"Hello"` |
| 子元件 | `{ "explicitList": ["a", "b"] }` | `["a", "b"]` |
| Data binding | `{ "path": "/data" }` | `{ "path": "/data" }` (same) |
| Text/Image 樣式 | `usageHint` | `variant` |
| Button 樣式 | `primary: true` | `variant: "primary"` |
| 動作格式 | `{ "name": "..." }` | `{ "event": { "name": "..." } }` |
| 選擇元件 | `MultipleChoice` | `ChoicePicker` |
| 版面對齊 | `distribution`、`alignment` | `justify`、`align` |
| TextField 值 | `text` | `value` |

## 即時範例

要查看所有元件的實際效果：

```bash
cd samples/client/angular
npm start -- gallery
```

## 延伸閱讀

!!! abstract "Schema 檔案"

    === "v0.8"

        [:material-code-json: 標準 Catalog 定義（JSON Schema）](https://a2ui.org/specification/v0_8/standard_catalog_definition.json)

    === "v0.9"

        [:material-code-json: 基本 Catalog 定義（JSON Schema）](https://a2ui.org/specification/v0_9/basic_catalog.json)

- **[自訂元件指南](../guides/custom-components.md)**：建立自己的元件
- **[主題指南](../guides/theming.md)**：讓元件風格符合你的品牌
