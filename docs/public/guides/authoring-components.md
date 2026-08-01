# 撰寫自訂元件

了解如何在 A2UI 中定義、實作和注冊自訂元件。本指南以 `rizzcharts` 範例為例，重點說明如何圍繞你的 Angular 程式碼來撰寫元件。

## 概覽

撰寫一個新元件主要包含四個步驟：

1.  **定義 Catalog Schema**：在 JSON Schema 中指定元件屬性和類型。
2.  **定義元件（客戶端）**：使用你的框架（例如 Angular）實作 UI。
3.  **注冊到 Renderer（客戶端）**：將元件加入客戶端 catalog。
4.  **從 Agent 呼叫**：透過 `send_a2ui_json_to_client` 指示 agent 使用該元件。

---

## 1. 定義 Catalog Schema

Catalog schema 定義了你的 catalog API。它列出可用元件及其屬性，agent 會用這些資訊構造 UI payload。

**這個 schema 是客戶端與伺服端（agent）之間的契約。** 兩端必須對該 schema 達成一致，渲染才能工作。客戶端宣告自己支援哪些 catalog，伺服端選擇相容的 catalog。關於這個握手如何工作，見 [A2UI Catalog Negotiation](../concepts/catalogs.md#a2ui-catalog-negotiation)。

在 [`rizzcharts`](../../../samples/community/agent/adk/rizzcharts/python/README.md) 範例中，catalog schema 定義在 [`rizzcharts_catalog_definition.json`](../../../samples/community/agent/adk/rizzcharts/catalog_schemas/0.9/rizzcharts_catalog_definition.json)。

下面是 `Chart` 元件的 schema：

```json
"Chart": {
  "type": "object",
  "description": "An interactive chart that uses a hierarchical list of objects for its data.",
  "properties": {
    "type": {
      "type": "string",
      "description": "The type of chart to render.",
      "enum": [
        "doughnut",
        "pie"
      ]
    },
    "title": {
      "type": "object",
      "description": "The title of the chart. Can be a literal string or a data model path.",
      "properties": {
        "literalString": {
          "type": "string"
        },
        "path": {
          "type": "string"
        }
      }
    },
    "chartData": {
      "type": "object",
      "description": "The data for the chart, provided as a list of items. Can be a literal array or a data model path.",
      "properties": {
        "literalArray": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "label": {
                "type": "string"
              },
              "value": {
                "type": "number"
              },
              "drillDown": {
                "type": "array",
                "description": "An optional list of items for the next level of data.",
                "items": {
                  "type": "object",
                  "properties": {
                    "label": {
                      "type": "string"
                    },
                    "value": {
                      "type": "number"
                    }
                  },
                  "required": [
                    "label",
                    "value"
                  ]
                }
              }
            },
            "required": [
              "label",
              "value"
            ]
          }
        },
        "path": {
          "type": "string"
        }
      }
    }
  },
  "required": [
    "type",
    "chartData"
  ]
}
```

---

## 2. 實作元件（客戶端）

使用客戶端框架實作元件。對於 Angular，元件應繼承 `@a2ui/angular/v0_9` 提供的 `CatalogComponent`。

在 `rizzcharts` 範例中，`Chart` 元件定義在 `chart.ts`。

首先，用 TypeScript 定義元件 API。這應該要對應到步驟 1 中定義的 JSON Schema。

```typescript
// api.ts
import {ComponentApi} from '@a2ui/web_core/v0_9';
import {z} from 'zod';

export const ChartApi = {
  name: 'Chart',
  schema: z.object({
    type: z.enum(['doughnut', 'pie']),
    title: z.string().optional(),
    chartData: z.array(
      z.object({
        label: z.string(),
        value: z.number(),
        drillDown: z.array(
          z.object({
            label: z.string(),
            value: z.number(),
          })
        ).optional(),
      })
    ),
  }).strict(),
} satisfies ComponentApi;
```

接著實作 Angular 元件：

```typescript
import {CatalogComponent} from '@a2ui/angular/v0_9';
import {Component, computed} from '@angular/core';
import {BaseChartDirective} from 'ng2-charts';
import {ChartApi} from './api';

@Component({
  selector: 'a2ui-chart',
  imports: [BaseChartDirective],
  template: `
    <div>
      <h2>{{ title() }}</h2>
      <canvas baseChart [data]="chartData()" [type]="chartType()"></canvas>
    </div>
  `,
})
export class Chart extends CatalogComponent<typeof ChartApi> {
  protected readonly chartType = computed(() => this.props()['type']?.value() || 'pie');
  protected readonly title = computed(() => this.props()['title']?.value() || '');
  protected readonly chartData = computed(() => {
    const rawData = this.props()['chartData']?.value() || [];
    return {
      labels: rawData.map(item => item.label),
      datasets: [
        {
          data: rawData.map(item => item.value),
        },
      ],
    };
  });
}
```

實作元件時請注意這些要點：

- **繼承 `CatalogComponent`**：這樣可以存取型別安全的 `props` signal input。
- **使用 `props()` Signal**：透過 `this.props()['propertyName']?.value()` 以反應式方式存取已解析的屬性。框架會自動處理資料繫結與運算式的解析。

---

## 3. 注冊到 Renderer（客戶端）

元件實作完成後，將其注冊到客戶端 catalog。這個步驟會把元件名（agent 使用的名稱）對映到實作類。

你可以使用 `AngularCatalog` 類別來定義你的 catalog。

```typescript
import {AngularCatalog, BASIC_COMPONENTS, BASIC_FUNCTIONS} from '@a2ui/angular/v0_9';
import {Chart} from './chart';
import {ChartApi} from './api';

const customChartComponent = {
  ...ChartApi,
  component: Chart
};

export const RIZZ_CHARTS_CATALOG = new AngularCatalog(
  'https://github.com/.../rizzcharts_catalog_definition.json',
  [...BASIC_COMPONENTS, customChartComponent],
  BASIC_FUNCTIONS
);
```

注冊要點：

- **即時注冊**：元件類別會直接在 catalog 定義中注冊。

---

## 4. 從 Agent 呼叫

要使用自訂元件，需要用 A2UI SDK 中理解你的 catalog 的工具初始化 agent。SDK 會負責解析 catalog 並向模型提供範例。

整體流程如下：

### 4.1 會話準備（Executor）

執行層（例如 `RizzchartsAgentExecutor`）會攔截傳入訊息，檢測 A2UI 是否啟用以及客戶端支援哪些 catalog。它會解析 catalog 並保存到 session state。

```python
# In agent_executor.py

use_ui = try_activate_a2ui_extension(context)
if use_ui:
    # Resolve catalog based on client capabilities
    a2ui_catalog = self.schema_manager.get_selected_catalog(
        client_ui_capabilities=capabilities
    )
    examples = self.schema_manager.load_examples(a2ui_catalog, validate=True)

    # Save to session (Event contains state_delta)
    await runner.session_service.append_event(
        session,
        Event(
            actions=EventActions(
                state_delta={
                    _A2UI_ENABLED_KEY: True,
                    _A2UI_CATALOG_KEY: a2ui_catalog,
                    _A2UI_EXAMPLES_KEY: examples,
                }
            ),
        ),
    )
```

### 4.2 Agent 工具設定

Agent 使用 [SendA2uiToClientToolset](../../../agent_sdks/python/a2ui_agent/src/a2ui/adk/send_a2ui_to_client_toolset.py) 獲得一個可用於向客戶端送出 A2UI 的工具。

```python
from a2ui.adk.send_a2ui_to_client_toolset import SendA2uiToClientToolset

a2ui_catalog = self.schema_manager.get_selected_catalog(
    client_ui_capabilities=capabilities
)
agent.tools = [
    SendA2uiToClientToolset(
        a2ui_catalog=a2ui_catalog,
        a2ui_enabled=True,
    )
]
```

### 4.3 工具執行

LLM 對 [SendA2uiToClientToolset](../../../agent_sdks/python/a2ui_agent/src/a2ui/adk/send_a2ui_to_client_toolset.py) 中工具的呼叫，會在 A2A Agent Executor 中透過 [A2uiEventConverter](../../../agent_sdks/python/a2ui_agent/src/a2ui/adk/a2a/event_converter.py) 被攔截。這會自動把工具呼叫轉換為帶有 A2UI payload 的 A2A DataPart。

```python
from a2ui.adk.a2a.event_converter import (
    A2uiEventConverter,
)

config = A2aAgentExecutorConfig(event_converter=A2uiEventConverter())
```
