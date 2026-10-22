# designer-module-agent

智能体管理后端：CRUD + Solon AI Harness 调试对话。

- `agent-api`：契约占位
- `agent-service`：实现（`/admin-api/agent/**`）

表结构：`sql/init/mysql/agent/schema.sql`  
菜单与演示种子：`sql/init/mysql/agent/data.sql`（DeepSeek 对话 + 阿里云百炼 Embedding/`text-embedding-v4`、Rerank/`qwen3-rerank`；含国内天气 MCP；分别填写对应 API Key）

## 包结构

对齐 system 模块：先分层、再按业务域分子包。

| 业务子包 | 说明 |
|----------|------|
| `agent` | 智能体定义 / 商店 |
| `session` | 会话、消息、调试对话 |
| `provider` | 模型供应商、厂商适配 |
| `knowledge` | 知识库、文档、RAG 索引检索 |
| `workflow` | 工作流 CRUD（引擎在根包 `workflow/`） |
| `tool` / `skill` / `mcp` | 工具、Skill、MCP |

分层示例：`controller.admin.session`、`service.session`、`mapper.session`、`model.dto.session`。

Service：接口在 `service.{域}`，实现类在 `service.{域}.impl`（`@Component`）。

根包保留：

- `harness/`：对话运行时公共（异步线程池、MCP/Skill 挂载、工具映射）
- `workflow/`：工作流引擎（activity / node / support）
- `constant/`、`util/`
