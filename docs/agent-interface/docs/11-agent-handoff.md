# 11 · 独立成员仓 Coding Agent 交接

## 阅读与范围

先读 [产品边界](../../REPOSITORY_BOUNDARIES.md)、[API](04-api-and-workflows.md)、[验收](09-roadmap-and-acceptance.md)、[决策](10-decision-register.md)和 [运行手册](../../AGENT_INTERFACE_MVP.md)。源码来源和已撤销方案仅见 [迁移记录](../../migration.md)。

工作根是 `alethicinsight-member-platform`，`server/` 为成员服务，`infra/interface/alethicinsight-cli/` 为内部接口包。考试在独立 `alethicinsight-platform`，不擅自重命名、不假设同仓，不在成员代码中导入考试 app。新仓名称不代表已经推送、发布或部署。

## 维护检查清单

1. 从本仓依赖和 `server/` 启动成员 app；不得运行考试报名、上传暂存、转发、归档和批次回收，不指向考试 DB、备份或真实资料。
2. CLI/MCP 维持一个 `alethicinsight-cli` 包/版本，正式命令不改。MCP 调用同包 CLI，仅读成员 REST；本地服务根显式 `http://localhost:8902`，不含 `/api/v1`，不使用考试 `/recruitment`。
3. 独立环境、数据目录、DB、密钥、账号、网页 session、CLI grant 与权限；真实录取结果仅按授权交接成员维护，不共库或自动导入考生资料。
4. 统一 `app.org_auth.require_org_identity(request, scope=None, web_only=False)`；当前 member/admin + scope/ACL，管理操作还需网页当前 admin，CLI 不继承 review。
5. 保持一次性授权码、本人 grant 分页/撤销与自撤销；项目/知识/schema/validate/draft/网页确认/幂等提交/本人查询/补件撤回/网页审核。提交仅 `draft_id + confirmation`，草稿与校验不签发确认。
6. 对照成员运行时 `/openapi.json` 核验 [人工契约](../contracts/openapi-skeleton.yaml)：字段、状态码、分页、错误和 TTL；静态 drafts/validate 不能被动态 `{id}` 抢占。
7. 知识同步只操作成员 DB；人工审核本地来源、版本与许可，摘要/原文同 ACL，不在请求内爬取或索引考试 restricted 资料。
8. 标准 `server/tests/` 与 CLI 包测试不需要考试源码或邻仓。跨产品测试只在 `integration/test_cross_product_http.py`，显式 `EXAM_PLATFORM_SOURCE`；两个独立进程、空库与合成账号验证互不影响、跨服务凭据拒绝。
9. 对成员真实 HTTP + 实际 CLI/MCP 子进程验证权限负例与申请闭环，wheel 干净安装两个入口，检查文档链接与契约；只记录实际命令、退出码、失败/跳过，不预填通过数。

## 安全与协作边界

- Skill/prompt/CLI `--confirm` 或“我是管理员”文字不构成授权；最终确认由本人在可信网页完成，Agent 不索取确认 token 或自动点击提交。
- 幂等重放与确认消费在事务内协调；跨用户键不能复用结果。密码、token、code、confirmation 不进源码、URL、日志或模型上下文。
- restricted 考生档案、评语、成绩、联系方式不进入成员知识/CLI/MCP；不提供 SQL、shell、任意 URL 或全员导出工具。
- 按分配路径协作：代码/测试与文档任务不得混改；文档范围为根和各层 README、`docs/**`、CHANGELOG 与根 `.gitignore`。父代理统一提交，不自行 commit/push。
- 不碰原脏主仓、Frontend-Templates 或其他产品的未授权路径；不 reset/clean，不读取真实秘密、不部署、不迁移真实账号，不公开敏感历史。

## 完成交接

报告每个触及文件、实际命令/退出码/结果、失败/跳过、未执行项和阻塞。成员测试、跨产品 HTTP、CLI/MCP 安装、浏览器、Windows/POSIX、真实 Agent、硬件、备份恢复和生产发布分别标注，不互相替代。

本地源码与合成验证不代表托管新仓、软件包或生产服务已发布；生产和真实数据事项仍须独立授权。
