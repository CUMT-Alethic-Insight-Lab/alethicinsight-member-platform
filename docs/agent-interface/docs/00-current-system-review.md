# 00 · 当前成员系统与契约边界

## 当前产品

`alethicinsight-member-platform` 是独立成员仓，根 `server/` 运行成员应用，`infra/interface/alethicinsight-cli/` 提供同包 CLI/本地 stdio MCP。考试产品位于独立 `alethicinsight-platform`，不在本仓运行。来源与撤销方案仅记录于 [迁移记录](../../migration.md)，不作为当前安装步骤。

- FastAPI/Pydantic/SQLite 与 vanilla JS 是成员产品的技术栈，不表示继承考试账号、密钥、DB 或部署拓扑。
- 成员网页登录使用不透明 Bearer session，不是 Cookie 或 OAuth/OIDC；CLI 使用一次性 code 交换短时、可撤销 grant，不能复用网页 token。
- 本成员库当前 member/admin 才能使用组织接口；管理操作还需当前 admin 的网页会话。考试管理员不自动获得成员权限，CLI grant 不继承 review。
- 同名 `app` 模块只在各产品自己的进程和导入路径使用，不从考试 checkout 导入业务。

## 必须保持的契约与安全约束

1. 成员 API 为 `/api/v1`，仅挂载在成员进程；考试兼容接口与 exam-kit 协议归独立考试产品。
2. 成员 Application 是项目加入等组织事务，不是考试报名。招新问卷、成绩、评语和联系方式不进入通用知识/CLI/MCP。
3. `profile:read:self` 只提供最小身份，不默认输出学号、手机号、出生年月等完整档案；主体响应使用稳定不可枚举 ID。
4. 当前账号/角色、scope、资源 ACL 逐请求核验；降级、退出、禁用、重置密码、改密和 grant 撤销须各自回归。
5. 表单带来源/revision/更新时间，确认绑定持久草稿与规范化摘要；内容、版本或身份变化后重新确认。最终确认在可信成员网页，不能用 CLI `--confirm` 或模型文字代替。
6. 幂等按 principal + operation + 请求摘要隔离，同键异请求冲突，确认消费与合法重放须协调，防止重试重复申请。
7. 搜索摘要、原文、缓存和导出同 ACL；restricted 不靠 prompt 保密。成员可见不等于允许外发给个人模型。
8. 外部同步有界、超时，不在事务请求中阻塞网络；知识从审核来源起步，不索引私聊或全量私有代码。
9. 录取结果只能经明确授权交接到成员维护流程，不共享 DB，不自动导入考试档案或 token。

## 证据边界

源码、人工契约、标准测试、真实 HTTP、人工浏览器确认、真实 Agent Host 和生产部署是不同证据，必须分别记录。当前文档整理不重新声明来源测试结果，不宣称远端已发布、生产已部署或真实成员已迁入。

生产网络、TLS、硬件、备份恢复、正式成员维护和审批责任、外部模型数据政策均须单独确认。OAuth/OIDC、Gitea、远程 MCP、设备资源和官网模型仍是后续设计。详见 [职责](../../REPOSITORY_BOUNDARIES.md)、[API](04-api-and-workflows.md)与 [验收](09-roadmap-and-acceptance.md)。
