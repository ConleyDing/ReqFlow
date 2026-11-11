# 25 技术依据与官方参考

本项目采用成熟的Web、API、ORM和容器技术。以下链接用于开发前核对配置和官方推荐实践；实际依赖版本在Day 2-7根据兼容性测试后锁定。不要只因文档页面显示“最新”就自动升级项目依赖。

| 技术 | 官方参考 | 本项目关注点 |
|---|---|---|
| Docker Compose | https://docs.docker.com/compose/gettingstarted/ | 服务编排、健康检查、depends_on、命名卷 |
| PostgreSQL容器部署 | https://docs.docker.com/guides/postgresql/ | 官方镜像环境变量、数据卷和备份注意事项 |
| FastAPI容器化 | https://fastapi.tiangolo.com/deployment/docker/ | 镜像构建和容器运行方式 |
| SQLAlchemy PostgreSQL | https://docs.sqlalchemy.org/en/20/dialects/postgresql.html | psycopg连接方言、类型和数据库适配 |
| SQLAlchemy asyncio（仅供参考） | https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html | 本项目首版选择同步模式，不需引入异步Session复杂度 |
| Vue | https://vuejs.org/guide/ | Vue 3组件、路由和组合式API |
| Vite | https://vite.dev/guide/ | 前端构建与开发服务器 |
| Alembic | https://alembic.sqlalchemy.org/ | 数据库迁移管理 |

## 版本冻结原则

1. Day 2记录Python、Node、PostgreSQL、Docker和依赖的可用版本。
2. 选取稳定、团队环境可安装的版本，不追求使用最新版本。
3. 前后端依赖分别锁版本，容器基础镜像固定主/次版本或Digest。
4. 版本变更必须在PR中说明原因并运行回归测试。
5. 项目发布时记录Docker版本、Compose版本、PostgreSQL版本和模型版本。
