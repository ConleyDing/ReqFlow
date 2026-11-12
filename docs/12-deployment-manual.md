# 12 部署手册（正式交付基线）

> 本文为目标部署方案。当前规划文档包尚不包含`frontend/`、`backend/`源码及两个Dockerfile，因此此刻不能直接执行成功的镜像构建。必须在这些文件实现后，由Day 7和Day 25分别验证Compose配置与干净环境部署；不得将文档/配置文件存在视为部署已通过。

## 1. 部署目标

用户只需准备Docker环境，复制环境变量示例并配置模型API，即可通过Docker Compose启动完整系统。应用由三个容器组成：Nginx+Vue、FastAPI、PostgreSQL。

## 2. 环境要求

- 支持Docker Engine/Desktop的64位操作系统。
- Docker Compose v2插件。
- 建议预留至少2GB可用内存和2GB磁盘空间（最终按镜像/数据实测调整）。
- 访问外部模型API所需的网络。
- 不要求宿主机安装Node.js、Python或PostgreSQL。

## 3. 配置文件

将`deploy/.env.example`复制为`deploy/.env`。至少修改：

- `POSTGRES_PASSWORD`：换成足够长的随机密码。
- `AI_BASE_URL`：模型服务API地址。
- `AI_API_KEY`：真实API密钥。
- `AI_MODEL`：服务商支持的模型名称。
- `APP_PORT`：Web访问端口。
- `POSTGRES_IMAGE`：开发默认`postgres:17-alpine`；正式Release前必须记录已验证的不可变镜像Digest，并将该Digest写入发布配置。

真实`.env`不得提交Git。若密码含特殊字符，应使用符合Compose环境变量解析规则的值；后端使用SQLAlchemy `URL.create("postgresql+psycopg", ...)`基于独立字段构造数据库连接，避免手工拼接URL时密码特殊字符编码错误。

## 4. 镜像构建约束（开发时必须遵守）

- `backend/Dockerfile`的工作目录必须固定为`/app`，并把后端代码、`alembic/`、`alembic.ini`和锁定的Python依赖安装到镜像中，保证Compose命令`alembic upgrade head && uvicorn app.main:app ...`在该目录可执行。
- `frontend/Dockerfile`采用多阶段构建：Node构建Vue静态资源，Nginx镜像负责运行；从仓库根构建上下文复制`deploy/nginx.conf`到`/etc/nginx/conf.d/default.conf`。
- 若Web健康检查使用`wget`，最终Nginx镜像必须提供该命令；否则Dockerfile需安装探针依赖或将健康检查调整为镜像内确实存在的命令。
- 构建上下文为仓库根；`.dockerignore`已排除`.git`、`deploy/.env`、`artifacts/`、`docs/`、`templates/`，防止真实密钥进入镜像构建上下文。
- 两个Dockerfile不得依赖宿主机预装Node.js、Python或PostgreSQL。

## 5. 启动

```bash
cp deploy/.env.example deploy/.env
# 编辑 deploy/.env 后执行：
docker compose --env-file deploy/.env -f deploy/docker-compose.yml config
docker compose --env-file deploy/.env -f deploy/docker-compose.yml up -d --build
```

启动顺序与迁移行为：

1. PostgreSQL启动并通过`pg_isready`健康检查。
2. FastAPI容器每次启动前执行`alembic upgrade head`；首次部署创建初始表结构，后续启动检查并应用尚未执行的迁移。
3. FastAPI就绪后，Nginx容器启动并提供静态页面及API代理。Compose同时通过Nginx容器内的`wget`请求`/health`检查Web服务健康状态。

访问：`http://127.0.0.1:8080`（以配置的APP_PORT为准）。

## 6. 状态与日志

```bash
docker compose --env-file deploy/.env -f deploy/docker-compose.yml ps
docker compose --env-file deploy/.env -f deploy/docker-compose.yml logs -f db backend web
```

健康检查：

- DB：`pg_isready`。
- Backend：`GET /api/v1/health/ready`，必须验证数据库连通。
- Web：Compose执行容器内`wget http://127.0.0.1/health`；人工再检查浏览器首页和`/health`。

## 7. 停止与升级

停止并保留数据：

```bash
docker compose --env-file deploy/.env -f deploy/docker-compose.yml down
```

重新构建并启动：

```bash
docker compose --env-file deploy/.env -f deploy/docker-compose.yml up -d --build
```

**危险操作：** `docker compose down -v`会删除命名卷及数据库数据。只有明确需要销毁测试数据时才可使用。

## 8. 数据备份与恢复

备份：

```bash
docker compose --env-file deploy/.env -f deploy/docker-compose.yml exec -T db \
  sh -c 'pg_dump --clean --if-exists -U "$POSTGRES_USER" "$POSTGRES_DB"' > backup.sql
```

恢复（先备份当前库。--clean 产物会先删除同名对象再重建；仍建议优先在测试环境演练）：

```bash
cat backup.sql | docker compose --env-file deploy/.env -f deploy/docker-compose.yml exec -T db \
  sh -c 'psql -v ON_ERROR_STOP=1 -U "$POSTGRES_USER" "$POSTGRES_DB"'
```

## 9. 常见问题

| 现象 | 可能原因 | 处理 |
|---|---|---|
| Web端口无法访问 | 端口被占用/容器未启动 | 检查APP_PORT和Compose状态 |
| Backend一直unhealthy | DB未ready、迁移失败、应用异常 | 查看backend与db日志 |
| 数据库认证失败 | 用户/密码配置不一致 | 核对.env；首次初始化后改环境变量不自动更改既有数据库用户密码 |
| AI配置错误 | URL、Key或模型名无效 | 核对服务商配置和调用额度 |
| AI请求超时 | 网络、模型服务繁忙或输入过长 | 检查网络，缩短输入，查看request_id |
| 重建后数据消失 | 误删Volume或更换Compose项目名/卷 | 检查卷状态，优先从备份恢复 |
| Nginx返回502 | Backend未就绪或代理地址错误 | 检查backend健康状态和Nginx配置 |

## 10. 安全注意事项

- 默认将Web端口绑定在127.0.0.1；未实现登录认证前禁止公网暴露。
- 数据库端口不映射到宿主机。
- 对外发布前必须补充HTTPS、访问认证、限流、密钥轮换和备份策略。
- 不公开上传`.env`、备份SQL、日志或真实用户输入。

## 11. 干净环境部署验收

使用未安装Python/Node/PostgreSQL的测试主机或干净虚拟机：

本清单第4、5、8步中的compose命令均按§5完整形式执行（`docker compose --env-file deploy/.env -f deploy/docker-compose.yml …`）。

1. 安装Docker和Compose v2。
2. 克隆实际仓库。
3. 配置`.env`。
4. 执行`docker compose config`。
5. 执行`up -d --build`。
6. 检查三个容器健康状态。
7. 访问Web，执行需求分析和用例生成。
8. 创建数据后执行`down`再`up`，确认数据仍在。
9. 完成pg_dump与恢复演练。
10. 记录操作系统、Docker版本、耗时、截图、日志和结果。

验收结果：待Day 25执行后填写。
