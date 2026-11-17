# 09 · 独立成员仓与可验证验收

## 事实边界

成员产品在 `alethicinsight-member-platform` 独立安装/启动，`server/` 只运行成员应用，`infra/` 是同包 CLI/MCP。考试产品保留独立仓 `alethicinsight-platform`，不假定存在相邻目录或共享父仓。来源历史见 [迁移记录](../../migration.md)。

代码存在、自动测试、本地 HTTP、真实浏览器、真实 Agent Host 和生产部署是不同证据。[API 契约](04-api-and-workflows.md)与 [运行手册](../../AGENT_INTERFACE_MVP.md)不替代执行验证，也不证明远端已发布。

## 成员标准验证（不需要考试源码）

从本仓根目录、已安装 [成员依赖](../../../server/requirements.txt) 的专用环境运行：

```sh
(cd server && python -m pytest tests -q)
(cd infra/interface/alethicinsight-cli && python -m pip install '.[test]' && python -m pytest -q)
python -m pip wheel --no-deps ./infra/interface/alethicinsight-cli --wheel-dir /path/to/new-temporary-wheel-dir
```

标准成员套件不得导入考试 app、加载考试 conftest，或要求 `EXAM_PLATFORM_SOURCE`。默认 CLI 源为本仓 `infra/interface/alethicinsight-cli/`。Node.js 用于已有页面 DOM 合约；缺依赖导致的跳过必须记录，不算真实浏览器验收。wheel 需要另在干净 venv 安装并运行两个正式入口及真实 HTTP，不以构建成功代替安装验证。

## 可选跨产品 HTTP 集成

跨产品测试独立放在本仓 `integration/test_cross_product_http.py`，不放入 `server/tests/`。操作者必须显式选择独立考试 checkout 根目录；未设置或无效路径不能自动猜同仓/邻仓，也不能伪造通过。

在另一个全新集成测试环境中，从本仓根执行，先替换为已经审核的本地考试 checkout：

```sh
export EXAM_PLATFORM_SOURCE='/absolute/path/to/alethicinsight-platform'
python -m pip install -r server/requirements.txt
python -m pip install -r "$EXAM_PLATFORM_SOURCE/server/requirements.txt"
python -m pytest integration/test_cross_product_http.py -q
```

PowerShell：

```powershell
$env:EXAM_PLATFORM_SOURCE = 'C:/work/alethicinsight-platform'
python -m pip install -r server/requirements.txt
python -m pip install -r "$env:EXAM_PLATFORM_SOURCE/server/requirements.txt"
python -m pytest integration/test_cross_product_http.py -q
Remove-Item Env:EXAM_PLATFORM_SOURCE
```

此环境仅运行合成集成测试，不使用生产配置/真实账号。两个服务子进程必须各有独立配置、密钥、DB、数据目录与导入路径；共用测试驱动解释器不表示共用运行状态。日常成员安装不需要安装考试依赖。

考试标准回归由考试仓负责人在其自己的环境从考试根执行 `(cd server && python -m pytest tests -q)`；这不是本仓的第二次成员测试，不在本任务擅自修改考试源码或重命名考试仓。

## 自动化门禁

| 领域 | 必须证明 | 证据要求 |
| --- | --- | --- |
| 成员独立启动 | 仅加载本仓应用/依赖，无考试路由、表或生命周期任务 | 标准成员测试与真实 HTTP |
| 数据与身份隔离 | 双空库/合成账号；初始化、写入和维护不改变另一库；跨服务 session/grant 拒绝 | 显式跨仓 HTTP，两个服务进程 |
| 考试边界 | 考试兼容接口保留，`/org` 与组织 `/api/v1` 不可用 | 考试仓标准回归与跨产品 HTTP |
| 账号生命周期 | 当前角色、禁用、密码变更及 grant 撤销生效，考试 admin 不继承成员权限 | 权限负例 |
| CLI grant | code 匹配/过期/单次兑换；本人授权分页/撤销和自撤销；不复用网页 token | HTTP 回归 |
| 可见性/分页 | 列表、摘要、详情/原文同 ACL；他人草稿不可读；restricted 不索引 | 契约及权限测试 |
| 确认/事务 | validate/draft 无确认；CLI 不能取确认；内容绑定/单次确认、幂等重放/冲突、补件/撤回/审核规则 | 事务回归 |
| CLI/MCP 包 | 干净 wheel 安装两个入口；真实成员 HTTP + CLI/MCP 合成闭环，无最终提交/审核工具 | 安装、协议与 HTTP |
| 契约/文档 | YAML/$ref/operationId/路径参数、本地 Markdown 链接，人工契约对照运行时 OpenAPI | 具体命令与结果 |

不预填通过数。每次记录 commit、OS/依赖、实际命令、退出码、失败/跳过及原因；来源通过数不能直接转为独立仓证据。

## 无模型人工端到端验收

1. 本地同时启动成员 8902 与独立考试 8901，使用两套空开发目录、独立合成账号/密钥，确认 DB 路径不同。
2. 验证相同 ID 的密码、角色、session/grant 不跨产品生效；考试 `/org` 与组织 API 不可用，成员不提供考试业务或回收任务。
3. 本人在 Agent 外经成员网页授权 CLI，查询可见项目/知识/schema，validate/保存草稿；响应无确认 token。
4. 在可信成员网页读取本人草稿、核对后确认提交；重试不重复创建；另一成员不能读，CLI 与非 admin 不能审核。
5. 网页当前 admin 审核，本人查询；角色变更或撤销 grant 后重复调用被拒。无 LLM/Gitea/远程 MCP 仍可完成业务。

自动 HTTP 可以模拟确认接口，不证明人类已看过内容；真实浏览器、MCP Host 与命令行 Agent 分别记录版本、环境和负例。

## 生产与后续门禁（另行授权）

- 成员网络/域名、TLS、服务账号、反代、秘密、审计和离机备份独立评审，不继承考试 `/recruitment` 或 edge 回源配置。
- 明确录取结果交接可由成员维护负责人核对后处理资格，但真实字段、来源许可、映射/去重、旧密码/session/grant 失效、保留与回退须另审；不能复制混合 DB 或自动同步完整考生资料。
- 独立 POSIX、目标硬件、备份恢复、真实浏览器和两种真实 Agent 各自验收；未执行即标未验证。
- OAuth/OIDC、Gitea、设备借用/通用资源、指派审核、远程 MCP、官网模型后续单独评审，不纳入当前已实现清单。
