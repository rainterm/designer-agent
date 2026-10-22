# 智能体平台

[![license](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![JDK](https://img.shields.io/badge/JDK-25-orange.svg)](https://adoptium.net/temurin/releases/)
[![Solon](https://img.shields.io/badge/Solon-4.1.0-16A34A.svg)](https://solon.noear.org/)
[![Vue](https://img.shields.io/badge/Vue-3.5-42b883.svg)](https://vuejs.org/)

## 简介

面向企业后台的**智能体管理与运行平台**：在管理端配置模型供应商、工具 / MCP / Skill、知识库与工作流，组装并发布智能体，通过 SSE 流式对话调试与使用。

本项目基于开源脚手架 **[designer-admin](https://gitee.com/rainterm/designer-admin)**（Solon + MyBatis-Plus + Vue 3 + Naive UI）开发，继承其登录、RBAC、文件、消息、定时任务等基础能力；本仓库的增量业务集中在 `designer-module-agent` 与前端 `modules/agent`。脚手架本身的设计说明与通用能力，请直接参阅上游仓库。

设计上与上游一致，坚持两条原则：

- **简洁、可复用**：智能体运行时复用 Solon AI Harness（对话、工具、MCP、Skill 挂载），业务模块只写配置与编排差异。
- **易扩展、可维护**：供应商方言、MCP 传输、工作流节点、内置工具均按类型注册扩展；新增能力不改核心对话链路。

### 为什么基于 Solon + Solon AI

- **脚手架成熟**：designer-admin 已解决鉴权、权限、文件、部署等后台共性，可专注智能体业务。
- **AI 运行时一体**：Solon AI / Harness / MCP / Flow 与 Solon 插件体系同栈，对话、工具调用、流式输出装配成本低。
- **启动快、易部署**：`java -jar` 单体运行，配合 `deploy/` 脚本即可本机打包 + Docker 部署。
- **贴合现代 JDK**：基于 JDK 25，虚拟线程承载 SSE 对话、工作流与知识库索引等 I/O 旁路任务。

## 内置能力

| 能力 | 说明 |
|---|---|
| 智能体商店 | 浏览已启用且已发布的智能体，一键进入会话对话 |
| 智能体管理 | 草稿保存 / 发布 / 历史版本 / 回滚；绑定工具、MCP、Skill、工作流、知识库 |
| 模型供应商 | 多厂商接入与拉模型列表（OpenAI 兼容、Responses、Ollama、Anthropic、Gemini、DashScope 等） |
| 工具 | 平台内置工具启停：`web_search` / `http_request` / `filesystem` / `browser` |
| MCP | 管理外部 MCP 服务（stdio / SSE / Streamable HTTP 等），探测可用工具 |
| Skill | 手工维护或上传 `.zip`（解析 `SKILL.md`），挂载到 Harness 工作区 |
| 知识库 / RAG | 文档上传与切片、Milvus 向量索引、检索测试；对话侧 Embedding + 可选 Rerank |
| 工作流 | 画布编排、发布与版本、同步 / SSE 运行；可作为智能体工具挂载 |
| 会话与调试 | 会话记录、SSE 流式对话（delta / tool / thinking / done 等事件）+ Markdown 渲染 |

### 智能体商店与管理

- **商店**：面向使用者，分页列出已启用且已发布的智能体，创建会话后进入聊天页。
- **管理**：配置内容（提示词、模型、采样、能力绑定、RAG）写入草稿 `draft_json`，**保存不影响线上运行**；**发布**将草稿提升为正式配置并写入版本历史；支持历史查看与回滚（回滚回到草稿）。
- 能力绑定字段：`toolIds` / `mcpIds` / `skillIds` / `workflowIds` / `knowledgeIds`；`fileEnabled` 控制对话是否可上传附件。
- RAG 可单独指定 Embedding / Rerank 供应商与模型，以及 TopK、相似度阈值。

### 模型供应商

- 后台维护 Base URL、API Key、方言类型，支持一键**拉取模型列表**并按能力归类（对话 / Embedding / Rerank 等）。
- 内置方言预设：`openai`、`openai-responses`、`ollama`、`anthropic`、`gemini`、`dashscope`；OpenAI 兼容模式可覆盖 DeepSeek、Kimi、智谱、硅基流动、火山、百炼兼容模式等。
- 种子数据默认预置 DeepSeek（`deepseek-flash`），填写 API Key 即可开始调试对话。

### 工具 · MCP · Skill

- **内置工具**：平台侧维护，仅支持启停；映射到 Solon AI Harness 工具名（含浏览器类走 webfetch）。
- **MCP**：支持 `stdio`、`sse`、`streamable` / `streamable_http` / `streamable_stateless`、`http` 等传输；可探测连通性与工具列表。种子数据含国内天气 MCP（`npx -y weather-china-mcp`，需本机 Node）。
- **Skill**：支持手工录入与 **`.zip` 包导入**（解析 `SKILL.md`），运行时挂载到 `designer.agent.workspace`。

### 知识库与 RAG

- 知识库 + 文档：支持上传、纯文本、URL；切片策略 `fixed` / `section_recursive`；PDF / DOCX 解析。
- 索引写入 **Milvus**（每个知识库独立 collection），支持重新索引与 SSE 进度；提供检索测试接口。
- 对话时按智能体绑定的知识库做向量检索，可选 Rerank 后再注入上下文。
- 向量库通过 `designer.agent.milvus.*`（或环境变量 `MILVUS_*`）配置；本地可用 `deploy/deploy-milvus.sh` 一键启动 Standalone。

### 工作流

- CRUD + 发布 / 历史 / 回滚；支持同步运行与 **SSE 流式运行**。
- 节点类型：`start` / `end` / `llm` / `http` / `knowledge` / `code`（JS Nashorn / Python GraalVM）/ `sql` / `output` / `condition`。
- 已发布工作流可挂到智能体上，作为 Function Tool 被模型调用。

### 会话与调试对话

- 基于 **Solon AI Harness** 执行调试对话：组装模型、系统提示、工具 / MCP / Skill / 工作流 / 知识库检索结果。
- `send-stream` 输出 SSE 事件：`user`、`delta`、`tool`、`status`、`thinking`、`step`、`progress`、`done`、`error`。
- 异步执行受 `designer.agent.async.max-concurrent` 限制（默认 64，虚拟线程池）。
- 前端聊天页：流式 Markdown、工具轨迹、思考过程等组件。

## 设计

### 技术栈选择

| 层 | 选型 | 版本 | 选择理由 |
|---|---|---|---|
| 运行时 | JDK | 25 | 虚拟线程承载 SSE / 索引 / 工作流旁路任务 |
| 后端框架 | Solon | 4.1.0 | 与上游脚手架一致，轻量可插件化 |
| AI 运行时 | Solon AI / Harness / MCP / Flow | 随 Solon | 对话、工具、MCP、流程编排同栈 |
| ORM | MyBatis-Plus | 3.5.17 | 智能体配置与会话落库 |
| 鉴权 | Sa-Token | 1.46.0 | 继承 designer-admin 登录与权限 |
| 向量库 | Milvus | 2.5.x | 知识库 Embedding 检索 |
| 数据库 | MySQL | 8.0.13+ | 配置、会话、版本历史 |
| 缓存 / 会话 | Redis | 7 | 登录会话与脚手架横切能力 |
| 前端 | Vue 3.5 + Naive UI + Vite | — | 与上游管理端同一套壳层 |

脚手架通用技术选型（限流、消息总线、存储、Quartz 等）见 [designer-admin](https://gitee.com/rainterm/designer-admin)。

### 分层结构

```
designer-agent/
├── designer-common/                 # 通用契约（上游脚手架）
├── designer-framework/              # 鉴权、限流、存储、消息等（上游脚手架）
├── designer-module-system/          # RBAC、文件、消息、任务等（上游脚手架）
├── designer-module-demo/            # 业务模块模板
├── designer-module-agent/           # ★ 智能体业务
│   ├── agent-api/                   # 跨模块契约占位
│   └── agent-service/               # /admin-api/agent/** 实现
│       └── .../module/agent/
│           ├── controller/admin/    # agent / provider / tool / mcp / skill / workflow / knowledge / session
│           ├── service/{域}/        # 接口 + impl
│           ├── mapper/{域}/
│           ├── model/{dto,entity,vo}/{域}/
│           ├── harness/             # 对话运行时：异步池、MCP/Skill 挂载、工具映射
│           └── workflow/            # 工作流引擎：activity / node / support
├── designer-server/                 # 启动组装 + app.yml
├── designer-ui/admin-web/
│   └── src/modules/agent/           # ★ 智能体前端（api / views / components）
├── sql/init/mysql/agent/            # schema + 演示种子
├── deploy/                          # Docker 脚本（含 deploy-milvus.sh）
└── docs/                            # 开发规范等（与上游一致）
```

一次调试对话的典型路径：

```
Chat Controller（SSE）
    → AgentChatService（落库用户消息、组装配置）
        → Solon AI Harness（模型 + 工具 / MCP / Skill / 工作流工具）
            → 知识库检索（Embedding → Milvus → 可选 Rerank）
                → SSE 事件流回写前端
```

### 前端页面

| 页面 | component |
|---|---|
| 智能体商店 | `agent/store/index` |
| 供应商管理 | `agent/provider/index` |
| 智能体管理 / 编辑 | `agent/agent/index` · `agent/agent/edit` |
| 工具管理 | `agent/tool/index` |
| MCP 管理 | `agent/mcp/index` |
| Skill 管理 | `agent/skill/index` |
| 工作流 / 编辑 | `agent/workflow/index` · `agent/workflow/edit` |
| 知识库 / 详情 | `agent/knowledge/index` · `agent/knowledge/detail` |
| 会话记录 / 调试对话 | `agent/session/index` · `agent/session/chat` |

模块说明见 [designer-module-agent/README.md](designer-module-agent/README.md) 与 [designer-ui/.../agent/README.md](designer-ui/admin-web/src/modules/agent/README.md)。

### 配置项

| 配置 | 环境变量 | 说明 |
|---|---|---|
| `designer.agent.workspace` | `AGENT_WORKSPACE` | Harness 工作区（默认 `./data/agent-harness`） |
| `designer.agent.async.max-concurrent` | `AGENT_ASYNC_MAX_CONCURRENT` | SSE / 工作流 / 索引进度并发上限（默认 64） |
| `designer.agent.milvus.uri` | `MILVUS_URI` | Milvus 地址（默认 `http://localhost:19530`） |
| `designer.agent.milvus.token` | `MILVUS_TOKEN` | 鉴权 Token（可空） |
| `designer.agent.milvus.database` | `MILVUS_DATABASE` | 库名（默认 `default`） |

### 部署脚本

在上游脚本基础上增加向量库：

| 脚本 | 作用 |
|---|---|
| `install-env.sh` | 检查 / 安装 Docker、JDK 25、Maven、Node.js |
| `deploy-db.sh` | 启动 MySQL + Redis |
| `deploy-milvus.sh` | 启动 Milvus Standalone（知识库 RAG 需要） |
| `deploy-app.sh` | 本地打包并启动 Server / Web |
| `deploy-minio.sh` | 可选，S3 兼容对象存储 |
| `stop.sh` / `restart.sh` / `logs.sh` | 停止、重启、查看日志 |

### 文档说明

| 文档 | 说明 |
|---|---|
| [designer-admin](https://gitee.com/rainterm/designer-admin) | 上游脚手架：登录、RBAC、文件、消息、SSO、部署设计等 |
| [docs/开发规范.md](docs/开发规范.md) | 前后端编码约定、模块依赖方向 |
| [designer-module-agent/README.md](designer-module-agent/README.md) | 智能体后端包结构与种子数据说明 |
| [designer-ui/.../agent/README.md](designer-ui/admin-web/src/modules/agent/README.md) | 智能体前端页面与路由 component |
| [deploy/](deploy) | Docker 部署脚本与 Nginx / Dockerfile |

## 快速开始（本地开发）

### 0. 环境要求

| 组件 | 版本 | 说明 |
|---|---|---|
| JDK | 25+ | 后端编译与运行 |
| Maven | 3.9+ | 后端构建 |
| Node.js | 18+（推荐 22） | 前端构建；stdio MCP 种子示例也需要 |
| pnpm | 10.x | 前端包管理 |
| MySQL | 8.0.13+ | 数据库 |
| Redis | 7 | 会话与脚手架横切能力（**必需**） |
| Milvus | 2.5.x | 知识库向量检索（用 RAG 时需要） |

```bash
cd deploy
./install-env.sh --check   # 仅检查
./install-env.sh           # 按需安装（Linux 建议 sudo）
```

### 1. 准备 MySQL、Redis（及可选 Milvus）

```bash
cd deploy
cp .env.example .env
./deploy-db.sh             # MySQL + Redis，不自动建表
./deploy-milvus.sh         # 可选：知识库 RAG
```

连接信息在 `designer-server/src/main/resources/app.yml`，可用 `DB_*` / `REDIS_*` / `MILVUS_*` 覆盖。

### 2. 初始化数据库

> 脚本为**破坏性初始化**（`DROP TABLE` 重建 + 种子数据），仅首次或需要重置时执行。

```bash
bash sql/init/mysql/00-run-init.sh

# 免交互示例
DB_HOST=127.0.0.1 DB_PORT=3306 DB_USER=root DB_PASSWORD=root DB_NAME=designer_agent \
  bash sql/init/mysql/00-run-init.sh
```

初始化顺序：`_base → system → demo → agent`。智能体种子含 DeepSeek 供应商、内置工具、天气 MCP、演示 Skill / 知识库 / 工作流，以及多个已发布演示智能体；对话前请在「供应商管理」填写 API Key。

### 3. 启动后端

```bash
mvn -pl designer-server -am -DskipTests clean package
java -jar designer-server/target/designer-server.jar
```

- 管理端 API：`http://localhost:8080/admin-api`
- 智能体 API：`http://localhost:8080/admin-api/agent/**`
- 接口文档：`http://localhost:8080/doc.html`

### 4. 启动前端

```bash
cd designer-ui/admin-web
pnpm install
pnpm run dev      # http://localhost:5173
```

### 默认账号

| 账号 | 密码 | 角色 |
|---|---|---|
| admin | admin123 | 超级管理员 |
| guest | guest123 | 普通用户（只读示例） |

> 演示账号，**部署到生产环境前必须修改密码或删除**。

### 建议体验路径

1. 登录后进入 **供应商管理**，为 DeepSeek（或自建供应商）填写 API Key。
2. 打开 **智能体商店**，选择 `demo-assistant` 等演示智能体开始对话。
3. 使用知识库 RAG 前：配置 Embedding 供应商 → 知识库详情中重建索引（需 Milvus 已启动）。
4. 使用天气 MCP 演示前：本机可执行 `npx`（Node.js）。

## 生产部署

数据库、向量库与应用**分开部署**：本机打包后构建运行镜像。

```bash
cd deploy
./install-env.sh
cp .env.example .env   # 修改密码、MILVUS_URI、VAULT_PASSWORD 等
./deploy-db.sh
./deploy-milvus.sh     # 使用知识库时
(cd .. && bash sql/init/mysql/00-run-init.sh)
./deploy-app.sh
```

| 入口 | 默认端口 | 说明 |
|---|---|---|
| 管理后台 | `APP_PORT=80` | `http://<服务器IP>` |
| 后端 API | `SERVER_PORT=8080` | 默认仅绑定本机，经 Nginx 转发 |
| Milvus | `MILVUS_PORT=19530` | 应用通过 `MILVUS_URI` 连接（容器内常用 `host.docker.internal`） |

发版日常执行 `./deploy-app.sh` 即可；初始化脚本勿对有数据的生产库重复执行。更多运维约定（反向代理、可信 IP、多实例 Redis / Quartz）与上游 [designer-admin](https://gitee.com/rainterm/designer-admin) 相同。

## 许可证

本项目基于 [MIT License](LICENSE) 开源，可自由用于商业项目。

底座脚手架：[designer-admin](https://gitee.com/rainterm/designer-admin)（同样 MIT）。
