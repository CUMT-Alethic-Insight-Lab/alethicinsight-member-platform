# 成员产品与仓库职责

## 本仓：alethicinsight-member-platform

- `server/`：独立成员服务、最小成员账号、网页会话、CLI 授权、项目、知识和组织事务申请。
- `infra/interface/alethicinsight-cli/`：内部接口基础设施；一个 `alethicinsight-cli` 包/版本提供 CLI 与本地 stdio MCP 两个入口，MCP 调用同包 CLI，二者只访问成员 REST，不直接读 DB。
- `docs/`：成员业务、安全、API、Skill 与本地人工运行契约，不保存真实成员、生产配置或私有运维正文。
- `server/tests/` 与包内 `tests/`：标准成员/接口包测试，不依赖考试源码。
- `integration/test_cross_product_http.py`：可选跨产品 HTTP 集成测试；显式 `EXAM_PLATFORM_SOURCE` 选择独立考试 checkout，不能猜测相邻目录或依赖同仓布局。

## 考试产品：alethicinsight-platform

考试仓名称保持现状，不擅自重命名。招新报名、机试、评审、交卷、考生资料、考试生命周期和 exam-kit 协议属于考试产品，不属于成员服务。本仓不启动考试维护任务，不复用考试部署脚本、edge 暂存转发或 `/recruitment` 路由。

两套产品独立安装/启动，分别配置环境、密钥、DB、数据目录、账号、会话、CLI grant、权限、备份和生命周期。相同账号标识不等于同一身份，考试 admin 不继承成员 admin；不共享 DB、不复制旧 token、不从考试库读取成员资格。

## 录取结果交接

成员维护负责人可以在考试产品给出**明确且经授权确认的录取结果**后，核对业务必要字段与交接依据，再按成员产品自己的流程建立或调整资格。交接不等于自动开通，不把考试报名表、成绩、评语或联系方式批量导入成员知识库。

当前没有自动录取同步、账号迁移或共库桥接接口。真实数据的字段、来源许可、映射/去重、审计、密码与会话失效、保留期和回退方案须另行审批；本地空库演练不授权真实账号操作。

## 相邻仓职责

公共上手资料归 `open-onboarding`，学习索引归 `awesome-ai`，正式制度引用 `governance-public`。私有运维材料归 `infra-handoff`，不承载公共 CLI/MCP 代码。官网与 exam-kit 是独立产品；本次不修改 Frontend-Templates。

仓库名只说明本地职责，不证明托管平台上已创建、发布或部署新仓；未核验的远端 URL 不在示例中编造。来源历史集中在 [migration.md](migration.md) 和 [CHANGELOG](../CHANGELOG.md)。
