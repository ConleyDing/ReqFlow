# ReqFlow AI

> AI需求分析与测试用例生成工作台（30天双人团队课程/工程项目）
>
> **项目状态：规划基线 / 尚未开始正式开发。** 本仓库中的需求、架构、接口和测试内容是开发依据，不代表功能已经实现或通过测试。

## 1. 项目简介

ReqFlow AI 面向软件工程学生、初级产品人员和小型团队中的个人使用者，将自然语言软件需求转化为结构化需求、待确认问题、验收标准和测试用例。用户可以检查、修改、确认AI输出，并导出Markdown需求文档和CSV测试用例。

核心业务闭环：

1. 创建项目并输入自然语言需求。
2. AI分析需求，提取功能要求、约束、边界条件和待确认问题。
3. 用户审阅、修改并确认需求；未确定的信息保持为问题，不得被AI擅自补成事实。
4. 基于已确认需求生成测试用例。
5. 用户编辑、保存并导出结果。

## 2. 核心范围（MVP）

- 项目创建、列表、详情、编辑和删除。
- 自然语言需求输入与AI分析。
- 结构化需求、待确认问题、验收标准的编辑和确认。
- 基于已确认需求生成测试用例。
- 测试用例编辑、保存。
- Markdown需求文档与CSV测试用例导出。
- 健康检查、错误提示、静态示例需求输入和Docker Compose部署。

本项目不做账号/多租户、实时协同、知识库RAG、AI自动写代码、自动执行测试、语音和复杂文档解析。

## 3. 技术栈

| 层次 | 技术 | 说明 |
|---|---|---|
| Web前端 | Vue 3 + TypeScript + Vite | 单页应用 |
| UI | Element Plus | 表单、表格、弹窗和反馈 |
| Web入口 | Nginx | 静态资源托管、`/api`反向代理 |
| 后端 | Python + FastAPI | REST API与业务逻辑 |
| 数据访问 | SQLAlchemy 2.x + psycopg 3 | 同步模式，降低两人团队复杂度 |
| 数据库迁移 | Alembic | 版本化Schema迁移 |
| 数据库 | **PostgreSQL 17（容器化）** | 唯一正式数据库；不使用SQLite |
| AI | 可配置的大模型API适配器 | 不在本项目训练模型 |
| 部署 | **Docker + Docker Compose** | 三个核心服务一键启动 |
| 测试 | pytest、Vitest（候选，Day 7 技术验证后确定）、人工端到端验收 | 以核心闭环为重点 |

依赖版本 Day 2 记录可用版本，Day 2-7 兼容性测试后锁定版本并提交 lock 文件；PostgreSQL正式Release前锁定已验证的镜像Digest，不将浮动标签作为最终发布依据。

## 4. 一键部署目标

正式发布时，用户在已安装 Docker Engine/Desktop 和 Docker Compose v2 的环境中：

```bash
# 1. 获取代码
git clone <实际仓库地址>
cd ReqFlow-AI  # 目录名以实际克隆结果为准

# 2. 复制配置并修改密码、模型服务信息
cp deploy/.env.example deploy/.env

# 3. 启动
docker compose --env-file deploy/.env -f deploy/docker-compose.yml up -d --build

# 4. 检查
docker compose --env-file deploy/.env -f deploy/docker-compose.yml ps
```

浏览器访问 `http://127.0.0.1:8080`（端口可配置）。后端每次启动前执行`alembic upgrade head`：首次部署创建初始表结构，后续启动检查并应用尚未执行的迁移。PostgreSQL数据保存在命名卷中。Compose会检查db、backend和web三项健康状态。停止服务使用 `down`，**不要随意使用 `down -v`，该命令会删除数据库卷**。

> 注意：这是本机本地的单用户MVP，默认仅绑定本机回环地址。未实现账号认证前，不得将服务直接暴露到公网。

## 5. 当前交付状态（重要）

本规划文档包当前包含文档、Compose配置草案、Nginx配置和环境变量示例；**尚不包含**实际`frontend/`、`backend/`、`tests/`源代码以及`frontend/Dockerfile`、`backend/Dockerfile`。因此，当前压缩包不能直接完成`up --build`并运行完整产品。待开发阶段创建缺失源码和Dockerfile、并完成实际部署验证后，才可称为可运行交付包。详见[对抗性审查报告](AUDIT_REPORT.md)及[交付验收清单](docs/20-deliverables-and-acceptance.md)。

## 6. 仓库目录

```text
ReqFlow-AI/
├── frontend/                 # Vue 3前端（开发阶段创建）
├── backend/                  # FastAPI后端（开发阶段创建）
├── deploy/
│   ├── docker-compose.yml    # 容器编排基线
│   ├── .env.example          # 非敏感配置示例
│   └── nginx.conf            # Nginx反向代理配置
├── docs/                     # 完整工程文档
├── templates/                # 任务、评审、测试记录模板
├── tests/                    # 自动化测试（开发阶段创建）
├── tools/                    # 一致性检查脚本与结果清单（开发阶段创建，OPEN-10）
├── artifacts/                # 交付成果——PPT/视频/报告，开发阶段产生
├── .gitignore
├── .dockerignore             # Docker构建上下文排除规则
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── CODE_OF_CONDUCT.md
├── CHANGELOG.md
├── THIRD-PARTY-NOTICES.md    # 第三方依赖许可证清单
├── AUDIT_REPORT.md           # 对抗性审查报告
└── MASTER_PROJECT_DOCUMENT.md # 完整规划总文档（docs/镜像）
```

## 7. 文档导航

- [对抗性审查报告与未关闭行动项](AUDIT_REPORT.md)
- [完整规划总文档](MASTER_PROJECT_DOCUMENT.md)
- `docs/`中的独立文档是维护主源，总文档由其汇总生成。

- [项目章程](docs/00-project-charter.md)
- [选型报告](docs/01-project-selection.md)
- [范围与WBS](docs/02-scope-and-wbs.md)
- [软件过程与项目管理计划](docs/03-process-and-management-plan.md)
- [30天实施计划](docs/04-30-day-plan.md)
- [产品需求规格说明书（PRD）](docs/05-prd.md)
- [AI专项设计](docs/06-ai-design.md)
- [系统总体与详细设计](docs/07-system-design.md)
- [数据库设计](docs/08-database-design.md)
- [API接口设计](docs/09-api-design.md)
- [UI/UX设计](docs/10-ui-ux-design.md)
- [测试计划与测试用例](docs/11-test-plan-and-cases.md)
- [部署手册](docs/12-deployment-manual.md)
- [用户手册](docs/13-user-manual.md)
- [维护说明书](docs/14-maintenance-manual.md)
- [开源发布计划](docs/15-open-source-release.md)
- [汇报PPT脚本](docs/16-presentation-storyboard.md)
- [系统演示视频脚本](docs/17-demo-video-script.md)
- [项目总结报告模板](docs/18-project-summary-template.md)
- [风险登记册](docs/19-risk-register.md)
- [交付物及最终验收清单](docs/20-deliverables-and-acceptance.md)
- [需求追踪矩阵](docs/21-requirements-traceability.md)
- [架构决策记录](docs/22-architecture-decisions.md)
- [AI辅助开发规范](docs/23-ai-assisted-development-standard.md)
- [用例规格说明](docs/24-use-cases.md)
- [技术依据与官方参考](docs/25-technical-references.md)

## 8. 项目基本约束

- 团队：2人，成员A负责前端与产品交互，成员B负责后端、AI和数据库；双方共同负责评审、集成、测试和最终交付。
- 周期：30个自然日，计划日期为2026-11-11至2026-12-10。
- 数据库：必须使用PostgreSQL；不使用SQLite。
- 部署：必须提供Dockerfile、Docker Compose、健康检查、持久化卷和干净环境部署验证。
- AI：真实调用配置的大模型；模拟数据只能作为明确标记的界面演示/测试夹具，不能冒充真实模型输出。
- 文档状态：所有“实际结果”须在实施后据实填写。

## 9. 项目状态与贡献

当前为规划文档基线。功能是否完成、测试是否通过、仓库是否发布，以开发过程中的证据为准。贡献方式见 [CONTRIBUTING.md](CONTRIBUTING.md)。
