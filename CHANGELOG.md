# Changelog

## [0.2.3] - 2026-11-11

- 第四轮对抗性审查：统一用例关联不变量的时点化口径——“每条测试用例关联且仅关联一条需求（单一requirement_id）；生成与保存新用例时，其关联需求状态必须为CONFIRMED；既有用例的查看、编辑与导出不受该时点约束”（`docs/05`、`docs/08`）。
- 用例生命周期与需求编辑联动裁定：CONFIRMED需求或其验收标准被PATCH编辑后，需求状态回到DRAFT；既有用例保留——可查看、编辑、导出，不自动删除；重新生成用例时在同一生成事务内先删除所选需求名下的旧用例再写入新用例（按需求替换，与全有或全无一致，生成失败不删旧用例）；`docs/06`、`docs/08`、`docs/09`、`docs/10`、`docs/13`、`docs/24`全链同步。
- 新增TC-035“需求编辑状态联动与用例替换”（FR-006/FR-010，P0），`docs/20`验收行与`docs/21`追踪矩阵FR-006/FR-010证据同步。
- 生成预检统一：拒绝情形恰为三种——没有已确认需求、所选需求ID不属于当前项目、所选需求ID状态不为CONFIRMED——均返回422 NO_CONFIRMED_REQUIREMENTS，不调用模型、不创建运行记录，消除“静默过滤未确认ID”读法（`docs/06`与`docs/09`完全一致）。
- 新增AI错误码映射表（`docs/09`）：503 AI_NOT_CONFIGURED、502 AI_PROVIDER_UNREACHABLE、502 AI_PROVIDER_AUTH_FAILED、429 AI_RATE_LIMITED、502 AI_PROVIDER_ERROR（上游5xx重试后仍失败）、504 AI_PROVIDER_TIMEOUT、502 AI_INVALID_OUTPUT（修复重试后仍失败）、422 INPUT_TOO_LARGE（不创建运行记录），及既有422 NO_CONFIRMED_REQUIREMENTS、422 INSUFFICIENT_REQUIREMENT_DETAIL；`docs/06`异常表补“错误码”列，`docs/07`状态码清单补502/503/504；两类运行记录FAILED时的error_code取值即本表。
- 错误响应统一包装扩展为`error:{code, message, request_id, details(可选数组)}`；模型INSUFFICIENT_REQUIREMENT_DETAIL信号中details的临时ID（R1/R2）由后端映射为对应requirement_code与缺失信息摘要后放入响应details；`docs/06`补“临时ID仅用于模型-后端之间，对外响应由后端映射为业务编号”。
- 生成接口响应结构补全：POST `/projects/{project_id}/test-generation-runs`同步成功返回201，响应体为运行记录对象（含status=SUCCEEDED、provider/model/prompt_version等）内嵌items=本次生成并已保存的测试用例数组，与`docs/07`时序“201返回运行记录（内嵌items用例列表）”核对一致。
- 新增澄清问题/分析运行获取契约：GET `/projects/{project_id}/analysis-runs`（分页列表）与GET `/analysis-runs/{run_id}`（单资源直返）响应结构，`docs/09`§7数据模型补AnalysisRun；questions数组反映最新状态与答案而非result_json快照，与`docs/10`页面C刷新后持续提醒的数据来源对齐。
- 依赖锁定执行锚点：`docs/04` Day 2当日交付补“记录语言/运行时/数据库及依赖可用版本（docs/25）”、Day 7当日交付补“前后端依赖锁定并提交lock文件”；`docs/20`源代码/部署包相关行验收证据补“lock文件已提交”。
- `README`§6与`docs/20`§3目录树各补tools/条目（一致性检查脚本与结果清单，开发阶段创建，OPEN-10）；`docs/15`发布步骤7扩写——`ReqFlow_AI_项目汇报.pptx`与`ReqFlow_AI_系统操作演示.mp4`以Release附件发布，与`docs/20`指针闭合。
- 规划口径修正：`docs/02` Day 20规则去绝对化（既定的测试、文档与发布任务除外；启用任何P1须按范围变更规则经两名成员评审并记录）；`docs/20`操作视频计划完成日改为“Day 28初版，Day 29定稿”、A-03行“超时”口径与所引TC实际覆盖对齐；`docs/13`导出说明补“CSV为UTF-8 with BOM编码，Excel直接打开中文不乱码”；`docs/14`§3升级流程补backend镜像更新后应同时重启web服务（或执行nginx -s reload）以刷新Nginx对上游容器地址的解析；`docs/12`§11清单开头补compose命令按§5完整形式执行的说明。
- 测试与手册修正：`docs/11`§11验收标准改为“演示/测试夹具数据明确标记，不冒充真实模型输出”、TC-030优先级P1→P0（NFR-010性能实测唯一承载）、TC-019补全compose命令形态；`docs/10`§6字符上限提示补来源说明（默认12000字符，后端AI_MAX_INPUT_CHARS配置，最终以后端校验为准）；`docs/06`§4.1需求分析示例summary与数组条目自洽（遵循AUD-24“不得虚构”原则）；`docs/24` UC-04补齐FR-012（Markdown导出）流程覆盖归属并按用例替换语义补后置条件；`docs/README`索引与`templates/README`模板用途同步（风险登记册三列动态更新机制、bug-report与bug-fix-task模板拆分）。
- 根文件修正：`.dockerignore`补排除`*.sql`、`*.dump`、`*.log`、`backups/`（与.gitignore防护面对齐）；`THIRD-PARTY-NOTICES.md` psycopg兼容性结论措辞修正（以未修改的独立库形式经SQLAlchemy dialect调用，不构成对该LGPL库的衍生作品，与MIT许可证兼容，发布时随清单附LGPL声明）；`docs/15`§3必备文件清单补`.dockerignore`。
- `AUDIT_REPORT.md`审查版本升级为0.2.3（第四轮对抗性修订版）、审查日期补第四轮修订，§3.3健康检查路径、§6同步义务与冻结结论、AUD-24行注、§7两处措辞修正（三列、细节指针）并追加“第四轮复核修订（0.2.3）”记录；本CHANGELOG [0.2.2]补记第三轮遗漏四条（见下）。
- 复核残留修复：`docs/09`§4与§7的AnalysisRun字段集统一（§7补requirements_count/questions_count/open_questions_count派生字段，§4详情契约补project_id与观测字段）、§7新增TestGenerationRun数据模型、分析POST标明成功201、§1直返约定补“运行记录类单资源可内嵌items明细”说明、summary补“由result_json投影对外返回”说明；不变量句限定为“生成与保存新用例时”并明示既有用例不受时点约束（`docs/05`、`docs/08`）；TC-031预期结果补“所属需求状态回到DRAFT”状态联动；`docs/07`预检分支标签明确为“三种拒绝情形之一”；OPEN-07范围扩至TC-035；`AUDIT_REPORT.md`§7标题改为“修订记录（0.2.2 / 0.2.3）”并增设第三轮小节标题。

## [0.2.2] - 2026-11-11

- 第三轮对抗性审查：补全AI契约——新增模型侧“信息不足”错误信号（`error.code=INSUFFICIENT_REQUIREMENT_DETAIL`，与`test_cases`互斥，由后端转换为HTTP 422），补齐`acceptance_criteria`元素结构与`sort_order`映射约定。
- 统一重试预算口径：单次API请求内模型调用总计最多2次（首次调用+1次重试），`AI_MAX_RETRIES=1`为5xx网络退避与JSON结构修复的合并预算，不叠加。
- 明确需求编辑与确认状态语义：编辑CONFIRMED需求后自动回到DRAFT，重复confirm为幂等成功（200）；移除超出需求基线的`DELETE /test-cases/{test_case_id}`孤立端点。
- `docs/08` `test_generation_runs`新增`latency_ms`/`input_tokens`/`output_tokens`三列，ER图新增`PROJECTS`→`CLARIFICATION_QUESTIONS`关系边。
- 备份恢复命令修正为`pg_dump --clean --if-exists`与`psql -v ON_ERROR_STOP=1`；仓库根新增`.dockerignore`与`THIRD-PARTY-NOTICES.md`（工作区文件总数49→51）；CSV导出编码定死UTF-8 with BOM。
- 新增NFR-011健康检查与TC-034分页用例并同步追踪矩阵；维护/演示等手册补计划态声明；统一依赖锁定与Day 25干净环境部署验证口径；前端测试框架决策点统一为Day 7（候选Vitest）。
- 诚信措辞修正：脚本化一致性检查降级为“人工静态核对辅以脚本（102项）”，新增OPEN-10要求检查脚本与结果清单随仓库提交。
- 交付物口径统一：`docs/20`交付物矩阵文件名对齐交付清单（`ReqFlow_AI_项目汇报.pptx`、`ReqFlow_AI_系统操作演示.mp4`、`project-summary.md`），最终验收表补充A-01a需求输入校验（TC-005、TC-027）与A-05a测试用例编辑（TC-016）。
- FR-014示例需求输入补齐排期与用例：Day 10交付前端静态示例文本一键填入/清空（标注示例输入，未提交前不写库），UC-02主流程、备选流程与关联需求同步覆盖。
- `docs/02`统一P1范围规则口径，并补WBS 6.2交付文档工作包。
- 风险登记册新增等级、状态、更新日期三列，等级按概率×影响综合判定；R-07、R-14应对策略对齐镜像Digest锁定与psycopg 3许可证结论。
- 消除UC编号二义：`docs/24`确立为用例规格唯一主源，PRD §12改为用例摘要与指针；UI/UX新增“设置/模型配置说明页”（页面E）并与PRD页面清单命名统一，需求输入区补敏感信息警示文案与需求摘要展示位置，需求条目补业务编号`requirement_code`。
- 开源发布合规：LICENSE移除MIT标准文本之外的批注行；`artifacts/`取消gitignore以纳入交付成果；`docs/15`发布前检查补LICENSE定稿统一措辞、CODE_OF_CONDUCT联系渠道与THIRD-PARTY-NOTICES.md复核项，`.env.example`路径修正为`deploy/.env.example`；`CONTRIBUTING.md`新增§6贡献授权条款。
- 文档与模板修复：`docs/README`索引表补齐25行中文“主要用途”说明；`templates/bug-report-template.md`重写为缺陷报告模板，与`bug-fix-task-template.md`职责拆分；`docs/03`取消AI-xxx编号并补迭代回顾、功能任务单与缺陷处理机制；`docs/25`参考表新增Pydantic、psycopg 3、Element Plus、pytest四行并补“提交lock文件”口径；`docs/16`汇报时长算术自洽修正；`docs/17`补录制计划态声明。
- 按上述修订重新生成`MASTER_PROJECT_DOCUMENT.md`，docs/00~25全部26个镜像段落与分文档逐字一致。

## [0.2.1] - 2026-11-10

- 第二轮对抗性审查：明确澄清问题答复不自动回写需求，统一无AI配置时的页面能力边界。
- 修订AI结构化输出示例，避免示例中出现未经确认的业务规则。
- 将30天计划基线调整为2026-11-11至2026-12-10，并修正甘特图与每日任务/里程碑的日期对齐。
- 明确澄清问题答复、未配置AI的功能边界和测试用例生成全有或全无规则；补齐生成失败运行记录生命周期。
- 对齐AI、Nginx和前端超时预算；修正数据库业务编号并发分配约定。
- 使用PyYAML完成Compose文件YAML语法静态解析，人工静态核对辅以脚本（102项，脚本将于开发阶段随仓库提交）；Docker Compose运行时验证仍待执行。

## [0.2.0] - 2026-11-10

- 完成规划文档对抗性审查并统一健康检查路径、API字段命名和成功响应规则。
- 补齐web容器健康检查，明确PostgreSQL镜像Digest发布锁定要求。
- 明确示例需求仅为前端静态填充，不创建项目或数据库记录。
- 补充FR/NFR追踪、非功能测试用例和当前交付边界。
- 重新生成主规划文档，避免主文档与独立文档漂移。

## [0.1.0] - 2026-11-09

- 初始项目规划和工程文档包。
- 固定PostgreSQL和Docker Compose三容器部署方向。

> Changelog记录文档和规划变更，不表示产品代码或功能已经实现。
