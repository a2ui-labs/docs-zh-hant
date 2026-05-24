# A2UI 目錄

## 概覽

本指南定義 A2UI 目錄架構，並提供實作路線圖。它會說明目錄 schema 的結構、如何使用預先建好的「基礎目錄」與如何定義自己的應用專屬目錄，以及目錄協商、版本控制與執行期驗證的技術流程。

## 目錄定義

目錄是一個 [JSON Schema 檔案](../../specification/v0_9/json/client_capabilities.json#L62C5-L95C6)，用來列出智慧體可用來定義 A2UI surface 的元件、函式與主題。智慧體送出的所有 A2UI JSON 都會依照選定的目錄進行驗證。

下方是 [目錄 JSON Schema](../../specification/v0_9/json/client_capabilities.json#L62C5-L95C6)：

```json
{
  "Catalog": {
    "type": "object",
    "description": "A collection of component and function definitions.",
    "properties": {
      "catalogId": {
        "type": "string",
        "description": "Unique identifier for this catalog."
      },
      "components": {
        "type": "object",
        "description": "Definitions for UI components supported by this catalog.",
        "additionalProperties": {
          "$ref": "https://json-schema.org/draft/2020-12/schema"
        }
      },
      "functions": {
        "type": "array",
        "description": "Definitions for functions supported by this catalog.",
        "items": {
          "$ref": "#/$defs/FunctionDefinition"
        }
      },
      "theme": {
        "title": "A2UI Theme",
        "description": "A schema that defines a catalog of A2UI theme properties.",
        "type": "object",
        "additionalProperties": {
          "$ref": "https://json-schema.org/draft/2020-12/schema"
        }
      }
    },
    "required": [
      "catalogId"
    ],
    "additionalProperties": false
  }
}
```

## 目錄策略

每個 A2UI surface 都由一個目錄驅動。目錄本質上就是一個 JSON Schema 檔案，它會告訴智慧體可使用哪些元件、函式與主題。

不論你是在做簡單原型還是複雜的生產級應用，需求都一樣：你必須提供一份目錄定義，讓智慧體用它來表達 UI。

### 基礎目錄

為了幫助開發者快速上手，A2UI 團隊維護了一份 [基礎目錄](../../specification/v0_9/catalogs/basic/catalog.json)。

這是一份預先定義好的目錄檔，包含一組通用元件（Button、Input、Card）與函式。它不是某種特別的目錄「類型」；它只是我們已經寫好、且已有開源渲染器支援的目錄版本。

基礎目錄可以讓你在不從零撰寫 schema 的情況下，先把應用跑起來，或驗證 A2UI 概念。它刻意保持精簡，這樣不同渲染器才更容易實作。

由於 A2UI 的設計目標是讓 LLM 在設計時或執行時都能生成 UI，我們不認為可攜性需要在多個客戶端之間使用統一的目錄；LLM 可以針對每個前端單獨理解各自的目錄。

[查看 A2UI v0.9 基礎目錄](../../specification/v0_9/catalogs/basic/catalog.json)

### 定義自己的目錄

雖然基礎目錄很適合起步，但大多數生產應用都會定義自己的目錄，以反映其專屬設計系統。

透過定義自己的目錄，你可以限制智慧體只能使用應用中實際存在的元件與視覺語言，而不是通用輸入框或按鈕。這份目錄可以完全從零建立，也可以從基礎目錄匯入定義以節省時間，例如在定義自己獨特的 Card 元件時，直接沿用基礎文字定義。

為了簡化實作，我們建議直接讓目錄反映客戶端的設計系統，而不是透過轉接器把基礎目錄映射過去。由於 A2UI 是為 GenUI 設計的，我們預期 LLM 能理解不同客戶端對應不同目錄。

[查看 Rizzcharts 目錄範例](../../samples/agent/adk/rizzcharts/catalog_schemas/0.9/rizzcharts_catalog_definition.json)

### 建議

| 使用情境 | 建議 | 成本 |
| :--- | :--- | :--- |
| 將 A2UI 加入成熟前端 | 定義一份與現有設計系統相符的目錄。 | 中等 |
| 將 A2UI 加入新專案 / 全新應用 | 先從基礎目錄開始，之後隨著應用演進再擴充成自己的目錄。 | 低（假設渲染器已存在） |

## 建立目錄

目錄是一份符合 [目錄 schema](../../specification/v0_9/json/client_capabilities.json#L62C5-L95C6) 的 JSON Schema 檔案，用來定義智慧體在建立 surface 時可以使用的元件、主題與函式。

### 範例：最小目錄

下面是一個定義單一元件的簡單目錄。

```json
{
  "$id": "https://github.com/.../hello_world/v1/catalog.json",
  "components": {
    "HelloWorldBanner": {
      "type": "object",
      "description": "A simple banner greeting.",
      "properties": {
        "message": {
          "type": "string",
          "description": "The banner text."
        },
        "backgroundColor": {
          "type": "string",
          "default": "#f0f0f0"
        }
      },
      "required": [
        "message"
      ]
    }
  }
}
```

當智慧體使用這份目錄時，它會產生嚴格符合該結構的 payload：

```json
[
  {
    "version": "v0.9",
    "createSurface": {
      "surfaceId": "hello-world-surface",
      "catalogId": "https://github.com/.../hello_world/v1/catalog.json"
    }
  },
  {
    "version": "v0.9",
    "updateComponents": {
      "surfaceId": "hello-world-surface",
      "components": [
        {
          "id": "root",
          "component": "HelloWorldBanner",
          "message": "Hello, world! Welcome to your first catalog.",
          "backgroundColor": "#4CAF50"
        }
      ]
    }
  }
]
```

### 獨立目錄

A2UI 目錄必須是獨立的，不可引用外部檔案，這樣才能簡化 LLM 推理與相依管理。

雖然最終目錄必須是獨立的，但在本地開發時，你仍然可以用 JSON Schema `$ref` 指向外部文件來模組化撰寫目錄。發佈前請先執行 `tools/build_catalog/assemble_catalog.py`，把所有外部檔案參照打包成單一、獨立的 JSON Schema 檔案：

```bash
uv run tools/build_catalog/assemble_catalog.py [INPUTS ...] --output-name <OUTPUT_NAME> [--catalog-id <ID>] [--version <VERSION>] [--extend-basic-catalog] [--out-dir <DIR>] [--verbose]
```

其中：
- `inputs`：一個或多個 A2UI 元件目錄 JSON 的路徑或 URL。
- `--output-name`：必填，合併後目錄的名稱，例如 `my_merged_catalog`。若未提供副檔名，會自動補上 `.json`。
- `--catalog-id`：輸出用的自訂 `catalogId`。預設為 `urn:a2ui:catalog:<base_name>`。
- `--version`：官方目錄 fallback 所使用的 A2UI 規範版本，可選 `0.9` 或 `0.10`。預設為 `0.9`。
- `--extend-basic-catalog`：若提供此參數，會在根輸出中自動包含整份 `basic_catalog.json`，不論輸入目錄是否顯式引用它。
- `--out-dir`、`-o`：組裝後目錄的輸出目錄。預設為 `dist`。
- `--verbose`、`-v`：若提供此參數，會啟用詳細除錯日誌，方便排查問題。

### 組合與匯入

你不需要從零定義所有內容。你可以建立一份目錄，重用基礎目錄或其他目錄中的既有元件，並沿用既有的渲染邏輯。

#### 範例：擴充基礎目錄

這份目錄會匯入基礎目錄的所有元素，並新增一個 `SuggestionChips` 元件。

```json
{
  "$id": "https://github.com/.../hello_world_with_all_basic/v1/catalog.json",
  "components": {
    "allOf": [
      { "$ref": "basic_catalog_definition.json#/components" },
      {
        "SuggestionChips": {
          "type": "object",
          "description": "A list of suggested prompts",
          "properties": {
            "suggestions": {
              "type": "array",
              "description": "The suggested prompts."
            }
          },
          "required": [ "suggestions" ]
        }
      }
    ]
  }
}
```

**發佈前請務必執行 `tools/build_catalog/assemble_catalog.py`，先解析外部 `$ref`。**

#### 範例：挑選性匯入元件

這份目錄只從基礎目錄匯入 `Text`，用來建立一個簡單的 Popup surface。

```json
{
  "$id": "https://github.com/.../hello_world_with_some_basic/v1/catalog.json",
  "components": {
    "allOf": [
      { "$ref": "basic_catalog.json#/components/Text" },
      {
        "Popup": {
          "type": "object",
          "description": "A modal overlay that displays an icon and text.",
          "properties": {
            "text": { "$ref": "common_types.json#/$defs/ComponentId" }
          },
          "required": [ "text" ]
        }
      }
    ]
  }
}
```

**發佈前請務必執行 `tools/build_catalog/assemble_catalog.py`，先解析外部 `$ref`。**

### 實作 Renderer

客戶端渲染器會透過將 schema 定義映射到實際程式碼來實作該目錄。

以下是 hello world 目錄的 TypeScript 渲染器範例：

```typescript
import { Catalog, DEFAULT_CATALOG } from '@a2ui/angular';
import { inputBinding } from '@angular/core';

export const RIZZ_CHARTS_CATALOG = {
  ...DEFAULT_CATALOG, // 包含基礎目錄
  HelloWorldBanner: {
    type: () => import('./hello_world_banner').then((r) => r.HelloWorldBanner),
    bindings: ({ properties }) => [
      inputBinding('message', () => ('message' in properties && properties['message']) || undefined)
    ],
  },
} as Catalog;
```

以及 `hello_world_banner` 的實作：

```typescript
import { DynamicComponent } from '@a2ui/angular';
import { Component, Input } from '@angular/core';

@Component({
  selector: 'hello-world-banner',
  imports: [],
  template: `
    <div>
      <h2>Hello World Banner</h2>
      <p>{{ message }}</p>
    </div>
  `,
})
export class HelloWorldBanner extends DynamicComponent {
  @Input() message?: string;
}
```

你可以在 [Rizzcharts 示範](../../samples/client/angular/projects/rizzcharts/src/a2ui-catalog/catalog.ts) 中看到可運作的客戶端渲染器範例。

## A2UI 目錄協商

由於客戶端與智慧體可能支援多個目錄，它們必須透過目錄協商握手，先對齊要使用哪一份目錄。

### 步驟 1：智慧體宣告支援的目錄（可選）

智慧體可以選擇宣告自己支援哪些目錄（例如在 A2A Agent Card 中）。這只是資訊用途；它能幫助客戶端知道智慧體是否支援它的特定功能，但客戶端不一定要使用它。

以下是 A2A AgentCard 的範例，宣告該 agent 支援 basic 與 rizzcharts 目錄：

```json
{
  "name": "Ecommerce Dashboard Agent",
  "description": "This agent visualizes ecommerce data...",
  "capabilities": {
    "extensions": [
      {
        "uri": "https://a2ui.org/a2a-extension/a2ui/v0.8",
        "description": "Provides agent driven UI using the A2UI JSON format.",
        "params": {
          "supportedCatalogIds": [
            "https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json",
            "https://github.com/.../rizzcharts_catalog_definition.json"
          ]
        }
      }
    ]
  }
}
```

### 步驟 2：客戶端宣告支援的目錄（必填）

客戶端會在每則訊息的 metadata 中，依照偏好順序把 `supportedCatalogIds` 傳給智慧體。這會讓智慧體明確知道客戶端目前可以渲染哪些內容。

以下是 metadata 中包含 `supportedCatalogIds` 的 A2A 訊息範例：

```json
{
  "parts": [
    {
      "text": "What is the current status of my flight?"
    }
  ],
  "metadata": {
    "a2uiClientCapabilities": {
      "supportedCatalogIds": [
        "https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json",
        "https://github.com/.../rizzcharts_catalog_definition.json"
      ]
    }
  }
}
```

### 步驟 3：智慧體選擇

當智慧體建立新的 surface 時，它會從客戶端的 `supportedCatalogIds` 清單中選擇最合適的項目。這個選擇會在該 surface 的生命週期內固定下來。如果找不到相容的目錄，智慧體就不會送出 UI。

以下是智慧體定義 surface 所用 `catalogId` 的 A2UI 訊息範例：

```json
{
  "createSurface": {
    "surfaceId": "salesDashboard",
    "catalogId": "https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json"
  }
}
```

## 目錄命名與版本控制

A2UI 元件目錄需要版本控制，因為目錄定義通常是在編譯期建好的，所以智慧體生成的內容與客戶端可渲染內容之間只要有不一致，就可能影響 UI。

### `catalogId` 命名慣例

`catalogId` 是客戶端與智慧體進行協商時使用的唯一文字識別碼。

* **格式：** 雖然 `catalogId` 技術上是一個字串，但 A2UI 慣例是使用 **URI**（例如 `https://example.com/catalogs/mysurface/v1/catalog.json`）。
* **用途：** 我們使用 URI 是為了讓 ID 在全域上唯一，也方便人類開發者在瀏覽器中檢視。
* **不會於執行期抓取：** 這個 URI 並不代表智慧體或客戶端會在執行期下載目錄。**目錄定義必須在編譯 / 部署前就為智慧體與客戶端所知。** 這個 URI 只是一個穩定識別碼。

### 版本控制指引

為了在不中斷舊客戶端或智慧體的情況下持續演進，A2UI 會根據變更是否 **可安全忽略** 來分類目錄更新。

雖然標準 JSON parser 會忽略未知欄位，但在 Server-Driven UI 中，少掉一個元件可能會讓整個視圖樹消失。為了在安全與彈性之間取得平衡，更新被分成 **破壞性** 與 **非破壞性** 類別，並依賴 **優雅降級** 來吸收版本落差。

*   **破壞性變更（需要提升主版本）**
    任何會改變結構、使舊客戶端無法安全忽略的變更，都必須在 `catalogId` URI 中把 **主版本** 往上調（例如 `v1` 變成 `v2`）。
    *   **新增容器元件：** 例如新增 `Grid` 或 `Accordion`。如果舊客戶端忽略這個容器，它會把底下所有子元件一起丟掉，導致 UI 樹損壞。
    *   **移除容器元件：** 例如移除 `Grid` 或 `Accordion`。如果舊智慧體仍使用這個容器，客戶端會忽略它，然後把底下所有子元件一起丟掉，導致 UI 樹損壞。
    *   **變更欄位型別：** 例如把某個屬性從 `string` 改成 `object`。這會讓舊客戶端的 JSON Schema 驗證失敗。
    *   **新增必要屬性：** 若沒有預設值，舊智慧體不會知道要送出它。

*   **非破壞性變更（在主版本內可接受）**
    那些可以被安全忽略、或在不破壞版面與資料模型的前提下優雅退化的變更，則可以維持在同一版本。
    *   **新增葉節點元件（非容器）：** 例如新增 `Badge` 或 `Tooltip`。如果被忽略，版面仍可保持完整。
    *   **新增可選屬性：** 例如在 Card 上新增 `subtitle`。
    *   **移除屬性：** 若智慧體不再送出它，客戶端可以安全忽略。
    *   **新增函式或樣式：** 通常可以忽略，且不會改變元件的語意。
    *   **中繼資料變更：** 更新 `description` 欄位或修正文檔 typo，不需要版本升級，也不影響執行期。

### 優雅降級

**非破壞性變更依賴優雅降級。** 如果智慧體在舊客戶端上使用了新的元件或屬性，客戶端 **必須** 以優雅方式處理（例如忽略它，或渲染文字回退 /「不支援」佔位符），而不是當機。客戶端也可以把驗證錯誤回報給智慧體，讓智慧體自行修正並自動降級 UI。

#### 優雅降級範例

以下是目錄版本不一致時在實務上的處理方式：

*   **舊 iOS 客戶端使用了比智慧體更舊的目錄**
    *   智慧體傳送了一個舊 iOS 客戶端不認識的新元件 `Badge`。客戶端會為它渲染通用文字框佔位符或安全文字說明，讓其餘介面仍可運作。
    *   智慧體在 `Button` 上傳送了一個舊客戶端不認識的新屬性 `badge`。客戶端會安全忽略它，並渲染標準按鈕。
    *   智慧體不再傳送後來版本中移除的 `Facepile` 元件，這對客戶端不會造成任何問題。

*   **Web 客戶端比智慧體先部署了新目錄版本**
    *   Web 客戶端支援新的 `Badge` 元件，但智慧體還不知道它。
    *   Web 客戶端已移除 `Button` 的 `badge` 屬性，因此若智慧體傳來，它會忽略它。
    *   Web 客戶端為 `Button` 新增了智慧體不知道的新樣式；同樣不會有問題，因為智慧體根本不會使用它們。

### 搭配 catalogId 的版本控制

我們建議在 `catalogId` 中包含版本號。這樣在遷移期間就可以透過 A2UI 目錄協商同時支援多個版本，確保零停機。

**建議模式：**

| 變更類型 | URI 範例 | 說明 |
| :--- | :--- | :--- |
| **目前版本** | .../rizzcharts/v1/catalog.json | 1.x 版，支援 1.x 分支中的所有增量更新。 |
| **破壞性變更** | .../rizzcharts/v2/catalog.json | 引入破壞性結構變更的新 schema。 |

### 處理遷移

若要在不破壞現有智慧體的情況下升級目錄，請使用 A2UI 目錄協商：

1. **客戶端更新：** 客戶端在其 `supportedCatalogIds` 清單中同時加入舊版與新版（例如 `[".../v2/...", ".../v1/..."]`）。
2. **智慧體更新：** 智慧體重新以 v2 schema 建置。當它看到客戶端支援 v2 時，就會優先使用它。
3. **舊版支援：** 尚未重新建置的舊智慧體，仍會在客戶端清單中匹配到 v1，確保它們繼續可用。

## A2UI 模式驗證與回退

為了確保穩定的使用者體驗，A2UI 採用雙階段驗證策略。這種「縱深防禦」方式會盡可能早地抓出錯誤，同時確保客戶端在遇到意外 payload 時仍然穩定。

### 雙階段驗證

1. **智慧體端（送出前）：** 在傳送任何 UI payload 之前，智慧體執行環境會先根據目錄定義驗證產生的 JSON。
   * 目的：在來源端抓出幻覺出的屬性或格式錯誤。
   * 結果：若驗證失敗，智慧體可以嘗試修正或重新生成 A2UI JSON；或者在對話型應用中優雅地回退為文字。
2. **客戶端：** 收到 payload 後，客戶端函式庫會依照本地目錄定義來驗證 JSON。
   * 目的：安全與穩定。這可確保在使用者裝置上執行的程式碼嚴格符合預期合約，防止版本不一致或遭破壞的智慧體輸出。
   * 結果：若在這一層失敗，會透過「error」客戶端訊息回報給智慧體。

### 優雅降級

即使 payload 通過 schema 驗證，渲染器仍可能遇到執行期問題，例如缺少資產、某個元件實作尚未載入，或平台限制。

客戶端在遇到這些錯誤時不應當機。相反地，應該採用優雅降級：

* **未知元件：** 若 schema 中有這個元件，但渲染器尚未實作，請渲染安全回退（例如帶有該元件除錯名稱的通用卡片），或直接跳過該節點。
* **文字 fallback：** 若整個 surface 無法渲染，顯示原始文字描述（如果有的話）或一則通用錯誤訊息：「此介面無法顯示。」

### 客戶端對伺服端的錯誤回報

當客戶端偵測到驗證錯誤或執行期失敗時，可以把它回報給智慧體。這能讓智慧體系統把失敗記錄下來供開發者追查，或調整未來行為。

客戶端會使用標準 A2UI Client-to-Server Event Schema 送出 `VALIDATION_FAILED` 事件。

以下是客戶端回報缺少必要欄位的範例：

```json
{
  "version": "v0.9",
  "error": {
    "code": "VALIDATION_FAILED",
    "surfaceId": "flight-status-card-123",
    "path": "/components/FlightCard/flightNumber",
    "message": "Missing required property 'flightNumber' in component 'FlightCard'."
  }
}
```

## 內聯目錄

客戶端在執行期送出的內聯目錄是支援的，但不建議用在生產環境。更多細節可見 [這裡](../../specification/v0_9/docs/a2ui_protocol.md#client-capabilities--metadata)。
