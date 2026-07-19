# Agent 開發指南

建立會產生 A2UI 介面的 AI Agent。本指南涵蓋如何從 LLM 產生並串流 UI 訊息。

## 快速概覽

建立 A2UI Agent：

1. **理解使用者意圖** → 決定要顯示什麼 UI
2. **產生 A2UI JSON** → 使用 LLM 的結構化輸出或提示詞
3. **驗證並串流** → 檢查 schema，傳送給 client
4. **處理操作** → 回應使用者互動

## 從簡單的 Agent 開始

我們會使用 ADK 來建立一個簡單的 agent。先從文字開始，之後再升級成 A2UI。

逐步說明請參考 [ADK quickstart](https://google.github.io/adk-docs/get-started/python/)。

```bash
pip install google-adk
adk create my_agent
```

接著編輯 `my_agent/agent.py`，建立一個非常簡單的餐廳推薦 agent。

```python
import json
from google.adk.agents.llm_agent import Agent
from google.adk.tools.tool_context import ToolContext

def get_restaurants(tool_context: ToolContext) -> str:
    """Call this tool to get a list of restaurants."""
    return json.dumps([
        {
            "name": "Xi'an Famous Foods",
            "detail": "Spicy and savory hand-pulled noodles.",
            "imageUrl": "http://localhost:10002/static/shrimpchowmein.jpeg",
            "rating": "★★★★☆",
            "infoLink": "[More Info](https://www.xianfoods.com/)",
            "address": "81 St Marks Pl, New York, NY 10003"
        },
        {
            "name": "Han Dynasty",
            "detail": "Authentic Szechuan cuisine.",
            "imageUrl": "http://localhost:10002/static/mapotofu.jpeg",
            "rating": "★★★★☆",
            "infoLink": "[More Info](https://www.handynasty.net/)",
            "address": "90 3rd Ave, New York, NY 10003"
        },
        {
            "name": "RedFarm",
            "detail": "Modern Chinese with a farm-to-table approach.",
            "imageUrl": "http://localhost:10002/static/beefbroccoli.jpeg",
            "rating": "★★★★☆",
            "infoLink": "[More Info](https://www.redfarmnyc.com/)",
            "address": "529 Hudson St, New York, NY 10014"
        },
    ])

AGENT_INSTRUCTION="""
You are a helpful restaurant finding assistant. Your goal is to help users find and book restaurants using a rich UI.

To achieve this, you MUST follow this logic:

1.  **For finding restaurants:**
    a. You MUST call the `get_restaurants` tool. Extract the cuisine, location, and a specific number (`count`) of restaurants from the user's query (e.g., for "top 5 chinese places", count is 5).
    b. After receiving the data, you MUST follow the instructions precisely to generate the final a2ui UI JSON, using the appropriate UI example from the `prompt_builder.py` based on the number of restaurants."""

root_agent = Agent(
    model='gemini-2.5-flash',
    name="restaurant_agent",
    description="An agent that finds restaurants and helps book tables.",
    instruction=AGENT_INSTRUCTION,
    tools=[get_restaurants],
)
```

別忘了設定 `GOOGLE_API_KEY` 環境變數才能執行這個範例。

```bash
echo 'GOOGLE_API_KEY="YOUR_API_KEY"' > .env
```

你可以使用 ADK web 介面來測試這個 agent：

```bash
adk web
```

從清單中選擇 `my_agent`，然後詢問 New York 的餐廳。你應該會在 UI 中看到一份以純文字呈現的餐廳清單。

## 產生 A2UI 訊息

要讓 LLM 產生 A2UI 訊息，需要一些提示詞工程。

> ⚠️ **Attention**
>
> 這一塊我們仍在設計中。相關的開發者體驗目前還沒有完全定案。

目前，我們先從 contact lookup 範例複製 `a2ui_schema.py`。這是讓你的 agent 取得 A2UI schema 與範例的最簡單方式（之後可能會調整）。

```bash
cp samples/agent/adk/contact_lookup/a2ui_schema.py my_agent/
```

先為 `agent.py` 加上新的 imports：

```python
# The schema for any A2UI message.  This never changes.
from .a2ui_schema import A2UI_SCHEMA
```

接著我們會修改 agent 指令，讓它產生 A2UI 訊息而不是純文字。未來的 UI 範例先留一個 placeholder。

```python

# Eventually you can copy & paste some UI examples here, for few-shot in context learning
RESTAURANT_UI_EXAMPLES = """
"""

# Construct the full prompt with UI instructions, examples, and schema
A2UI_AND_AGENT_INSTRUCTION = AGENT_INSTRUCTION + f"""

Your final output MUST be a a2ui UI JSON response.

To generate the response, you MUST follow these rules:
1.  Your response MUST be in two parts, separated by the delimiter: `---a2ui_JSON---`.
2.  The first part is your conversational text response.
3.  The second part is a single, raw JSON object which is a list of A2UI messages.
4.  The JSON part MUST validate against the A2UI JSON SCHEMA provided below.

--- UI TEMPLATE RULES ---
-   If the query is for a list of restaurants, use the restaurant data you have already received from the `get_restaurants` tool to populate the `dataModelUpdate.contents` array (e.g., as a `valueMap` for the "items" key).
-   If the number of restaurants is 5 or fewer, you MUST use the `SINGLE_COLUMN_LIST_EXAMPLE` template.
-   If the number of restaurants is more than 5, you MUST use the `TWO_COLUMN_LIST_EXAMPLE` template.
-   If the query is to book a restaurant (e.g., "USER_WANTS_TO_BOOK..."), you MUST use the `BOOKING_FORM_EXAMPLE` template.
-   If the query is a booking submission (e.g., "User submitted a booking..."), you MUST use the `CONFIRMATION_EXAMPLE` template.

{RESTAURANT_UI_EXAMPLES}

---BEGIN A2UI JSON SCHEMA---
{A2UI_SCHEMA}
---END A2UI JSON SCHEMA---
"""

root_agent = Agent(
    model='gemini-2.5-flash',
    name="restaurant_agent",
    description="An agent that finds restaurants and helps book tables.",
    instruction=A2UI_AND_AGENT_INSTRUCTION,
    tools=[get_restaurants],
)
```

## 理解輸出

你的 agent 將不再只輸出文字，而是輸出文字加上一個 **A2UI 訊息的 JSON 清單**。

我們匯入的 `A2UI_SCHEMA` 是一個標準 JSON schema，用來定義有效操作，例如：

* `render`（顯示 UI）
* `update`（變更既有 UI 中的資料）

因為輸出是結構化 JSON，你可以在送到 client 之前先解析並驗證它。

```python
# 1. Parse the JSON
# Warning: Parsing the output as JSON is a fragile implementation useful for documentation.
# LLMs often put Markdown fences around JSON output, and can make other mistakes.
# Rely on frameworks to parse the JSON for you.
parsed_json_data = json.loads(json_string_cleaned)

# 2. Validate against A2UI_SCHEMA
# This ensures the LLM generated valid A2UI commands
jsonschema.validate(
    instance=parsed_json_data, schema=self.a2ui_schema_object
)
```

透過根據 `A2UI_SCHEMA` 驗證輸出，你可以確保 client 永遠不會收到格式錯誤的 UI 指令。

TODO：請在本指南後續補上如何解析、驗證並把輸出送到 client renderer 的範例，且不要使用 A2A 擴充。
