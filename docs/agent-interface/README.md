# Alethic Insight · 内部成员平台接口体系

面向认证内部成员的知识、项目与事务接口。网页、REST、CLI 和个人 Agent 可采用不同工具，但使用同一成员业务契约；不是公众聊天 AI、自动经营组织或自主审批系统。

本设计属于独立成员仓 `alethicinsight-member-platform`：`server/` 为成员服务，`infra/interface/alethicinsight-cli/` 为同包 CLI/MCP。考试属于独立 `recruitment-exam-platform`；不共用账号、DB、密钥、会话或生命周期。源码与文档不证明新仓已发布或生产已部署。

## 使用入口

- [成员平台安装与启动](../../README.md)
- [CLI/MCP 同包安装](../../infra/README.md)
- [本地完整运行手册](../AGENT_INTERFACE_MVP.md)

成员本地服务根显式使用 `http://localhost:8902`，CLI base 不含 `/api/v1`，不再使用考试 `/recruitment`。网页登录和 CLI grant 都属于成员独立账号/DB，不继承考试管理员权限。CLI/MCP 只查询/准备草稿，正式提交由本人在可信网页核对确认，review 由本产品网页当前 admin 执行。

## 设计原则

1. 仅服务认证内部成员，scope 与资源 ACL 逐请求验证；公开入口不等于匿名可访问内部数据。
2. API 是业务能力逻辑入口；网页、CLI/MCP 都不能绕过服务端规则或直接读 DB。
3. BYOA：成员自行选择并承担个人模型费用；没有模型也能查询、填表与办理事务。
4. 成员产品内部可采用 FastAPI/SQLite 模块化单体，但不能与考试共库/身份/生命周期。技术复用不等于共享运行状态。
5. Skill L1/L2/L3 只是上下文发现，不是鉴权。资料、搜索摘要与原文同权限，成员可见不自动等于可外发给模型。
6. Gitea、设备/资源、官网模型、OAuth/OIDC 和远程 MCP 是后续方案，不是本次交付。N5095/4GB/512GB 仅为待实测设计约束；可选模型的计量周期/预算/价格须另行确认，不是已上线制度。

## 文档目录

| 文档 | 作用 |
| --- | --- |
| [00 · 当前系统边界](docs/00-current-system-review.md) | 当前成员范围、有效契约与证据边界 |
| [01 · 产品与用户故事](docs/01-product-and-user-stories.md) | 价值、范围与长期场景 |
| [02 · 架构](docs/02-architecture.md) | 成员产品内部各层与独立产品边界 |
| [03 · 认证](docs/03-auth-and-permissions.md) | 当前 grant 与未来 OAuth 的区别 |
| [04 · API 与工作流](docs/04-api-and-workflows.md) | 当前成员契约与可信网页确认 |
| [05 · CLI/MCP/Skill](docs/05-cli-mcp-skills.md) | 同包适配、命令与安全限制 |
| [06 · 数据模型/集成](docs/06-data-model-and-integrations.md) | 独立成员 DB、逻辑模型与未来集成 |
| [07 · 运维方案](docs/07-deployment-ops.md) | 未执行的成员部署、备份/恢复设计 |
| [08 · 安全隐私](docs/08-security-privacy.md) | 威胁模型与外部 Agent 数据界限 |
| [09 · 验收](docs/09-roadmap-and-acceptance.md) | 独立成员标准测试、跨仓集成与上线门禁 |
| [10 · 决策](docs/10-decision-register.md) | 固定边界与未决事项 |
| [11 · Coding Agent 交接](docs/11-agent-handoff.md) | 独立仓维护、协作范围与证据要求 |
| [12 · 参考标准](docs/12-references-and-compatibility.md) | 官方资料与兼容性限制 |
| [OpenAPI](contracts/openapi-skeleton.yaml) | 人工维护成员契约，需对照运行时 |
| [org-access Skill](skills/org-access/SKILL.md) | 薄工作流，不含凭据或实时数据 |
| [CLI 示例](examples/cli-usage.md) | 本地授权/查询/草稿，不代表生产验证 |

## 最小旅程与待办

成员在 Agent 外完成 CLI 授权，查询可见项目和制度，按 schema 准备草稿，本人在可信成员网页最终核对与提交，负责人网页审核，本人查询状态。审批与资源资格不能由 Agent 自己推断。

真实成员维护/退出负责人、首批项目与审批制度、来源许可、允许外发的数据分级、生产网络/域名、备份目标及 RPO/RTO 都须负责人确认。真实浏览器/两种 Agent、POSIX、硬件、恢复与生产分别验收，不能由协议测试代替。

## 相邻仓职责

公共上手归 `open-onboarding`，学习索引归 `awesome-ai`，正式制度引用 `governance-public`，私有运维归 `infra-handoff`。CLI 与 stdio MCP 同一个 `alethicinsight-cli` 发行包/版本，归成员 infra，不另建 MCP 仓。已独立的 exam-kit 不属于成员产品；Frontend-Templates 完全不改。

成员仓为 `alethicinsight-member-platform`，考试仓已按授权恢复为 `recruitment-exam-platform`。可通过明确录取结果经授权交接成员资格，但不共 DB、不自动同步考生资料。新仓名称不代表已发布，不编造未经核验的远端 URL；不部署、不读秘密、不迁移真实数据。参见[产品边界](../REPOSITORY_BOUNDARIES.md)与[迁移记录](../migration.md)。
