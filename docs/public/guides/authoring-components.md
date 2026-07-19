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

使用客戶端框架實作元件。對於 Angular，元件應繼承 `@a2ui/angular` 提供的 `DynamicComponent`。

在 [`orchestrator`](../../../samples/community/client/angular/projects/orchestrator/README.md) 範例中，`Chart` 元件定義在 [`chart.ts`](../../../samples/community/client/angular/projects/orchestrator/src/a2ui-catalog/chart.ts)。

{% raw %}

```typescript
import {DynamicComponent} from '@a2ui/angular';
import * as Primitives from '@a2ui/web_core/types/primitives';
import * as Types from '@a2ui/web_core/types/types';
import {Component, computed, input, Signal, signal} from '@angular/core';

@Component({
  selector: 'a2ui-chart',
  template: `
    <div>
      <h2>{{ resolvedTitle() }}</h2>
      <canvas baseChart [data]="currentData()" [type]="chartType()"></canvas>
    </div>
  `,
})
export class Chart extends DynamicComponent<Types.CustomNode> {
  readonly type = input.required<string>();
  protected readonly chartType = computed(() => this.type() as ChartType);

  readonly title = input<Primitives.StringValue | null>();
  protected readonly resolvedTitle = computed(() => super.resolvePrimitive(this.title() ?? null));

  readonly chartData = input.required<Primitives.StringValue | null>();
  // ... data resolution logic using super.resolvePrimitive for data paths
}
```

{% endraw %}

實作元件時請注意這些要點：

- **繼承 `DynamicComponent`**：這樣可以存取 `resolvePrimitive` 來解析資料繫結。
- **使用 Angular Inputs**：將 schema 中的屬性對映到 Angular input。

---

## 3. 注冊到 Renderer（客戶端）

元件實作完成後，將其注冊到客戶端 catalog。這個步驟會把元件名（agent 使用的名稱）對映到實作類。

在 [`orchestrator`](../../../samples/community/client/angular/projects/orchestrator/README.md) 範例中，這在 [`catalog.ts`](../../../samples/community/client/angular/projects/orchestrator/src/a2ui-catalog/catalog.ts) 中完成。

```typescript
import {Catalog, DEFAULT_CATALOG} from '@a2ui/angular';
import {inputBinding} from '@angular/core';

export const RIZZ_CHARTS_CATALOG = {
  ...DEFAULT_CATALOG,
  Chart: {
    type: () => import('./chart').then(r => r.Chart),
    bindings: ({properties}) => [
      inputBinding('type', () => ('type' in properties && properties['type']) || undefined),
      inputBinding('title', () => ('title' in properties && properties['title']) || undefined),
      inputBinding(
        'chartData',
        () => ('chartData' in properties && properties['chartData']) || undefined,
      ),
    ],
  },
} as Catalog;
```

注冊要點：

- **懶加載**：使用 `import()` 懶加載元件程式碼。
- **Input 繫結**：使用 `inputBinding` 將 schema 屬性對映到 Angular input。

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
