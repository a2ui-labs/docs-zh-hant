# 在任意 Agent 框架與 Harness 中使用 A2UI

A2UI 是一種宣告式 UI 格式。[AG-UI](https://ag-ui.com/) 是在 agent 與應用程式之間承載 A2UI 訊息的傳輸層。無論你的 AG-UI 應用或 harness 是由 ADK、LangGraph、Mastra、Strands、CrewAI、Google Chat、Slack，還是任何其他支援 AG-UI 的 agent 框架或服務所驅動，都可以依照本指南為其加入 A2UI。

<style>
  .agui-demo-video {
    border-radius: 8px;
    display: block;
    margin: 24px auto;
    max-width: 100%;
    width: 75%;
  }

  @media (max-width: 700px) {
    .agui-demo-video {
      width: 100%;
    }
  }
</style>

<video class="agui-demo-video" controls playsinline preload="metadata">
  <source src="https://cdn.copilotkit.ai/docs/a2ui/ag-ui-a2ui-demo.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

以下範例使用與 AG-UI 相容的執行環境工具，讓你可以專注在 A2UI 的部分：啟用 renderer、為 agent 提供 catalog，以及將 UI 更新即時串流回使用者。若要了解協議層級的設定與概念，請參閱 [AG-UI 文檔](https://docs.ag-ui.com/)。

## Agent 技能（Agent skills）

如果你是用 coding agent 來完成這項串接工作，請在它修改你的應用程式之前，先載入 [AG-UI 的 `ag-ui-a2ui-integration` skill](https://github.com/ag-ui-protocol/ag-ui/tree/main/skills/ag-ui-a2ui-integration)。這個 skill 涵蓋了 AG-UI 框架轉接器（adapter）、`create-ag-ui-app` 支援的旗標、傳輸層設定、A2UI 執行環境與 renderer 的串接方式，以及針對 AG-UI + A2UI 應用的端對端驗證方法。

如果你的應用是用 CopilotKit 來渲染 A2UI，也請一併載入 [CopilotKit 的 `a2ui-renderer` skill](https://github.com/CopilotKit/CopilotKit/blob/main/skills/a2ui-renderer/SKILL.md)，以取得 CopilotKit v2 執行環境、provider、主題與 catalog 相關的慣例做法。

## 1. 設定 AG-UI

從你目前已經在使用的 agent 框架開始，接著在 agent 與應用程式之間加入一條 AG-UI 執行環境連線。這個執行環境會將 agent 的事件（包含 A2UI 訊息）串流到前端介面。

使用 AG-UI CLI 來搭配你想要的客戶端與 agent 框架，快速產生一個 AG-UI 應用骨架：

```bash
npx create-ag-ui-app@latest
```

你也可以直接從支援的框架範本開始：

```bash
npx create-ag-ui-app@latest --adk
npx create-ag-ui-app@latest --langgraph-py
npx create-ag-ui-app@latest --langgraph-js
```

Strands 目前還沒有對應的腳手架旗標——你需要自行將既有的 Strands agent 包裝起來（見下方 Strands 分頁）。

真正重要的是傳輸層的約定：你的應用程式接收 AG-UI 事件，並將 A2UI payload 導向 A2UI renderer。有些腳手架路徑底層是用 [CopilotKit 的 A2UI 執行環境](https://docs.copilotkit.ai/generative-ui/a2ui) 搭配 Next.js 實作，但整體設定介面仍然以 AG-UI 為優先。

## 2. 設定你的 Agent 或 Harness

不論使用哪種框架，啟用 A2UI 的步驟都是一樣的：將你的 agent 接上 AG-UI、啟用 A2UI payload，並在應用程式中渲染這些 payload。請從你目前已經在使用的框架或 harness 開始。以下程式碼片段取自對應的 AG-UI 整合方案，展示 AG-UI 所包裝的框架原生 agent 樣貌。

=== "ADK"

    當你的 agent 已經運行在 Google 的 Agent Development Kit 之上時，可以使用 ADK。AG-UI 的 ADK middleware 會將該 agent 以 AG-UI 事件串流的形式對外公開：

    ```python
    from fastapi import FastAPI
    from ag_ui_adk import ADKAgent, AGUIToolset, add_adk_fastapi_endpoint
    from google.adk.agents import Agent

    my_agent = Agent(
        name="assistant",
        instruction="You are a helpful assistant.",
        tools=[
            AGUIToolset(),  # Adds tools provided by the AG-UI client.
        ],
    )

    agent = ADKAgent(
        adk_agent=my_agent,
        app_name="my_app",
        user_id="user123",
    )

    app = FastAPI()
    add_adk_fastapi_endpoint(app, agent, path="/chat")
    ```

    詳見 [AG-UI ADK middleware](https://github.com/ag-ui-protocol/ag-ui/tree/main/integrations/adk-middleware/python)。

=== "LangGraph（Python）"

    當你的 agent 工作流程是由一組有狀態節點所構成的圖（graph）時，可以使用 LangGraph。從你原本的 LangGraph agent 開始即可——A2UI 不需要在 graph 上額外做任何 tool 串接：

    ```python
    from copilotkit import CopilotKitMiddleware
    from langchain.agents import create_agent
    from langchain_google_genai import ChatGoogleGenerativeAI

    gemini = ChatGoogleGenerativeAI(
        model="gemini-2.5-pro",
        thinking_budget=1024,
    )

    # A plain LangGraph agent — no A2UI tool wiring on the graph. The CopilotKit
    # runtime forwards your frontend catalog and injects the `generate_a2ui` tool;
    # include CopilotKitMiddleware to get A2UI capability.
    graph = create_agent(
        model=gemini,
        tools=[],
        middleware=[CopilotKitMiddleware()],
        system_prompt="You are a helpful assistant.",
    )
    ```

    LangGraph 的 A2UI tool 是在 CopilotKit middleware 這一層運作的，因此需要加入 `CopilotKitMiddleware` 才能取得 A2UI 能力。CopilotKit 執行環境會自動轉發你的 catalog 並注入 `generate_a2ui`。此範例透過 LangChain 的 Google GenAI 整合來使用 Gemini。

    詳見 [AG-UI LangGraph 整合](https://github.com/ag-ui-protocol/ag-ui/tree/main/integrations/langgraph/python) 與 [ChatGoogleGenerativeAI 整合](https://docs.langchain.com/oss/python/integrations/chat/google_generative_ai)。

=== "LangGraph（FastAPI）"

    當你要把同一個 LangGraph graph 架在 FastAPI 應用之後對外提供服務時，可以使用 FastAPI 變體。agent 的結構完全相同——只要匯出同一個 `graph`，並透過 AG-UI 的 LangGraph endpoint 提供服務：

    ```python
    from copilotkit import CopilotKitMiddleware
    from langchain.agents import create_agent
    from langchain_google_genai import ChatGoogleGenerativeAI

    gemini = ChatGoogleGenerativeAI(
        model="gemini-2.5-pro",
        thinking_budget=1024,
    )

    graph = create_agent(
        model=gemini,
        tools=[],
        middleware=[CopilotKitMiddleware()],
        system_prompt="You are a helpful assistant.",
    )
    ```

    詳見 [AG-UI LangGraph 整合](https://github.com/ag-ui-protocol/ag-ui/tree/main/integrations/langgraph/python)。

=== "LangGraph（TypeScript）"

    當你的 LangGraph agent 是以 TypeScript 撰寫時，可以使用 TypeScript 變體。其結構與 Python 版 agent 相同——一個單純的 graph 加上 CopilotKit middleware：

    ```ts
    import { createAgent } from "langchain";
    import { ChatOpenAI } from "@langchain/openai";
    import { copilotkitMiddleware } from "@copilotkit/sdk-js/langgraph";

    export const graph = createAgent({
      model: new ChatOpenAI({ model: "gpt-4o" }),
      tools: [],
      middleware: [copilotkitMiddleware],
      systemPrompt: "You are a helpful assistant.",
    });
    ```

    詳見 [AG-UI LangGraph TypeScript 整合](https://github.com/ag-ui-protocol/ag-ui/tree/main/integrations/langgraph/typescript)。

=== "Strands（Python）"

    當你的 agent 編排（orchestration）是建立在 AWS Strands 之上時，可以使用 Strands。用 AG-UI 的 Strands adapter 包裝一個單純的 Strands agent：

    ```python
    from strands import Agent
    from ag_ui_strands import StrandsAgent

    strands_agent = Agent(
        system_prompt="You are a helpful assistant.",
    )

    agent = StrandsAgent(
        agent=strands_agent,
        name="my-agent",
        description="A Strands agent exposed via AG-UI",
    )
    ```

    詳見 [AG-UI AWS Strands 整合](https://github.com/ag-ui-protocol/ag-ui/tree/main/integrations/aws-strands/python)。

=== "Strands（TypeScript）"

    當你的 Strands agent 是以 TypeScript 撰寫時，可以使用 TypeScript 變體。AG-UI 的 Strands adapter 會將該 Strands agent 包裝成 AG-UI 客戶端可用的形式：

    ```ts
    import { Agent } from "@strands-agents/sdk";
    import { StrandsAgent } from "@ag-ui/aws-strands";
    import { createStrandsApp } from "@ag-ui/aws-strands/server";

    const strandsAgent = new Agent({
      systemPrompt: "You are a helpful assistant.",
      tools: [],
    });

    const aguiAgent = new StrandsAgent({
      agent: strandsAgent,
      name: "MyAgent",
      description: "A Strands agent exposed via AG-UI",
    });

    const app = await createStrandsApp(aguiAgent, { path: "/invocations" });
    app.listen(8000);
    ```

    詳見 [AG-UI AWS Strands 整合](https://github.com/ag-ui-protocol/ag-ui/tree/main/integrations/aws-strands/typescript)。

=== "Slack"

    當使用者體驗是承載在 Slack 應用中時，可以使用 Slack。將 Slack 討論串（thread）導向同一個 AG-UI agent endpoint，同一份 AG-UI 事件串流就能同時餵給 Slack harness，並透過該介面的客戶端橋接（client bridge）來渲染 A2UI。

    <video class="agui-demo-video" controls playsinline preload="metadata">
      <source src="https://cdn.copilotkit.ai/docs/a2ui/ag-ui-slack-demo.mp4" type="video/mp4">
      Your browser does not support the video tag.
    </video>

    CopilotKit 的 Slack adapter 已經實作了這個模式：

    ```ts
    import { createBot } from "@copilotkit/bot";
    import {
      slack,
      SanitizingHttpAgent,
      defaultSlackTools,
      defaultSlackContext,
    } from "@copilotkit/bot-slack";

    const bot = createBot({
      adapters: [
        slack({
          botToken: process.env.SLACK_BOT_TOKEN!,
          appToken: process.env.SLACK_APP_TOKEN!,
        }),
      ],
      agent: (threadId) => {
        const agent = new SanitizingHttpAgent({
          url: process.env.AGENT_URL!,
        });
        agent.threadId = threadId;
        return agent;
      },
      tools: [...defaultSlackTools],
      context: [...defaultSlackContext],
    });

    bot.onMention(async ({ thread }) => {
      await thread.runAgent();
    });

    await bot.start();
    ```

以上程式碼片段建立的是 AG-UI 伺服器端連線。Slack 透過自己的 harness 與客戶端橋接，使用同一套 AG-UI/A2UI 約定。接下來的章節會說明如何在應用介面中啟用 A2UI 渲染、catalog 與元件定義。

## 3. 啟用 A2UI

從你想要的開發體驗出發：定義 agent 可以看到的 catalog definitions、將每個 definition 對映到一個 renderer、建立 catalog，然後把這個 catalog 傳入 CopilotKit。前端的 catalog 設定就是啟用 A2UI 的目標介面。

{% raw %}

```tsx
import {CopilotKit, CopilotChat} from '@copilotkit/react-core/v2';
import {
  createCatalog,
  type CatalogDefinitions,
  type CatalogRenderers,
} from '@copilotkit/a2ui-renderer';
import {z} from 'zod';

// catalog definitions — describe the building block components to the agent
export const catalogDefinitions = {
  Card: {
    description: 'A titled card container.',
    props: z.object({title: z.string(), subtitle: z.string().optional()}),
  },
  PrimaryButton: {
    description: 'A styled primary button.',
    props: z.object({label: z.string(), action: z.any().optional()}),
  },
} satisfies CatalogDefinitions;

// catalog renderers — how each primitive renders in the DOM (React, in this example)
export const catalogRenderers = {
  Card: MyCard,
  PrimaryButton: MyPrimaryButton,
} satisfies CatalogRenderers<typeof catalogDefinitions>;

// definitions + renderers together define a catalog declaration
const catalog = createCatalog(catalogDefinitions, catalogRenderers, {
  catalogId: 'my-catalog',
  includeBasicCatalog: true,
});

<CopilotKit runtimeUrl="/api/copilotkit" a2ui={{catalog}}>
  <CopilotChat />
</CopilotKit>;
```

{% endraw %}

把 catalog 傳給 provider 會自動啟用 A2UI 並注入 `generate_a2ui` tool，因此你的 agent 不需要額外的執行環境設定就能產生 surface（CopilotKit ≥ 1.61.2）。你也可以選擇不使用這個自動機制，或是在沒有 catalog 的情況下手動啟用，方法是直接設定執行環境：

```ts title="app/api/copilotkit/route.ts"
import {CopilotRuntime} from '@copilotkit/runtime';

const runtime = new CopilotRuntime({
  agents: {default: myAgent},
  a2ui: {injectA2UITool: true},
});
```

可以用 `a2ui: { injectA2UITool: true, agents: ["my-agent"] }` 將其限制到特定 agent。如果你的流程使用固定的 schema，且 agent 已經直接回傳 `a2ui_operations`，那麼只要設定 `a2ui: true` 或 `a2ui: {}` 就足夠了。

### 自訂元件（BYOC）

A2UI 內建了一個 catalog（Text、Image、Card 等），可以讓你立即得到一個可運作的 surface。下面展開的 BYOC 流程，示範了在一個真實應用中，同樣的 catalog 模式如何拆分到不同檔案中實作。

1. **Definitions（定義）**：Zod schema 加上一段自然語言描述。這是 agent 在其 system prompt 中會看到的內容。請注意，對於 client-side function（用戶端函式），client 會在執行期讀取目前作用中 catalog definition 的設定，藉此決定該 function 的執行邊界（例如是否為 clientOnly）。
2. **Renderers（渲染器）**：有類型的 React 元件，每個 definition 對應一個。這是使用者實際看到的內容。
3. **Registration（註冊）**：透過 provider 傳入 catalog，讓 A2UI renderer 知道該如何繪製你的元件。

#### 1. 定義元件 schema

用 Zod 建立與平台無關的定義。`description` 欄位會被注入到 agent 的 prompt 中，讓 LLM 知道什麼時候該使用每個元件；schema 則用來驗證 agent 傳來的 props。

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

把每個 definition 對映到一個 React 元件。`createCatalog` 會以 definitions 的類型作為泛型參數，因此 renderer 收到的 props 會依照 Zod schema 做類型檢查，像 `props.text` 這樣的拼字錯誤會直接變成編譯錯誤。

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
  includeBasicCatalog: true, // merges with built-in components
});
```

{% endraw %}

`catalogId` 是 agent 用來定位這個 catalog 的穩定句柄；`includeBasicCatalog: true` 會讓內建元件與你自己的元件一起可用（省略它的話，就只會渲染你自己的元件）。

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

現在 agent 除了內建元件外，也會看到你的自訂元件，並能在它發出的任何 A2UI surface 中使用這些元件。

完整的 BYOC 參考資料（多個 catalog、主題化 hook、進階模式）請參閱 CopilotKit 的 [Custom Components (BYOC) 章節](https://docs.copilotkit.ai/generative-ui/a2ui)。

## 4. 進階用法

完整的 A2UI 整合介面（自訂 catalog、細粒度控制、進階模式）請參閱 CopilotKit 的 [A2UI 文檔](https://docs.copilotkit.ai/generative-ui/a2ui)。

## 接下來

- **[A2UI Composer](https://a2ui-composer.ag-ui.com/)**：以可視化方式建構 widget。
- **[概念 › 傳輸層](../concepts/transports.md)**：說明 A2UI 如何對映到 AG-UI。
- **[v0.9 規範](../specification/v0.9-a2ui.md)**：底層協議。
