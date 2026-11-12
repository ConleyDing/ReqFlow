# 15 开源社区发布计划

## 1. 平台选择

建议GitHub作为主仓库，便于面向国际开发者展示、管理Issue/PR和Release；若课程或本地网络环境要求，也可同步维护Gitee镜像。最终以真实创建的仓库地址为准，不在文档中虚构URL、Star、下载量或反馈。

## 2. 开源许可证

建议MIT License，适合小型学习/工具项目，条款简洁。正式发布前需将LICENSE中的版权主体占位符替换为真实团队/组织信息，并核查所有第三方依赖许可证。

## 3. 仓库必备文件

- README.md：项目介绍、功能、架构、截图、部署和AI配置。
- LICENSE：正式许可证。
- CONTRIBUTING.md：贡献流程。
- CODE_OF_CONDUCT.md：协作行为准则。
- CHANGELOG.md：版本变更。
- THIRD-PARTY-NOTICES.md：第三方依赖许可证清单，Release前按Day 2-7锁定的版本复核更新。
- .gitignore：排除密钥、构建产物、数据库备份。
- .dockerignore：Docker构建上下文排除规则（与.gitignore防护面对齐，排除*.sql、*.dump、*.log、backups/等）。
- docs/：工程文档。
- frontend/、backend/、tests/、deploy/：源代码、测试和部署。
- `deploy/.env.example`：仅包含配置键和安全占位值。

## 4. 发布步骤

1. 创建真实公开仓库，补充描述、Topics和默认分支。
2. 将代码和文档提交到main，确认`.env`、数据库备份和个人数据未提交。
3. 检查README中的部署命令能在干净环境执行。
4. 执行测试和安全检查，记录版本号、Commit SHA和测试报告。
5. 创建Release候选版并由两人复核。
6. 核心验收通过后打`v1.0.0`标签并发布Release。
7. 发布截图、操作视频、文档链接和问题反馈渠道；`ReqFlow_AI_项目汇报.pptx`与`ReqFlow_AI_系统操作演示.mp4`以Release附件发布。
8. 后续Issue按Bug/Feature/Docs分类维护。

## 5. README必须展示

项目一句话定位、适用人群、核心流程、功能截图、AI能力边界、技术栈、架构图、快速部署、模型配置、示例需求输入（静态样例，不是内置示例项目）、限制、安全提醒、许可证、贡献指南和已知问题。

## 6. 发布前检查

- [ ] 公开仓库真实可访问。
- [ ] LICENSE 版权主体已替换为真实团队信息、正文仅含标准 MIT 文本（无占位符、无批注）。
- [ ] 依赖许可证已检查。
- [ ] CODE_OF_CONDUCT联系渠道已补充为真实有效方式。
- [ ] THIRD-PARTY-NOTICES.md已按锁定版本复核并随仓库发布。
- [ ] 无真实API Key、密码、日志和用户数据。
- [ ] 默认部署不会意外暴露数据库端口。
- [ ] 未实现功能未标记为已完成。
- [ ] README命令经过实际验证。
- [ ] Release附件及版本说明与代码一致。

## 7. 维护约定

对外问题不承诺即时响应；两名维护者至少在项目交付期内定期查看Issue。Bug修复需补测试，功能新增需经过范围评估。真实仓库地址、Release链接和发布时间在发布后补录。
