# ReqFlow AI 对抗性审查报告

**审查版本：** 0.2.1（第二轮对抗性修订版）  
**审查日期：** 2026-11-10  
**审查对象：** 0.1.0规划文档包中的README、总规划文档、26份编号文档、Compose/Nginx/环境变量草案及过程模板。  
**审查性质：** 文档内一致性与静态配置审查；不是产品代码测试或部署验收。

## 1. 审查结论

0.1.0版本不应直接冻结为开发基线。审查发现并修订了健康检查路径、API/数据库字段、容器健康检查、示例数据实现范围、AI运行状态、需求确认规则、评分算术、需求追踪和交付边界等问题。

0.2.1版已完成第二轮交叉修订，并将可由文档自身解决的冲突统一。**但不能据此声称“系统无缺陷”或“Compose已验证可运行”**：当前压缩包仍缺少前后端源码和两个Dockerfile；当前执行环境未安装Docker，因此没有执行`docker compose config`、镜像构建、容器启动、健康检查、PostgreSQL迁移或真实AI调用。

## 2. 问题清单与修订记录

| ID | 原问题 | 风险 | 修订结果 |
|---|---|---|---|
| AUD-01 | Compose检查`/api/v1/health/ready`，API文档却写无前缀路径 | Backend被误判为unhealthy | API文档明确所有路径均包含`/api/v1`，与Compose统一 |
| AUD-02 | API使用`code`，DB使用`requirement_code`/`case_code` | 前后端字段映射错误 | API样例和资源模型统一使用数据库业务字段名 |
| AUD-03 | 文档要求db/backend/web健康检查，Compose缺web healthcheck | 部署验收口径不一致 | web增加`wget /health`健康检查，系统设计补充三项检查 |
| AUD-04 | FR-014示例项目无表、接口和测试定义 | 范围不闭合 | 改为前端静态示例需求文本，填充输入框但不创建项目/写DB，新增TC-025 |
| AUD-05 | OPEN澄清问题是否阻止需求确认不明确 | UI与服务端可能采用不同规则 | 明确问题为analysis_run级；OPEN问题持续提醒，用户仍可显式确认已核验需求，确认不自动关闭问题 |
| AUD-06 | 会议纪要评分算术错误 | 排名表可信度下降 | 按既定权重修正为89.9 |
| AUD-07 | 追踪矩阵遗漏NFR-003/005/006/007/008/010及延期FR | 需求无法完整验收 | 重写矩阵，覆盖FR-001..017和NFR-001..010，列出测试或审查证据 |
| AUD-08 | API成功响应允许直返或包`data`两种实现 | 客户端契约不稳定 | 固定资源直返、列表分页对象、204无内容、错误统一包装 |
| AUD-09 | AI运行状态同时出现PENDING/RUNNING和RUNNING/SUCCEEDED/FAILED | 状态机不一致 | 统一创建时RUNNING，结束时SUCCEEDED/FAILED |
| AUD-10 | DB连接变量存在但连接构造方式未规定 | 密码特殊字符可能导致URL拼接错误 | 明确用SQLAlchemy `URL.create("postgresql+psycopg", ...)`构造连接 |
| AUD-11 | `postgres:17-alpine`浮动标签与发布锁定原则冲突 | 重建环境可能拉取不同补丁构建 | 改为可配置`POSTGRES_IMAGE`；开发默认17-alpine，Release前须锁定不可变Digest |
| AUD-12 | 原始需求持久化范围在AI说明中含糊 | 隐私告知不充分 | 明确存入`analysis_runs.source_text`，提醒不提交敏感信息 |
| AUD-13 | 规划包存在Compose但缺Dockerfile和应用源码 | 易被误认为已可运行 | README和验收文档明确当前不是可运行系统，缺失文件需开发产出 |
| AUD-14 | 总规划文档为独立重复内容，存在漂移风险 | 主文档和分文档可能不一致 | 将各独立文档作为主数据源，重新按编号生成总规划文档 |
| AUD-15 | `NEEDS_CLARIFICATION`未在AI输出Schema中定义，问题也没有requirement级外键 | 需求状态难以一致落库 | 需求状态统一为DRAFT/CONFIRMED；澄清问题保持analysis_run级别 |
| AUD-16 | 测试用例存在REVIEWED状态但没有对应产品操作 | DB/UI状态无法闭环 | MVP移除测试用例审核状态字段 |
| AUD-17 | FR-008缺少编号明确的测试用例，测试表新增行缺少表头 | 验收追踪不完整 | 新增TC-031并修正测试表结构 |
| AUD-18 | 无登录认证但部分文字允许可信内网场景 | 内网其他用户可无认证读写 | 限定为单用户本机本地场景，不承诺局域网共享 |
| AUD-19 | 后端Compose启动命令依赖工作目录和文件布局但Dockerfile尚未定义 | 容器启动时找不到alembic或app模块 | 在部署手册固定`/app`工作目录、alembic.ini和前端Nginx配置复制要求 |
| AUD-20 | 业务测试用例编号示例`TC-001`与测试计划的TC编号重叠 | 测试记录、数据库数据和演示编号混淆 | 业务数据编号改为`CASE-001`，测试计划编号保留`TC-001` |
| AUD-21 | 项目计划Day 1日期与审查时点重合，且甘特图跨度与逐日计划不完全一致 | 项目启动日不现实，里程碑日期可能错位 | 将执行基线调整为2026-11-11至2026-12-10；甘特图、逐日表和里程碑逐日对齐 |
| AUD-22 | 澄清问题回答是否自动改写需求/重跑AI未定义；部分文档表达容易造成自动合并预期 | 用户答案可能未进入已确认需求，测试用例使用旧规则 | 统一规定答案仅保存到问题记录；用户手动编辑需求/验收标准后确认；不自动重跑AI |
| AUD-23 | 未配置AI时“手工编辑”表述可能被理解为支持手动新建需求/用例 | UI超出API/PRD定义的创建能力 | 明确只能管理、查看和编辑已有记录；不能创建新的AI分析结果/AI生成用例 |
| AUD-24 | AI示例JSON中曾包含依据不足的验收标准；风险登记册仍使用事实/假设/问题三分输出说法 | 示例和正式Schema/Prompt边界不一致 | 示例改为无具体验收标准并输出澄清问题；MVP不提供独立假设字段，模型推断不得成为需求事实 |
| AUD-25 | 需求信息不足时是否允许部分生成测试用例未统一 | 可能出现部分数据保存、API返回失败但已有用例的歧义 | 统一采用全有或全无：返回`422 INSUFFICIENT_REQUIREMENT_DETAIL`，不保存任何部分用例，补充确认后重试 |
| AUD-26 | 测试用例生成运行记录在DB有RUNNING/FAILED状态，但AI流程/时序图只描述成功后保存运行记录 | 模型失败或需求信息不足时缺少可追踪运行记录 | 统一为预检通过后先建RUNNING记录；成功保存全部用例并标记SUCCEEDED；模型失败/信息不足时标记FAILED且不保存部分用例；预检失败不创建记录 |
| AUD-27 | AI单次请求60秒且允许一次重试，Nginx读取超时原为100秒 | 重试可能超过代理读取超时，产生不必要的504 | Nginx读取超时改为150秒，并要求未来调整模型超时/重试时同步复核代理和前端超时 |

## 3. 统一后的关键契约

### 3.1 技术架构

- 前端：Vue 3 + TypeScript + Vite + Element Plus。
- Web入口：Nginx，托管静态前端并代理`/api/`。
- 后端：FastAPI + SQLAlchemy 2.x同步Session + psycopg 3 + Pydantic。
- 数据库：PostgreSQL 17；不使用SQLite。
- 迁移：Alembic。
- 部署：Docker Compose三个服务：`web`、`backend`、`db`。
- PostgreSQL数据：命名卷`postgres_data`。
- AI：一个真实外部模型API；单次请求超时默认60秒、最多一次受控重试，Nginx读取超时150秒、前端AI请求建议160秒；MVP不自建模型、不使用RAG/队列/多智能体。

### 3.2 健康检查

- DB：`pg_isready -U $POSTGRES_USER -d $POSTGRES_DB`。
- Backend：`GET /api/v1/health/ready`，实际检查数据库连接。
- Web：容器内请求`http://127.0.0.1/health`。
- 启动顺序：backend等待db `service_healthy`；web等待backend `service_healthy`。

Docker Compose官方文档明确，`depends_on`的短语法只保证启动顺序，长语法的`condition: service_healthy`才会等待依赖健康检查通过。因此当前Compose采用长语法属于正确方向，必须保留健康检查实现。参考：[Docker Compose services](https://docs.docker.com/reference/compose-file/services/)、[startup order](https://docs.docker.com/compose/how-tos/startup-order/)。

### 3.3 API与DB字段

- 需求资源：`requirement_code`。
- 测试用例资源：`case_code`。
- 数据库访问配置：`DB_HOST`、`DB_PORT`、`POSTGRES_DB`、`POSTGRES_USER`、`POSTGRES_PASSWORD`；用SQLAlchemy URL对象构建连接。
- API成功响应：单资源直返；列表`items/total/page/page_size`；删除成功204；错误统一`error.code/message/request_id`。
- 健康检查完整路径均带`/api/v1`。

### 3.4 示例和确认语义

- 示例需求是前端静态文本，不是数据库示例项目。
- 点击加载示例只填充文本框；用户点击分析后才调用真实模型并保存运行记录。
- 澄清问题属于一次分析运行；回答仅保存到问题，不自动回写需求/标准或触发AI重分析；用户需手动更新并确认。保持OPEN时持续显示提醒。
- AI输出始终是草稿；只有用户明确确认的需求才能用于生成用例。测试用例生成先创建RUNNING运行记录，成功则全部保存并标记SUCCEEDED；模型失败或需求信息不足则标记FAILED且不保存部分用例。

## 4. 静态检查结果

| 检查项 | 结果 | 说明 |
|---|---|---|
| 压缩包文件清单 | 49个文件（增加审查报告） | 重新打包后执行ZIP完整性测试，包含主文档、分文档、审查报告、模板和部署草案 |
| 独立工程文档 | 27份 | `docs/00`至`docs/25`，另含索引 |
| Markdown相对链接 | 通过 | 审查时未发现失效的相对链接 |
| PostgreSQL/SQLite约束 | 通过 | 文档明确PostgreSQL唯一正式数据库，不使用SQLite |
| 日期一致性 | 通过 | 计划均为2026-11-11至2026-12-10，30个自然日 |
| 评分算术 | 已修订 | 前六个评分按公式重新核对，会议纪要项目修正89.9 |
| FR/NFR追踪 | 已补齐 | PRD全部FR/NFR均出现在追踪矩阵，FR-008有TC-031；FR-010包含信息不足全量拒绝TC-032 |
| YAML语法 | 通过（PyYAML safe_load） | 三服务均可解析且包含健康检查；未执行Docker Compose自身配置解析 |
| 脚本化静态一致性检查 | 通过（102/102） | 文件清单、相对链接、30天排期、里程碑日期、评分算术、FR/NFR/TC覆盖、状态枚举、Compose关键结构及数据库/接口约定检查 |
| Docker镜像构建 | 未执行 | 当前无Docker CLI，且源码/Dockerfile未交付 |
| PostgreSQL迁移/持久化 | 未执行 | 无应用源码和实际数据库容器 |
| 真实模型调用 | 未执行 | 需团队提供API服务和密钥后验证 |

## 5. 尚不能关闭的风险/行动项

| ID | 需实际完成的行动 | 责任建议 | 截止点 |
|---|---|---|---|
| OPEN-01 | 创建`backend/Dockerfile`、`frontend/Dockerfile`及源代码 | A/B | Day 7骨架，Day 25完整部署 |
| OPEN-02 | 安装Docker后运行`docker compose config`并修复配置问题 | B | Day 7 |
| OPEN-03 | 验证Nginx Alpine镜像中`wget`可用；否则调整Web健康检查命令 | B | Day 7 |
| OPEN-04 | 构建镜像后记录PostgreSQL 17的确切镜像Digest | B | Day 7锁定，Release再核对 |
| OPEN-05 | 按接口契约实现健康路由和成功响应模型 | B | API骨架阶段 |
| OPEN-06 | 按`requirement_code`/`case_code`完成前后端类型和DB映射 | A/B | Day 6契约评审，Day 9联调 |
| OPEN-07 | 为TC-025至TC-033补充自动化或人工记录模板 | A/B | Day 21-24 |
| OPEN-08 | 真实模型API调用、AI样本评分及失败样本记录 | A/B | Day 2、14、22 |
| OPEN-09 | 干净环境部署、数据持久化、备份恢复演练 | B主责，A复核 | Day 25 |

## 6. 审查限制与冻结声明

本次审查覆盖已交付文件的静态内容与可解析性，不包含实际业务代码，因此不能证明运行时“绝对无矛盾”。最终开发中，代码/OpenAPI/SQLAlchemy模型/Alembic迁移/Dockerfile/Compose必须作为一个整体再次审查。任何修改FR、NFR、API字段、数据库列、环境变量、容器名称或端口的PR，必须同步更新本矩阵及总规划文档。

**结论：0.2.1可作为经两轮文档交叉修订后的开发规划基线；不能作为已完成、已构建、已部署或已通过功能验收的软件版本。**
