# 第三方依赖许可证清单（THIRD-PARTY-NOTICES）

本清单记录运行时/构建时主要第三方依赖的许可证；版本以Day 2-7锁定的lock文件为准，Release前复核更新；维护责任：成员B。下表“基线参考版本”仅为规划期参考值，正式版本一律以锁定结果为准。

| 组件 | 基线参考版本 | 许可证 | 使用方式 | 与本项目MIT许可证的兼容性结论 |
|---|---|---|---|---|
| Vue 3 | 3.5.x | MIT | 前端运行时框架，经npm安装并随前端构建产物分发 | 兼容 |
| Element Plus | 2.14.x | MIT | 前端UI组件库，经npm安装并随前端构建产物分发 | 兼容 |
| Vite | 8.3.x | MIT | 前端构建工具，仅构建期使用，不进入运行时产物 | 兼容 |
| TypeScript | 7.0.x | Apache-2.0 | 开发/构建期语言与编译器，仅构建期使用 | 兼容（Apache-2.0可与MIT项目并存，且仅为构建期工具，不进入运行时产物） |
| FastAPI | 0.162.x | MIT | 后端运行时Web框架，经pip安装并随后端镜像分发 | 兼容 |
| SQLAlchemy | 2.1.x | MIT | 后端运行时ORM，经pip安装并随后端镜像分发 | 兼容 |
| Alembic | 1.20.x | MIT | 数据库迁移工具，后端启动时对数据库执行迁移 | 兼容 |
| Pydantic | 2.14.x | MIT | 后端运行时数据校验与Schema模型（含AI输出结构校验） | 兼容 |
| psycopg 3 | 3.3.x | LGPL-3.0-only | PostgreSQL驱动，以未修改的独立Python库形式经SQLAlchemy dialect调用 | 本项目代码以未修改的独立库形式经SQLAlchemy dialect调用psycopg，不构成对该LGPL库的衍生作品，与本项目MIT许可证兼容；发布时随本清单附LGPL声明 |
| pytest | 9.2.x | MIT | 后端测试框架，仅开发/测试期使用，不进入发布产物 | 兼容 |
| PostgreSQL 17 | 17.x（compose默认`postgres:17-alpine`，Release前锁定Digest） | PostgreSQL License | 容器化运行时数据库服务，以独立容器运行且未修改 | 兼容（宽松许可证，且以独立服务运行） |
| Nginx | 以`frontend/Dockerfile`基础镜像锁定为准 | BSD-2-Clause | 容器化前端入口与API反向代理，以独立容器运行且未修改 | 兼容（宽松许可证，且以独立容器运行） |
