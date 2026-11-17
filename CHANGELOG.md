# Changelog

## Unreleased — 独立成员仓整理

- 新成员仓名为 `alethicinsight-member-platform`。原库 `alethicinsight-platform` 的 PR #6 已合并；来源基线为 `9dc7229`，成员子树分离提交为 `da82783`。分离根目录即原成员子树，当前布局为 `server/`、`infra/`、`docs/`。
- 承接经审阅的成员设计契约、OpenAPI 人工契约、org-access Skill 和本地运行手册，修正独立仓路径、产品边界、安装与测试示例。
- CLI 与本地 stdio MCP 同属内部 infra 的一个 `alethicinsight-cli` 包/版本；正式命令 `alethicinsight-cli`、`alethicinsight-mcp` 和成员 API 名保持不变。
- 标准成员测试不依赖考试源码；跨产品 HTTP 测试独立到 `integration/test_cross_product_http.py`，显式通过 `EXAM_PLATFORM_SOURCE` 选择考试源码。
- 根 `.gitignore` 保护虚拟环境、本地数据、凭据、数据库、日志和构建缓存；保留合成 fixtures 的可追踪性，任何真实秘密仍禁止提交。
- 考试仓已按授权从 `alethicinsight-platform` 恢复为 `recruitment-exam-platform`；当前考试引用与 checkout 示例对齐新名，来源历史保留旧名。旧 GitHub URL 保留重定向，考试仓成员副本仍待 [PR #7](https://github.com/CUMT-Alethic-Insight-Lab/recruitment-exam-platform/pull/7) 移除，不能视为已完成源码去重。不共享 DB/账号/密钥/会话；可经明确录取结果人工交接成员资格，不自动同步考生档案。

来源和历史证据见 [docs/migration.md](docs/migration.md)。这是未发布整理记录，不代表已推送、发布包、部署生产或迁移真实账号，也不将来源测试数字当作独立仓已复测的结果。
