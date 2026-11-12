# 09 API接口设计说明书

**Base Path：** `/api/v1`。除健康检查表中明确写出的完整路径外，业务接口表中的路径均为相对路径；实际请求须在路径前拼接`/api/v1`。  
**格式：** JSON；导出接口返回文件流。  
**认证：** MVP单用户本机本地部署，无用户登录；因此不得直接暴露公网。

## 1. 通用约定

- 主键使用UUID字符串。
- 时间使用ISO 8601 UTC格式。
- 成功响应统一采用资源直返：单个资源返回资源对象（运行记录类单资源可内嵌`items`子资源数组，属内嵌明细而非分页信封，见§5生成接口）；列表返回`{ "items": [], "total": 0, "page": 1, "page_size": 20 }`；无内容操作返回HTTP 204。不得在部分接口额外包裹`data`。
- 错误响应统一采用`{"error":{"code":"...","message":"...","request_id":"...","details":[...]}}`；`details`为可选字符串数组，承载可供用户定位问题的业务细节，无细节时省略该字段。AI相关`code`取值见§8错误码映射表。
- 列表接口支持`page`和`page_size`，默认20，最大100。
- 请求体由Pydantic校验；未知字段默认拒绝或明确忽略，团队需统一配置。

## 2. 健康检查

| 方法 | 路径 | 用途 |
|---|---|---|
| GET | `/api/v1/health/live` | 进程存活，不依赖DB |
| GET | `/api/v1/health/ready` | 检查数据库连接并返回就绪状态；Compose后端健康检查使用此路径 |

## 3. 项目接口

| 方法 | 路径 | 用途 | 主要响应 |
|---|---|---|---|
| POST | `/projects` | 创建项目 | 项目对象 |
| GET | `/projects` | 分页项目列表 | items、total、page |
| GET | `/projects/{project_id}` | 项目详情 | 项目对象 |
| PATCH | `/projects/{project_id}` | 更新名称/描述 | 更新后项目 |
| DELETE | `/projects/{project_id}` | 删除项目及关联内容 | 204 |

创建项目请求示例：

```json
{"name":"校园报修系统","description":"用于整理报修需求和测试用例"}
```

## 4. 需求分析接口

| 方法 | 路径 | 用途 |
|---|---|---|
| POST | `/projects/{project_id}/analysis-runs` | 触发AI需求分析 |
| GET | `/projects/{project_id}/analysis-runs` | 查询分析历史 |
| GET | `/analysis-runs/{run_id}` | 获取分析运行详情 |
| GET | `/projects/{project_id}/requirements` | 获取需求列表 |
| PATCH | `/requirements/{requirement_id}` | 修改需求；编辑CONFIRMED需求后状态自动回到DRAFT |
| POST | `/requirements/{requirement_id}/confirm` | 确认需求；对已CONFIRMED需求重复调用为幂等成功（200） |
| PATCH | `/clarification-questions/{question_id}` | 回答或忽略问题；回答只更新问题记录，不自动改写需求 |
| PATCH | `/acceptance-criteria/{criterion_id}` | 修改验收标准；对CONFIRMED需求的验收标准执行编辑后，该需求状态同样回到DRAFT |

需求编辑与确认状态语义：`status`仅有DRAFT和CONFIRMED两种。对CONFIRMED需求执行PATCH编辑后，状态自动回到DRAFT，需重新确认；PATCH响应返回更新后的需求对象，其中`status`为`DRAFT`。对CONFIRMED需求的验收标准执行PATCH编辑，同样使该需求状态回到DRAFT（与需求编辑对称）。对已CONFIRMED需求重复调用confirm为幂等成功：返回200，状态保持CONFIRMED不变，不报错。需求由CONFIRMED回到DRAFT后，其已生成的测试用例保留——仍可查看、编辑、导出，不自动删除；重新确认并再次生成用例时，该需求名下旧用例在生成事务内被新用例替换（见§5）。

触发分析请求：

```json
{"source_text":"用户可以使用手机号注册账号，并通过手机号和密码登录。忘记密码后可以重新设置密码。"}
```

成功返回HTTP 201，响应体为本次分析运行对象（与`GET /analysis-runs/{run_id}`返回同一AnalysisRun结构，见§7），关键结构如下：

```json
{
  "id":"<uuid>",
  "status":"SUCCEEDED",
  "summary":"...",
  "requirements":[{"id":"<uuid>","requirement_code":"REQ-001","title":"手机号注册","status":"DRAFT"}],
  "questions":[{"id":"<uuid>","question":"注册是否需要验证码？","reason":"原文未说明手机号格式、验证方式和账号唯一性规则","status":"OPEN","answer":null}]
}
```

分析运行获取契约：

- `GET /projects/{project_id}/analysis-runs`：分页列表（结构见§1），`items`元素为运行摘要对象，字段`id, status, provider, model, prompt_version, requirements_count, questions_count, open_questions_count, error_code, created_at, finished_at`，按`created_at`倒序排列。
- `GET /analysis-runs/{run_id}`：单资源直返（无分页包裹），返回§7 AnalysisRun完整对象——在摘要对象字段基础上增加`project_id`、`error_message`、`latency_ms`、`input_tokens`、`output_tokens`，以及`source_text`（本次原始输入）、`summary`（分析摘要）、`requirements`（该次运行产出的需求摘要数组，元素含`id, requirement_code, title, status`）与`questions`（数组，元素为ClarificationQuestion结构，见§7）。`questions`与`requirements`反映数据库最新状态（含用户后续的回答与需求编辑），不是分析完成时`result_json`的快照。

页面C刷新后对未处理问题的持续提醒（`docs/05-prd.md`§8业务规则第6条）以列表中的`open_questions_count`与详情中的`questions`数组为数据来源。完整数据模型见§7 AnalysisRun。

问题答复请求示例：

```json
{"status":"RESOLVED","answer":"注册必须使用11位数字手机号；是否要求短信验证码仍需产品负责人确认。"}
```

忽略请求示例：`{"status":"IGNORED"}`。`RESOLVED`必须携带非空`answer`；`IGNORED`必须不传答案或将答案置空；`OPEN`只用于初始未处理状态，不接受用户直接设置。答复不会自动修改需求、验收标准或触发AI重分析。用户需手动编辑相关需求/标准后再确认。

## 5. 测试用例接口

| 方法 | 路径 | 用途 |
|---|---|---|
| POST | `/projects/{project_id}/test-generation-runs` | 基于已确认需求生成用例 |
| GET | `/projects/{project_id}/test-cases` | 查询用例列表 |
| PATCH | `/test-cases/{test_case_id}` | 编辑用例 |

生成请求：

```json
{"requirement_ids":["<confirmed-requirement-uuid>"]}
```

预检拒绝情形恰为三种——没有已确认需求／所选需求ID不属于当前项目／所选需求ID状态不为CONFIRMED——均返回422和`NO_CONFIRMED_REQUIREMENTS`，不调用模型、不创建生成运行记录；任一情形命中即整体拒绝，不得静默过滤未确认或不属于本项目的ID后继续生成。通过预检后先创建`RUNNING`的`test_generation_runs`记录。后端检测到模型输出的信息不足信号（`error.code=INSUFFICIENT_REQUIREMENT_DETAIL`，与`test_cases`互斥，模型侧输出契约见`docs/06-ai-design.md`§4.2）后，返回422和`INSUFFICIENT_REQUIREMENT_DETAIL`，将该运行记录置为`FAILED`并保存脱敏错误，同时将模型信号`details`中的需求临时ID（如R1/R2）映射为对应需求的`requirement_code`与缺失信息摘要后放入响应`error.details`数组（见§1），用户可据此定位需补充的需求；本次生成全量失败且不保存任何部分用例。模型调用/结构校验失败同样将运行记录置为`FAILED`；各失败情形均不删除既有旧用例。成功时在同一事务内先删除所选需求名下的既有旧用例，再写入本次生成的全部新用例，并将运行记录置为`SUCCEEDED`——按需求替换，本次未选中的需求其用例不受影响（需求被编辑回DRAFT后其既有用例保留，直至重新确认并再次生成时被替换，见§4）。

生成接口为同步接口：成功返回HTTP 201，响应体为本次运行记录对象（`status=SUCCEEDED`，含`provider`、`model`、`prompt_version`等），并内嵌`items`数组，即本次生成并已保存的测试用例，元素与§7 TestCase结构一致；失败按§8错误码映射表返回，不含`items`。成功响应示例：

```json
{
  "id": "<run-uuid>",
  "project_id": "<project-uuid>",
  "status": "SUCCEEDED",
  "provider": "...",
  "model": "...",
  "prompt_version": "...",
  "source_requirement_ids": ["<requirement-uuid>"],
  "error_code": null,
  "latency_ms": 8600,
  "input_tokens": 900,
  "output_tokens": 1600,
  "created_at": "2026-11-11T08:00:00Z",
  "finished_at": "2026-11-11T08:00:09Z",
  "items": [
    {"id":"<uuid>","requirement_id":"<requirement-uuid>","case_code":"CASE-001","title":"使用未注册的11位手机号完成注册","test_type":"positive","priority":"high","preconditions":["手机号为11位数字","该手机号尚未注册"],"steps":["打开注册页面","输入11位未注册手机号","提交注册"],"expected_result":"系统创建账号并显示注册成功提示。"}
  ]
}
```

## 6. 导出接口

| 方法 | 路径 | Content-Type | 文件 |
|---|---|---|---|
| GET | `/projects/{project_id}/exports/requirements.md` | `text/markdown; charset=utf-8` | 需求文档 |
| GET | `/projects/{project_id}/exports/test-cases.csv` | `text/csv; charset=utf-8` | 测试用例表；导出编码定死为UTF-8 with BOM，保证Excel直接打开中文不乱码 |

## 7. 主要数据模型

### Requirement

`id, project_id, requirement_code, title, description, requirement_type, priority, status, source_excerpt, acceptance_criteria[]`

`acceptance_criteria[]`元素含`id, criterion_text, sort_order`，与数据库`acceptance_criteria`表列对齐；AI输出侧的`acceptance_criteria`元素结构与`sort_order`映射规则见`docs/06-ai-design.md`§4.1。`status`取值DRAFT/CONFIRMED，编辑与确认的状态语义见§4。

### ClarificationQuestion

`id, question, reason, status, answer`

### AnalysisRun

`id, project_id, status, provider, model, prompt_version, source_text, summary, error_code, error_message, latency_ms, input_tokens, output_tokens, requirements_count, questions_count, open_questions_count, created_at, finished_at, requirements[], questions[]`

与数据库`analysis_runs`表对应（`result_json`仅供后端内部使用，不直接对外暴露；`summary`由后端从`result_json`投影对外返回）；`status`取值RUNNING/SUCCEEDED/FAILED，`error_code`取值见§8映射表。`requirements_count`、`questions_count`、`open_questions_count`为服务端按数据库最新数据计算的派生字段（`open_questions_count`为状态OPEN的问题数）。`requirements[]`为该次运行产出需求的摘要数组（`id, requirement_code, title, status`），`questions[]`为ClarificationQuestion数组；两者均反映数据库最新状态而非`result_json`快照。列表接口返回运行摘要对象、单资源接口直返完整对象，契约见§4。

### TestGenerationRun

`id, project_id, status, provider, model, prompt_version, source_requirement_ids[], error_code, error_message, latency_ms, input_tokens, output_tokens, created_at, finished_at`

与数据库`test_generation_runs`表对应（`result_json`仅供后端内部使用）；字段语义与AnalysisRun一致。生成成功响应（见§5）内嵌的`items`用例数组即校验后`result_json`的对外投影。`status`取值RUNNING/SUCCEEDED/FAILED，`error_code`取值见§8映射表。

### TestCase

`id, requirement_id, case_code, title, test_type, priority, preconditions[], steps[], expected_result`

## 8. 同步调用与超时

MVP不引入后台任务队列。AI请求由API同步等待，前端显示加载状态并禁用重复点击；模型单次HTTP请求超时默认60秒；单次API请求内模型调用总计最多2次（首次调用+1次重试），`AI_MAX_RETRIES=1`为5xx网络退避重试与JSON结构修复重试的合并预算——任一先发生即消耗唯一重试机会，不叠加；Nginx读取超时设置为150秒，为两次调用及有限退避预留时间；前端AI请求超时建议设置为160秒，普通CRUD请求可使用较短超时。若调整重试次数或单次超时，必须同步调整Nginx与前端超时设置。调用前创建RUNNING运行记录，调用完成后更新为SUCCEEDED或FAILED。若实际服务时延经测试经常超过该值，再评估异步任务，不在首版提前引入Redis/Celery。

**AI错误码映射表**（与`docs/06-ai-design.md`§8异常处理表保持一致）：

| 场景 | HTTP状态码 | `error_code` |
|---|---|---|
| 未配置AI仍调用 | 503 | `AI_NOT_CONFIGURED` |
| 网络/DNS失败 | 502 | `AI_PROVIDER_UNREACHABLE` |
| 上游返回401/403 | 502 | `AI_PROVIDER_AUTH_FAILED` |
| 上游返回429 | 429 | `AI_RATE_LIMITED` |
| 上游5xx（重试后仍失败） | 502 | `AI_PROVIDER_ERROR` |
| 请求超时 | 504 | `AI_PROVIDER_TIMEOUT` |
| JSON无效/输出截断（修复重试后仍失败） | 502 | `AI_INVALID_OUTPUT` |
| 生成输入超限 | 422 | `INPUT_TOO_LARGE`（预检拒绝，不创建运行记录；分析文本超长同码预检拒绝） |
| 预检失败：没有已确认需求／所选需求ID不属于当前项目／所选需求ID状态不为CONFIRMED | 422 | `NO_CONFIRMED_REQUIREMENTS`（不调用模型、不创建运行记录，见§5） |
| 模型输出信息不足信号 | 422 | `INSUFFICIENT_REQUIREMENT_DETAIL`（运行记录置为FAILED，见§5） |

数据库两类运行记录表（`analysis_runs`/`test_generation_runs`）的`error_code`列（VARCHAR(80)）取值即本表；运行记录状态为FAILED时写入对应错误码。

## 9. 幂等与并发

- 前端在请求中禁用重复提交。
- 生成接口可接收可选`Idempotency-Key`；如实现成本过高，至少以请求状态和按钮禁用避免双击。
- 更新接口使用`updated_at`或版本字段防止静默覆盖；MVP可先采用最后写入生效，并在UI提示保存成功。
- 生成用例时校验所有需求ID存在、属于当前项目且为CONFIRMED；任一不满足即整体拒绝（422 `NO_CONFIRMED_REQUIREMENTS`，见§5预检），不静默过滤后继续生成。

## 10. 配置与字段映射约定

- 后端数据库配置读取`DB_HOST`、`DB_PORT`、`POSTGRES_DB`、`POSTGRES_USER`、`POSTGRES_PASSWORD`；使用SQLAlchemy `URL.create("postgresql+psycopg", ...)`构造连接，不要求单独维护可能需要URL转义的`DATABASE_URL`。
- 需求资源字段固定使用`requirement_code`；业务测试用例资源字段固定使用`case_code`（格式`CASE-001`）。不得在API响应中缩写为`code`。测试计划中的`TC-001`仅为验证用例编号。
- 模型输出中的`temp_id`仅用于单次AI响应内关联；持久化时由后端映射到数据库UUID和业务编号。
- 项目示例内容由前端静态样例填充，不新增示例项目API或数据库表；未点击“分析”前不写入数据库。

## 11. 接口验收

- 所有成功路径和典型错误路径有接口测试。
- 删除项目后关联记录不可再查询。
- 数据库不可用时ready返回失败，live仍可返回成功。
- API Key、Prompt内部密钥和堆栈不出现在响应中。
