# 成员仓来源与迁移记录

本页仅记录源码/文档来源和历史证据边界，不是发布、部署或真实数据迁移记录。当前产品入口见 [README](../README.md)，职责见 [产品边界](REPOSITORY_BOUNDARIES.md)。

## 独立仓来源

- 历史原库名：`alethicinsight-platform`。该考试仓现已按授权恢复为 [`recruitment-exam-platform`](https://github.com/CUMT-Alethic-Insight-Lab/recruitment-exam-platform)，旧 GitHub URL 保留重定向；下列来源基线、提交与 PR #6 仍按当时的仓名记录，不追改历史。
- 原库 PR #6 已 MERGED，`origin/main` 来源基线为 `9dc7229`（完整 SHA：`9dc7229cae1ee709c89a243e035b2685a11f2475`）。
- 从原成员子树 `member-platform` 通过 `git subtree` 分离；subtree 提交为 `da82783`（完整 SHA：`da827835222ac1ef30aac4d86b091afb32e27468`）。
- 新成员仓名：`alethicinsight-member-platform`；仓根就是原成员子树根，当前路径为 `server/` 和 `infra/interface/alethicinsight-cli/`，命令不再带旧子树前缀。
- 成员设计契约、OpenAPI 人工契约、CLI 示例、org-access Skill 来自来源工作树的 `docs/agent-interface`；成员运行手册来自 `docs/AGENT_INTERFACE_MVP.md`。承接成员文本并适配独立仓，不复制考试资料、私有运维正文、真实账号、凭据或数据库。

本地仓名和分离提交不能证明远端新仓已创建、推送或公开发布。此文档任务不执行 commit/push、软件包发布、生产部署、真实身份/知识迁移或敏感历史公开。

## 已撤销方案与布局变化

此前把招新平台增量扩展为成员平台并复用账号/DB/启动流程的方案已撤销。来源旧混合基线 `229e69c` 不能当作当前成员架构；技术复用不表示运行状态共享。

PR #6 完成的是同仓的成员/考试运行边界整理；随后将成员子树分离为本独立仓。原“第一阶段同仓、独立仓映射待定”的文案不再是当前操作依据。`server/` 在本仓明确指成员服务，不能误读为考试服务。

考试仓名称恢复不等于源码清理完成：考试仓根 `server/` 仅提供考试运行能力，但其 `main` 仍有 `member-platform/` 副本，待 [PR #7](https://github.com/CUMT-Alethic-Insight-Lab/recruitment-exam-platform/pull/7) 移除。本次只对齐成员文档中的当前仓名与 checkout 示例，不宣称该 PR 已合并或源码已去重；既有本地 checkout、历史证据与保全路径不因远端更名而自动改名。

保留的成员契约为独立账号/DB/会话/CLI grant、逐请求角色/scope/ACL、内容绑定网页确认和幂等提交、网页 admin 审核。CLI/MCP 仍为内部 infra 一个包/版本，正式入口和 `/api/v1` API 名不改。

## 来源验证记录，不是本次复测

以下信息由本次交接提供，不是文档执行者重新运行得到：来源同仓隔离已通过考试 **308**、成员 **128**、CLI **193**。这些数字只适用于来源提交与当时布局；不能作为新独立仓的当前通过数，也不能代替其标准命令、显式跨仓集成、安装产物和 HTTP 验证。

来源运行手册另记录过 2026-11-04 的本地合成教程复核：独立临时环境中的管理员初始化、成员登录/强制改密/HTTP 查询、同密钥重启和已安装 CLI 授权查询。该记录不等于交互隐藏输入、人工浏览器确认或真实 Agent Host 验收；本次未复跑。旧混合布局的 451/566/311 等数字不再放在当前运行手册中，也不用于独立仓验收。

新仓测试应逐次记录实际 commit、命令、依赖/OS、退出码、失败/跳过和证据范围。标准成员测试不需要考试源码；跨产品测试独立为 `integration/test_cross_product_http.py`，显式使用 `EXAM_PLATFORM_SOURCE`。详情见 [验收](agent-interface/docs/09-roadmap-and-acceptance.md)。

## 数据与生产事项

源码分离不迁移任何真实成员身份。考试可在明确录取结果经授权确认后交接必要信息，再由成员维护负责人按独立成员流程处理；不共用 DB、不导入成绩/评语/完整考生档案、不继承 token。任何自动化交接须另立契约和安全评审。

生产网络、域名/TLS/反代、秘密托管、备份恢复、真实数据映射与回退、许可及外部模型数据政策都需要独立审批。未执行项不能用本地合成 HTTP、协议测试或历史通过数代替。
