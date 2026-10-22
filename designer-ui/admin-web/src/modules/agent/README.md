# 智能体模块

后端：`designer-module-agent`（Solon AI Harness 驱动调试对话）。前端：`api/index.js` 请求 `/admin-api/agent/*`。

| 页面 | component |
|---|---|
| 智能体商店（使用者） | `agent/store/index` |
| 供应商管理 | `agent/provider/index` |
| 智能体管理 | `agent/agent/index` |
| 智能体编辑 | `agent/agent/edit` |
| 工具管理 | `agent/tool/index`（平台内置，仅启停） |
| MCP 管理 | `agent/mcp/index` |
| Skill 管理 | `agent/skill/index`（支持上传 `.zip`，解析 `SKILL.md`） |
| 工作流 | `agent/workflow/index` |
| 工作流编辑 | `agent/workflow/edit`（画布编排，本期只持久化） |
| 知识库 | `agent/knowledge/index` |
| 知识库详情 | `agent/knowledge/detail`（文档截断注入对话上下文） |
| 会话记录 | `agent/session/index`（表格按单次问答展示） |
| 调试对话 | `agent/session/chat`（SSE 流式 + Markdown 渲染） |

能力绑定：`toolIds` / `mcpIds` / `skillIds` / `workflowIds` / `knowledgeIds`；`fileEnabled` 控制对话是否可上传附件。

SQL：`sql/init/mysql/agent/schema.sql`、`data.sql`（菜单 ID 6000 段）。

配置：`designer.agent.workspace`（默认 `./data/agent-harness`）。
