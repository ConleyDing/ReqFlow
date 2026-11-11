# 09 API接口设计说明书

**Base Path：** `/api/v1`。除健康检查表中明确写出的完整路径外，业务接口表中的路径均为相对路径；实际请求须在路径前拼接`/api/v1`。  
**格式：** JSON；导出接口返回文件流。  
**认证：** MVP单用户本机本地部署，无用户登录；因此不得直接暴露公网。

## 1. 通用约定

- 主键使用UUID字符串。
- 时间使用ISO 8601 UTC格式。
- 成功响应统一采用资源直返：单个资源返回资源对象；列表返回`{ "items": [], "total": 0, "page": 1, "page_size": 20 }`；无内容操作返回HTTP 204。不得在部分接口额外包裹`data`。
- 错误响应统一采用`{"error":{"code":"...","message":"...","request_id":"..."}}`。
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
| PATCH | `/requirements/{requirement_id}` | 修改需求 |
| POST | `/requirements/{requirement_id}/confirm` | 确认需求 |
| PATCH | `/clarification-questions/{question_id}` | 回答或忽略问题；回答只更新问题记录，不自动改写需求 |
| PATCH | `/acceptance-criteria/{criterion_id}` | 修改验收标准 |

触发分析请求：

```json
{"source_text":"用户可以使用手机号注册账号，并通过手机号和密码登录。忘记密码后可以重新设置密码。"}
```

成功响应关键结构：

```json
{
  "run_id":"<uuid>",
  "status":"SUCCEEDED",
  "summary":"...",
  "requirements":[{"id":"<uuid>","requirement_code":"REQ-001","title":"手机号注册","status":"DRAFT"}],
  "questions":[{"id":"<uuid>","question":"注册是否需要验证码？","reason":"原文未说明手机号格式、验证方式和账号唯一性规则","status":"OPEN","answer":null}]
}
```

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
| DELETE | `/test-cases/{test_case_id}` | 删除用例 |

生成请求：

```json
{"requirement_ids":["<confirmed-requirement-uuid>"]}
```

若没有已确认需求，或所选需求ID不属于当前项目/状态不为CONFIRMED，预检阶段返回422和`NO_CONFIRMED_REQUIREMENTS`，不调用模型、不创建生成运行记录。通过预检后先创建`RUNNING`的`test_generation_runs`记录。若模型判断任何一条所选需求仍缺少明确预期结果所需的信息，返回422和`INSUFFICIENT_REQUIREMENT_DETAIL`，将该运行记录置为`FAILED`并保存脱敏错误；本次生成全量失败且不保存部分用例。模型调用/结构校验失败同样将运行记录置为`FAILED`；成功时在同一事务中保存全部用例并将运行记录置为`SUCCEEDED`。

## 6. 导出接口

| 方法 | 路径 | Content-Type | 文件 |
|---|---|---|---|
| GET | `/projects/{project_id}/exports/requirements.md` | `text/markdown; charset=utf-8` | 需求文档 |
| GET | `/projects/{project_id}/exports/test-cases.csv` | `text/csv; charset=utf-8` | 测试用例表 |

## 7. 主要数据模型

### Requirement

`id, project_id, requirement_code, title, description, requirement_type, priority, status, source_excerpt, acceptance_criteria[]`

### ClarificationQuestion

`id, question, reason, status, answer`

### TestCase

`id, requirement_id, case_code, title, test_type, priority, preconditions[], steps[], expected_result`

## 8. 同步调用与超时

MVP不引入后台任务队列。AI请求由API同步等待，前端显示加载状态并禁用重复点击；模型单次HTTP请求超时默认60秒，最多允许一次受控重试；Nginx读取超时设置为150秒，为两次调用及有限退避预留时间；前端AI请求超时建议设置为160秒，普通CRUD请求可使用较短超时。若调整重试次数或单次超时，必须同步调整Nginx与前端超时设置。调用前创建RUNNING运行记录，调用完成后更新为SUCCEEDED或FAILED。若实际服务时延经测试经常超过该值，再评估异步任务，不在首版提前引入Redis/Celery。

## 9. 幂等与并发

- 前端在请求中禁用重复提交。
- 生成接口可接收可选`Idempotency-Key`；如实现成本过高，至少以请求状态和按钮禁用避免双击。
- 更新接口使用`updated_at`或版本字段防止静默覆盖；MVP可先采用最后写入生效，并在UI提示保存成功。
- 生成用例时校验所有需求ID存在、属于当前项目且为CONFIRMED。

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
