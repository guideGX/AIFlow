# AiFlow

AiFlow 是一个开源的 AI Agent 开发平台，支持通过提示词（Prompt）、工作流（Workflow）灵活创建和编排智能体。平台整合了丰富的模型、插件和 MCP Server，支持一站式效果测评，助力开发者快速搭建生产级智能体。

## 功能特性

- **智能体管理** — 创建、配置、部署 AI Agent，支持 Chat、CoT、CoT Process 等多种类型
- **可视化工作流** — 基于 DSL 的节点图执行引擎，拖拽式构建复杂工作流
- **对话交互** — 与智能体实时对话，支持流式输出和多轮上下文
- **插件生态** — 可扩展的插件架构，支持知识库、MCP、自定义工具等
- **多租户空间** — 个人空间 + 企业空间，支持团队协作和权限管理
- **模型管理** — 灵活接入多种大模型，支持效果测评和对比
- **发布管理** — 一键发布为 API、MCP Server，或发布到应用市场

## 技术架构

```
浏览器 → Console Frontend (React 18 + TypeScript, :3000)
              ↓ REST + SSE
         Console Hub (Java 21 + Spring Boot, :8080)
           ↙        ↘
   Core Agent       Core Workflow
   (Python/FastAPI)  (Java/Spring Boot)
              ↓
   共享基础设施：MySQL + Redis + MinIO
```

### 技术栈

| 层级 | 技术 |
|------|------|
| **前端** | React 18、TypeScript、Vite、Ant Design、Tailwind CSS、Zustand + Recoil |
| **后端 (Hub)** | Java 21、Spring Boot 3.5、Spring Security、MyBatis Plus、Maven |
| **工作流引擎** | Java 21、Spring Boot 3.5、DSL 节点图执行、SSE 回调 |
| **Agent 服务** | Python 3.11+、FastAPI、SQLAlchemy 2.0、Pydantic |
| **基础设施** | MySQL 8、Redis 7、MinIO、Docker Compose |

## 项目结构

```
AiFlow/
├── console/                    # Web 控制台（全栈）
│   ├── frontend/               # React 18 + TypeScript + Vite
│   └── backend/                # Java 21 Spring Boot（Maven 多模块）
│       ├── commons/            # 公共模块：DTO、工具类、通用配置
│       ├── hub/                # 核心 API 服务：控制器、业务逻辑、数据层
│       └── toolkit/            # 扩展工具：WebSocket、SSE、任务调度
├── core/                       # Python 微服务
│   ├── common/                 # 共享工具库
│   ├── agent/                  # AI Agent 服务（FastAPI）
│   ├── workflow/               # Python 工作流引擎
│   └── plugin/                 # 插件服务（aitools、link）
├── core-workflow-java/         # Java 工作流引擎
├── docker/                     # Docker Compose 部署配置
├── scripts/                    # 数据库初始化脚本
└── Makefile                    # CI/CD 编排
```

## 快速开始

### 环境要求

- **JDK 21** — 后端编译运行
- **Node.js 18+** — 前端构建
- **MySQL 8** — 数据存储
- **Redis 7** — 缓存和会话
- **MinIO** — 对象存储（可选）

### 启动后端 (Hub)

```bash
cd console/backend

# 设置环境变量
export MYSQL_HOST=localhost
export MYSQL_PORT=3306
export MYSQL_DB=paiflow-console
export MYSQL_USER=root
export MYSQL_PASSWORD=your_password
export REDIS_HOST=localhost

# 构建并启动
mvn clean install -DskipTests
mvn spring-boot:run -pl hub
```

后端服务默认运行在 `http://localhost:8080`。

### 启动前端

```bash
cd console/frontend

npm install
npm run dev
```

前端开发服务器默认运行在 `http://localhost:3000`。

### Docker 一键部署

```bash
cd docker/PaiFlow
docker compose up
```

启动全部服务：MySQL (3307)、Redis、MinIO (9000/9001)、前端 (3000→1881)、Hub (8081→8080)、工作流引擎 (7880)。

## 开发指南

### 后端代码规范

项目使用以下代码质量工具：

- **Spotless** — 代码格式化
- **Checkstyle** — 编码规范检查
- **SpotBugs** — 静态缺陷分析
- **PMD** — 代码质量扫描

```bash
mvn spotless:apply          # 格式化代码
mvn checkstyle:check        # 检查编码规范
mvn spotbugs:check          # 静态分析
```

### 前端开发

```bash
npm run dev          # 启动开发服务器（热更新）
npm run build        # 生产构建
npm run quality      # 完整检查：格式化 + Lint + 类型检查
npm run lint:fix     # 自动修复 ESLint 错误
npm run type-check   # TypeScript 类型检查
```

### CI/CD 命令

顶层 Makefile 提供智能的多语言 CI/CD 工具链：

```bash
make setup       # 一次性环境初始化
make format      # 格式化代码
make lint        # 代码质量检查
make test        # 运行测试
make build       # 构建项目
make ci          # 完整流水线：format + check + test + build
make clean       # 清理构建产物
```

## 核心模块说明

### Console Backend

Java 21 Spring Boot 多模块架构：

| 模块 | 说明 |
|------|------|
| `commons` | 共享 DTO、实体类、工具类、通用注解和切面 |
| `hub` | REST API 控制器、业务服务、MyBatis Mapper、安全配置 |
| `toolkit` | WebSocket、SSE 长连接、定时任务、文件处理 |

关键依赖：Spring Security + OAuth2、MyBatis Plus、Lombok + MapStruct、OkHttp、Fastjson2、Redisson、EasyExcel、MinIO。

### Java Workflow Engine

基于 DSL 的可视化工作流引擎：

- 节点图执行：开始 → LLM → 工具 → 知识库 → 条件分支 → 结束
- SSE 实时回调：流式推送节点执行状态
- 变量池管理：节点间数据传递和上下文维护
- Spring AI 集成：统一的模型调用接口

## 许可证

本项目基于 [Apache License 2.0](LICENSE) 开源。
