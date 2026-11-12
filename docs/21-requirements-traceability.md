# 21 需求追踪矩阵

本矩阵覆盖PRD全部FR-001至FR-017及NFR-001至NFR-011。P1/P2需求标记为延期，不属于30天MVP验收；其余P0需求必须有测试或评审证据。UC 引用均指向 docs/24（用例规格唯一主源）。

## 1. 功能需求追踪

| 需求ID | 需求描述 | 优先级/基线 | 设计章节 | API/数据 | 测试/验收 | 证据 |
|---|---|---|---|---|---|---|
| FR-001 | 创建项目 | P0 | PRD 6、UC-01、UI 3 | POST /api/v1/projects；projects | TC-001/002 | 项目创建截图/API测试 |
| FR-002 | 项目列表 | P0 | PRD 6、UI 3 | GET /api/v1/projects；projects | TC-001/034 | 列表与分页测试 |
| FR-003 | 项目详情/编辑/删除 | P0 | PRD 6、UC-01、DB 3.1 | GET/PATCH/DELETE /api/v1/projects/{id} | TC-003/004 | CRUD及级联删除测试 |
| FR-004 | 需求输入 | P0 | PRD 6、UC-02、AI 3 | POST /api/v1/projects/{id}/analysis-runs；analysis_runs.source_text | TC-005/027 | 输入校验、边界测试 |
| FR-005 | AI需求分析 | P0 | PRD 6、AI 3-5、系统设计 7.1 | analysis-runs/requirements/questions/criteria | TC-006至009 | 真实调用、失败记录、AI评估 |
| FR-006 | 编辑需求 | P0 | PRD 6、UI 3 | PATCH /api/v1/requirements/{id}；requirements | TC-010/035 | 修改后持久化测试；CONFIRMED需求编辑后状态回DRAFT、既有用例保留 |
| FR-007 | 处理待确认问题 | P0 | PRD 6、AI 3、UC-03 | PATCH /api/v1/clarification-questions/{id} | TC-011/012 | 状态和回答持久化 |
| FR-008 | 编辑验收标准 | P0 | PRD 6、DB 3.5 | PATCH /api/v1/acceptance-criteria/{id} | TC-031 | 字段和关联测试 |
| FR-009 | 确认需求 | P0 | PRD 7、UC-03 | POST /api/v1/requirements/{id}/confirm | TC-013 | 状态与前置规则测试 |
| FR-010 | AI测试用例生成 | P0 | PRD 6、AI 3、系统设计 7.2 | POST /api/v1/projects/{id}/test-generation-runs；test_cases | TC-014/015/032/033/035 | 真实模型调用、来源关联、信息不足/模型失败时全量拒绝并记录失败运行；重新生成按需求替换旧用例 |
| FR-011 | 编辑测试用例 | P0 | PRD 6、DB 3.7 | PATCH /api/v1/test-cases/{id} | TC-016 | 编辑持久化测试 |
| FR-012 | Markdown导出 | P0 | PRD 6、API 6 | GET /api/v1/projects/{id}/exports/requirements.md | TC-017 | 文件内容和编码 |
| FR-013 | CSV导出 | P0 | PRD 6、API 6 | GET /api/v1/projects/{id}/exports/test-cases.csv | TC-018 | CSV内容、转义及编码 |
| FR-014 | 示例需求输入 | P0 | PRD 6、UC-02、UI 3、用户手册 3、7 | 前端静态样例，不调用API、不写DB | TC-025 | 输入框截图、未提交前无DB写入证明 |
| FR-015 | 测试用例质量建议 | P1，延期 | PRD 6 | 不纳入MVP | 不测试；范围冻结后不实现 | 若未来实现再增测试 |
| FR-016 | 需求模板 | P1，延期 | PRD 6 | 不纳入MVP | 不测试；范围冻结后不实现 | 若未来实现再增测试 |
| FR-017 | PDF导出 | P2，不纳入 | PRD 6 | 不纳入MVP | 不测试 | 后续版本评估 |

## 2. 非功能需求追踪

| 需求ID | 需求描述 | 设计/实现依据 | 测试/评审 | 证据 |
|---|---|---|---|---|
| NFR-001 | 容器化易部署 | 系统设计、Compose、部署手册 | TC-021 | 干净环境部署日志和截图 |
| NFR-002 | PostgreSQL持久化 | DB设计、postgres_data命名卷 | TC-019/020 | down/up和容器重建前后数据比对 |
| NFR-003 | 加载/空/成功/失败/超时状态 | PRD、UI/UX | TC-026 | 浏览器状态截图/录屏 |
| NFR-004 | 安全（本地绑定、后端密钥、DB不暴露） | ADR-006、系统设计、安全说明 | TC-023/024 + 密钥扫描 | 扫描报告、端口检查 |
| NFR-005 | 输入最大12000字符 | PRD、AI设计、Compose配置 | TC-027 | 前后端拒绝及模型未被调用的记录 |
| NFR-006 | 可维护性（分层、类型、迁移、日志） | 系统设计、AI开发规范 | 代码审查清单 | PR记录、迁移审查、代码审查结论 |
| NFR-007 | Chrome/Edge桌面兼容 | PRD、UI/UX | TC-028 | 浏览器版本与冒烟测试记录 |
| NFR-008 | 需求/测试及AI运行可追踪 | DB设计、AI设计、API | TC-029 | provider/model/prompt_version和requirement_id记录 |
| NFR-009 | 数据备份恢复 | DB设计、部署手册 | TC-022 | pg_dump及测试实例恢复记录 |
| NFR-010 | 非AI列表/详情P95目标 | PRD 9 | TC-030 | 固定测试环境的P50/P95记录；目标未达标如实说明 |
| NFR-011 | 健康检查（db/backend/web三级） | 系统设计 3、部署手册 6 | 部署验证/审查（TC-021） | 部署验证记录（三容器健康状态与端点判定输出） |

## 3. 变更规则

实施中如增删需求，必须同步更新PRD、接口/数据设计、测试用例和本矩阵。P1/P2不允许悄悄转成P0；任何范围调整须经两名成员评审并记录决定。
