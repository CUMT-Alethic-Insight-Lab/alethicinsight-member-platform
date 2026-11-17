# Alethic Insight 成员平台

`alethicinsight-member-platform` 面向认证内部成员，提供知识检索、项目目录、项目加入申请、本人状态查询和网页审核。成员可使用网页、REST、CLI 或个人 Agent；没有模型也能办理事务。招新报名、机试、评审与考生档案属于独立考试产品，不在本仓运行。

## 安装与本地启动

需要 Python 3.10+。以下命令从本仓根目录在新 Git Bash/Linux 终端执行，仅用于空库和合成账号的本地开发；不要指向真实成员资料、考试数据或备份。

```sh
python -m venv .venv-member
# Windows Git Bash；Linux 使用 source .venv-member/bin/activate
source .venv-member/Scripts/activate
python -m pip install -r server/requirements.txt
export MEMBER_PLATFORM_DATA_DIR="$PWD/.local-member"
export MEMBER_PLATFORM_DB_PATH="$MEMBER_PLATFORM_DATA_DIR/member.db"
export MEMBER_PLATFORM_ROLE=authoritative
export MEMBER_PLATFORM_PUBLIC_PREFIX=''
# 仅首次初始化生成；在受控秘密管理位置保留，重启必须恢复同一值
export MEMBER_PLATFORM_SECRET_KEY="$(python -c 'import secrets; print(secrets.token_urlsafe(48))')"
(cd server && python scripts/bootstrap_member.py --student-id local-owner)
```

bootstrap 交互读取两次不少于 12 字符的唯一临时管理员密码，只允许空成员库执行，不通过参数或文件传密码。没有默认管理员、默认密码或自助注册。成功后在同一终端启动：

```sh
(cd server && python -m uvicorn app.main:app --host 127.0.0.1 --port 8902)
```

- 打开 `http://localhost:8902/login`，以显式创建的 `local-owner` 登录，先修改临时密码，再进入 `/org`。
- 管理员可在 `/login` 管理成员；在 `/org` 管理项目和处理已提交的申请。普通成员可查询知识、项目和本人申请。
- REST 根为 `http://localhost:8902/api/v1`；运行时契约为 `http://localhost:8902/openapi.json`。
- 新库的项目和知识目录可以为空；由获授权管理员创建合成项目，知识来源按手册审核，不预置真实成员或虚构库存。

重启时恢复**同一成员密钥、DATA_DIR 和 DB_PATH**，只运行 uvicorn，不重新生成密钥或 bootstrap。更换密钥会使已有密码验证失败。成员服务只读取 `MEMBER_PLATFORM_*` 配置，密钥至少 32 字符且独立于考试；不打印、提交或向 Agent 粘贴秘密。完整配置及 PowerShell 示例见 [server/README.md](server/README.md)。

## CLI 与本地 MCP

CLI/MCP 是本仓内部 infra 的**同一个 `alethicinsight-cli` 发行包、同一版本**，安装提供两个正式命令，不另建 MCP 仓或包。安装不会启动服务。

```sh
python -m pip install ./infra/interface/alethicinsight-cli
export ALETHICINSIGHT_API_BASE_URL=http://localhost:8902
alethicinsight-cli --help
alethicinsight-mcp --help
```

服务根不含 `/api/v1`，没有默认线上地址。登录前按 [infra 说明](infra/README.md#隔离凭据与登录)创建仓库外全新私有 `ALETHICINSIGHT_CONFIG_DIR`，由本人在 Agent 会话外完成 `alethicinsight-cli auth login`。默认只读授权可查项目、知识和本人申请；表单 schema、校验和草稿另需显式 `applications:create`。

CLI/MCP 只读取、校验和准备草稿。本人必须在可信 `/org` 网页重新核对并最终提交；只有成员平台当前网页 admin 可审核，CLI grant 不继承 admin review。密码、网页 token、一次性 code、CLI token 和确认 token 不进入模型上下文。详见 [包 README](infra/interface/alethicinsight-cli/README.md)、[运行手册](docs/AGENT_INTERFACE_MVP.md)及 [org-access Skill](docs/agent-interface/skills/org-access/SKILL.md)。

## 独立产品边界

本仓 `server/` 独立于考试仓 [`recruitment-exam-platform`](https://github.com/CUMT-Alethic-Insight-Lab/recruitment-exam-platform)，`infra/` 只通过成员 REST 访问业务，不直接读库。两套产品分别拥有进程、配置、密钥、数据目录、数据库、账号、会话、授权、权限与生命周期。考试管理员不自动成为成员管理员；相同账号标识不表示共享身份。

考试录取后可以通过**经负责人确认的明确录取结果交接**进入成员维护流程，但不共用 DB、不自动导入考生档案、不继承考试 token。当前不提供自动跨产品同步接口。考试仓名已按授权恢复；本次仅更新成员文档，不再执行重命名。考试仓 `main` 中的成员副本仍待 [PR #7](https://github.com/CUMT-Alethic-Insight-Lab/recruitment-exam-platform/pull/7) 移除，不能视为已完成源码去重。详见 [产品与仓库职责](docs/REPOSITORY_BOUNDARIES.md)。

## 开发验证

标准成员测试不需要考试源码；Node.js 用于已有组织页面 DOM 合约测试。

```sh
(cd server && python -m pytest tests -q)
(cd infra/interface/alethicinsight-cli && python -m pip install '.[test]' && python -m pytest -q)
```

跨产品 HTTP 测试单独放在 `integration/test_cross_product_http.py`，需要操作者显式设置 `EXAM_PLATFORM_SOURCE` 为独立考试 checkout 根目录，并安装其测试所需依赖；不纳入标准成员测试。命令、边界和验收记录要求见 [验收门禁](docs/agent-interface/docs/09-roadmap-and-acceptance.md)。这些命令不是本次运行或通过声明。

## 文档与交付范围

- [运行与人工运维手册](docs/AGENT_INTERFACE_MVP.md)：首次初始化、重启、CLI/MCP、知识同步、备份/恢复方案。
- [设计契约索引](docs/agent-interface/README.md)：产品、架构、权限、API、数据、运维和安全约束。
- [OpenAPI 人工契约](docs/agent-interface/contracts/openapi-skeleton.yaml)：需与实际运行时契约比对，不替代执行验证。
- [迁移记录](docs/migration.md)与 [CHANGELOG](CHANGELOG.md)：来源、基线及历史证据边界。

本仓文档不宣称新仓或软件包已发布，也不宣称生产部署、真实账号/知识迁移、真实浏览器、真实 Agent Host、目标硬件或备份恢复已验收。上述工作需要各自授权与证据。
