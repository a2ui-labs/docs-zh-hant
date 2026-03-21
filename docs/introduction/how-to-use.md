# 我可以如何使用 A2UI？

選擇最符合你角色與使用場景的整合方式。

## 三條路徑

### 路徑 1：建立宿主應用（前端）

把 A2UI 渲染能力整合進你現有的應用，或建立一個全新的智慧體驅動前端。

**選擇渲染器：**

- **Web：** Lit、Angular、React
- **行動端 / 桌面端：** Flutter GenUI SDK

**快速安裝：**

如果是 Angular：

```bash
npm install @a2ui/angular @a2ui/web-lib
```

如果是 React：

```bash
npm install @a2ui/react @a2ui/web-lib
```

接上智慧體訊息來源（SSE、WebSockets 或 A2A），並自訂樣式，使其符合你的品牌設計。

**下一步：** [客戶端設定指南](../guides/client-setup.md) | [主題與樣式](../guides/theming.md)

---

### 路徑 2：建立智慧體（後端）

建立一個可以為任何相容客戶端生成 A2UI 回應的智慧體。

**選擇你的框架：**

- **Python：** Google ADK、LangChain、自訂方案
- **Node.js：** A2A SDK、Vercel AI SDK、自訂方案

把 A2UI schema 放進 LLM prompt 中，生成 JSONL 訊息，並透過 SSE、WebSockets 或 A2A 串流傳送到客戶端。

**下一步：** [智慧體開發指南](../guides/agent-development.md)

---

### 路徑 3：使用現成框架

透過內建支援 A2UI 的框架來使用：

- **[AG UI / CopilotKit](https://ag-ui.com/)** - 具備 A2UI 渲染能力的全端 React 框架
- **[Flutter GenUI SDK](https://docs.flutter.dev/ai/genui)** - 跨平台生成式 UI（底層使用 A2UI）

**下一步：** [智慧體 UI 生態](agent-ui-ecosystem.md) | [A2UI 在哪裡被使用？](../ecosystem/a2ui-in-the-world.md)
