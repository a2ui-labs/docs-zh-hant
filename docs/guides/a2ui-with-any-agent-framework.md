# 在任何 Agent 框架中使用 A2UI（使用 AG-UI）

A2UI 是一種宣告式 UI 格式。[AG-UI](https://ag-ui.com/) 是在 agent 與瀏覽器之間承載 A2UI 訊息的傳輸層。CopilotKit 的 AG-UI 實作是目前最快把 A2UI 放到使用者面前的路徑：CopilotKit 支援的任何 agent 框架（ADK、LangGraph、CrewAI、Mastra、自訂 Python/TS 服務等）都可以發出 A2UI，並且無需撰寫傳輸膠水程式碼就能在 React 應用中渲染。

!!! info "事實來源"

    本指南同步了 CopilotKit [ADK + A2UI 文檔](https://docs.copilotkit.ai/adk/generative-ui/a2ui)中的關鍵步驟。
    最新 API surface 請以 CopilotKit 文檔為準。

## 1. 設定 CopilotKit

在你選擇的框架（ADK、LangGraph、CrewAI、Mastra 等）的 React/Next.js 應用中安裝 CopilotKit：

```bash
npx copilotkit@latest init
```

也可以按照 [CopilotKit quickstart](https://docs.copilotkit.ai/quickstart) 將其接入現有專案。這是標準 CopilotKit 設定，不包含 A2UI 專用腳手架。

## 2. 啟用 A2UI

### 後端

在 `CopilotRuntime` 中開啟 A2UI，並注入 `render_a2ui` tool，讓 agent 可以生成 A2UI surface：

```ts title="app/api/copilotkit/route.ts"
import {CopilotRuntime} from '@copilotkit/runtime';

const runtime = new CopilotRuntime({
  agents: {default: myAgent},
  a2ui: {injectA2UITool: true},
});
```

可以用 `a2ui: { injectA2UITool: true, agents: ["my-agent"] }` 將其限制到特定 agent。

### 前端

A2UI renderer 會自動激活。也可以選擇傳入主題：

{% raw %}

```tsx
import {CopilotKitProvider} from '@copilotkit/react-core/v2';
import '@copilotkit/react-core/v2/styles.css';
import {myCustomTheme} from '@copilotkit/a2ui-renderer';

<CopilotKitProvider runtimeUrl="/api/copilotkit" a2ui={{theme: myCustomTheme}}>
  {children}
</CopilotKitProvider>;
```

{% endraw %}

### 自訂元件（BYOC）

A2UI 內置 catalog（Text、Image、Card 等），可以立即得到可工作的 surface。真正的能力在於用 **你的** React 元件擴展它：你的設計系統、你的資料形狀，讓 agent 可以從你已經信任的原語中組合介面。一個 catalog 包含三部分：

1. **Definitions**：Zod schema 加自然語言描述。這是 agent 在 system prompt 中看到的內容。
2. **Renderers**：帶類型的 React 元件，每個 definition 對應一個。這是使用者看到的內容。
3. **Registration**：透過 provider 傳入 catalog，讓 A2UI renderer 知道如何繪制你的元件。

#### 1. 定義元件 schema

用 Zod 建立與平台無關的定義。`description` 欄位會注入 agent 的 prompt，讓 LLM 知道何時應該使用每個元件；schema 會驗證 agent 發來的 props。

```ts title="lib/a2ui/definitions.ts"
import {z} from 'zod';

export const myDefinitions = {
  StatusBadge: {
    description: 'A colored status badge.',
    props: z.object({
      text: z.string(),
      variant: z.enum(['success', 'warning', 'error']).optional(),
    }),
  },
  Metric: {
    description: 'A key metric with label and value.',
    props: z.object({
      label: z.string(),
      value: z.string(),
      trend: z.enum(['up', 'down']).optional(),
    }),
  },
};

export type MyDefinitions = typeof myDefinitions;
```

#### 2. 建立 React renderer

把每個 definition 對映到一個 React 元件。`createCatalog` 會以 definitions 類型為泛型，因此 renderer 接收到的 props 會按 Zod schema 做類型檢查，`props.text` 這樣的拼寫錯誤會成為編譯錯誤。

{% raw %}

```tsx title="lib/a2ui/renderers.tsx"
'use client';

import {createCatalog, type CatalogRenderers} from '@copilotkit/a2ui-renderer';
import {myDefinitions, type MyDefinitions} from './definitions';

const myRenderers: CatalogRenderers<MyDefinitions> = {
  StatusBadge: ({props}) => {
    const colors = {
      success: {bg: '#dcfce7', text: '#166534'},
      warning: {bg: '#fef3c7', text: '#92400e'},
      error: {bg: '#fee2e2', text: '#991b1b'},
    };
    const c = colors[props.variant ?? 'success'];
    return (
      <span
        style={{
          padding: '2px 8px',
          borderRadius: 9999,
          fontSize: '0.75rem',
          background: c.bg,
          color: c.text,
        }}
      >
        {props.text}
      </span>
    );
  },

  Metric: ({props}) => (
    <div>
      <div style={{fontSize: '0.75rem', color: '#6b7280'}}>{props.label}</div>
      <div style={{fontSize: '1.5rem', fontWeight: 700}}>
        {props.value} {props.trend === 'up' ? '↑' : props.trend === 'down' ? '↓' : ''}
      </div>
    </div>
  ),
};

export const myCatalog = createCatalog(myDefinitions, myRenderers, {
  catalogId: 'my-app-catalog',
  includeBasicCatalog: true, // 與內置元件合並
});
```

{% endraw %}

`catalogId` 是 agent 用來定位這個 catalog 的穩定句柄；`includeBasicCatalog: true` 會讓內置元件與你自己的元件一起可用（省略它則只渲染你的元件）。

#### 3. 將 catalog 傳給 CopilotKit

{% raw %}

```tsx title="app/layout.tsx"
'use client';

import {CopilotKitProvider} from '@copilotkit/react-core/v2';
import '@copilotkit/react-core/v2/styles.css';
import {myCatalog} from '@/lib/a2ui/renderers';

export default function Layout({children}: {children: React.ReactNode}) {
  return (
    <CopilotKitProvider runtimeUrl="/api/copilotkit" a2ui={{catalog: myCatalog}}>
      {children}
    </CopilotKitProvider>
  );
}
```

{% endraw %}

Agent 現在會在內置元件之外看到你的自訂元件，並能在它發出的任何 A2UI surface 中使用這些元件。

完整 BYOC 參考（多個 catalog、主題 hook、高級模式）見 CopilotKit 的 [Custom Components (BYOC) section](https://docs.copilotkit.ai/adk/generative-ui/a2ui#custom-components-byoc)。

## 3. 高級用法

完整的 A2UI 集成 surface（自訂 catalog、細粒度控制、高級模式）見 CopilotKit 的 [A2UI docs](https://docs.copilotkit.ai/generative-ui/a2ui)。

## 接下來

- **[A2UI Composer](https://a2ui-composer.ag-ui.com/)**：可視化建構 widget。
- **[概念 › 傳輸層](../concepts/transports.md)**：A2UI 如何對映到 AG-UI。
- **[v0.9 規范](../specification/v0.9-a2ui.md)**：底層協議。
