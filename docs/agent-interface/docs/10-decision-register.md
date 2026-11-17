# 10 · 成员产品决策登记与未决事项

> 本页固定独立成员产品边界与本地契约，不表示独立仓测试已复跑、新仓已发布或生产已批准。来源事实仅见 [migration.md](../../migration.md)。

## 当前固定决策

| ID | 决策 | 边界与理由 |
| --- | --- | --- |
| D00 | 成员与考试为独立产品和独立源码仓 | 成员 `alethicinsight-member-platform`，考试保留 `alethicinsight-platform`；不擅自重命名 |
| D00-layout | 本仓根 `server/` 为成员服务，`infra/` 为内部接口包 | 独立 DB/账号/密钥/会话/权限/生命周期与进程，不依赖共同父仓 |
| D00-handoff | 录取结果可经明确授权交接至成员维护 | 不共库、不自动同步考生资料或 token；当前无自动交接接口 |
| D01 | 成员网页沿用不透明 Bearer 机制；CLI 用独立短时 grant | 仅本成员库账号/会话；一次性 `request_id + code`，不是 OAuth/OIDC，考试 token 不可复用 |
| D02 | 成员本库当前 `account.role` member/admin 具有组织权限 | 逐请求核验身份/scope/资源 ACL；考试 admin 不自动有成员权限；网页管理还需当前 admin |
| D03 | 成员 API 内部 `/api/v1`，本地服务根 `http://localhost:8902` | CLI base 不含 `/api/v1`；撤销成员 `/recruitment` 公网前缀预期；不改 API 名，考试兼容 API 留在考试进程 |
| D04 | 现有首个事务为 `project-join` | schema revision + 持久草稿，不宣称资源/设备/通用表单均已实现 |
| D05 | 可信成员网页内容绑定确认与当前 admin 审核 | draft/validate 不返回确认 token；CLI 无最终提交/review；业务审批权仍需组织授权 |
| D07 | 仅索引批准公开/成员低敏来源 | restricted 招新档案、成绩/评语、联系方式、私有代码排除；摘要与原文同 ACL |
| D09 | FastAPI/Pydantic/SQLite/vanilla JS 技术栈与独立成员 DB | 成员独立身份与运行状态，不要求微服务化；技术复用不表示共库 |
| D12 | CLI/MCP 同包同版本归成员 infra，本地 stdio MCP | 不新建 MCP 仓；无远程传输、OAuth 或真实 Host 认证声明 |
| D13 | 提交 `draft_id + confirmation` 与幂等头 | 主体+操作+请求摘要绑定幂等，确认/提交原子处理，不接受 fields 直提 |
| D14 | 列表统一 `{items,next_cursor}` | 项目、知识、申请/grant 有界分页，不提供全员导出 |

成员代码内身份约定保持 `from app.org_auth import OrgIdentity, require_org_identity`、`require_org_identity(request, scope=None, web_only=False)`；内部 `student_id/role/principal_id/auth_type/scopes` 不授权对外暴露学号。这里 `app` 是独立成员应用，不能导入独立考试 checkout 的 `app`。

## 待负责人确认或验证

| ID | 待办 | 不得假定 | 阻塞范围 |
| --- | --- | --- | --- |
| D00-proof | 独立仓标准成员/CLI 回归、显式跨仓 HTTP 与合成闭环 | 不继承来源通过数；成员标准测试不需考试源码，跨产品使用 `EXAM_PLATFORM_SOURCE` | 本地交付验收 |
| D00-release | 托管平台发布/远端 URL 与权限 | 本地新仓名不代表已推送或发布，不编造 URL | 另行发布授权 |
| D00-data | 真实身份、历史事务的最小化映射与迁移方案 | 空库演练不等于真实账号迁移；不复制混合库/旧 token | 另行数据迁移 |
| D02-ops | 正式成员维护、退出与权限负责人 | 本地 role 不替代组织制度，考试角色不继承 | 真实成员运营 |
| D03-ops | 成员独立生产网络、域名、TLS/反代、写入权威 | 考试 edge 暂存/回源不能复用为成员部署 | 生产发布 |
| D05-ops | 正式加入条件和审批责任人 | 开发示例不是制度；技术 admin 不等于业务授权 | 真实项目开放 |
| D06 | Gitea 状态、逐仓权威源与许可 | 未核实，知识同步不是全仓镜像 | 仓库集成 |
| D07-ops | 外部模型可见目录与来源许可 | 成员可见不等于允许外发，提示词不是授权 | 真实知识导入 |
| D08 | Windows/POSIX、浏览器、两种真实 Agent 版本与实测 | 协议/自动测试不能代替实机 | 客户端验收 |
| D10 | 本产品离机备份、密钥/恢复责任、RPO/RTO | 考试归档与同盘副本不等于成员恢复 | 生产上线 |
| D11 | 可选官网模型及预算 | 不影响无模型业务，不承诺人均固定次数 | 可选模型 |
| D12-ops | TTL、日志/审计保留、限流与撤销策略 | 不预填未经实现/实测参数 | 运维安全审核 |

## 更新规则

后续变更记录负责人、日期、依据、受影响 API、兼容/数据与回退策略。独立仓源码和合成数据验证可以交付审查；生产、真实迁移、远程访问、身份委托和数据公开仍需分别人工批准。参见 [验收](09-roadmap-and-acceptance.md)与[交接](11-agent-handoff.md)。
