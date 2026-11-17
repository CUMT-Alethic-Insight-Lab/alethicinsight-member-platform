# 内部成员平台 MVP：本地运行与人工运维手册

本手册服务于独立成员仓 `alethicinsight-member-platform`，布局为成员 `server/` 与内部接口包 `infra/interface/alethicinsight-cli/`。考试属于独立 `alethicinsight-platform`，不在本仓启动；新仓名称不代表已经发布或生产部署。来源与历史证据仅见 [migration.md](migration.md)。

- [产品安装入口](../README.md)、[infra 安装入口](../infra/README.md)
- [API 与工作流](agent-interface/docs/04-api-and-workflows.md)、[OpenAPI](agent-interface/contracts/openapi-skeleton.yaml)
- [验收](agent-interface/docs/09-roadmap-and-acceptance.md)、[决策](agent-interface/docs/10-decision-register.md)、[集成交接](agent-interface/docs/11-agent-handoff.md)

## 1. 服务、数据与权限边界

成员服务根示例为 **`http://localhost:8902`**，内部 REST `/api/v1`，网页 `/org`；CLI base 是服务根，不含 `/api/v1`。不预设成员生产域名、`/recruitment` 前缀或考试 edge/authoritative 反代链路，不把独立考试部署脚本用作成员部署。

成员服务拥有独立 DB、账号、网页 Bearer session 与 CLI grant；不能使用考试 DB、账号凭据或会话。相同 API 登录路径或配置键拼写不代表身份共用。只有本成员库当前 member/admin 可用组织接口；网页管理还需当前 admin，CLI scope 不授予 review。考试管理员不是成员平台管理员。

API 名保持不变；`Application` 仍是项目加入等组织事务，不是招新 application。独立考试产品保留 `/api/auth/*`、`/api/campaigns/*` 和旧 exam-kit 协议，但不对成员库提供桥接。CLI 一次性 code 交换是短时授权，不是 OAuth/OIDC。本次不交付 Gitea、远程 MCP、官网内置模型或真实 Agent 兼容认证。

## 2. 隔离本地运行

从本仓根在新的 Git Bash/Linux 终端执行。Python 3.10+，成员环境独立于考试；配置变量须在导入本仓 `server/app/config.py` 前设置。

```sh
python -m venv .venv-member
# Windows Git Bash；Linux 使用 source .venv-member/bin/activate
source .venv-member/Scripts/activate
python -m pip install -r server/requirements.txt
export MEMBER_PLATFORM_DATA_DIR="$PWD/.local-member"
export MEMBER_PLATFORM_DB_PATH="$MEMBER_PLATFORM_DATA_DIR/member.db"
export MEMBER_PLATFORM_ROLE=authoritative
export MEMBER_PLATFORM_PUBLIC_PREFIX=''
# 仅首次初始化生成：在受控秘密管理位置保留，重启必须恢复同一值
export MEMBER_PLATFORM_SECRET_KEY="$(python -c 'import secrets; print(secrets.token_urlsafe(48))')"
(cd server && python scripts/bootstrap_member.py --student-id local-owner)
```

bootstrap 交互读取两次唯一临时管理员密码（不少于 12 字符），只允许空成员库执行，不通过参数/文件传密码。没有默认管理员、默认密码或自助注册。初始化成功后，同一终端执行：

```sh
(cd server && python -m uvicorn app.main:app --host 127.0.0.1 --port 8902)
```

成员配置只读取 `MEMBER_PLATFORM_*`；旧 `ALE_THIC_*`、`SECRET_KEY` 不作为配置来源。独立成员密钥必须不少于 32 字符且不同于考试密钥，**首次生成后必须保留，重启恢复同一值**以及相同 DATA_DIR/DB_PATH，再只执行 uvicorn。不重新生成密钥、不重做 bootstrap；更换密钥会使已有密码验证失败。密钥存受控秘密管理位置，不打印、提交、粘贴给 Agent。PowerShell 与全部配置见 [成员服务 README](../server/README.md)。

只使用本成员产品的新开发目录，不指向考试数据目录、生产镜像或备份，不复制真实成员。成员应用工作目录为本仓 `server/`，不设置指向考试 checkout 的 `PYTHONPATH`。提交前检查状态，禁止提交真实 DB/WAL、凭据和日志。

先访问 `http://localhost:8902/login`，以显式 bootstrap 的 `local-owner` 和临时密码登录，**必须先改密**才能查询组织 API/管理成员；再进入 `/org`。`/openapi.json` 是运行时契约。隔离应用不加载考试报名、交卷、staging、归档或批次回收；`/org` 禁止 iframe 嵌套，组织错误保持 envelope/request_id。成员实例角色用 `MEMBER_PLATFORM_ROLE`，不是 `ALE_THIC_ROLE`；生产拓扑另审。

考试服务启动遵循独立考试 checkout 的 README，不从本仓启动考试。双服务的 DB、数据目录、密钥和会话必须独立；不能以两个端口共用同库假装独立。可选双服务自动验收只通过显式 `EXAM_PLATFORM_SOURCE` 选择考试源码，详见第 6 节；日常成员测试不需要考试源码。

## 3. CLI 安装与登录

Python 3.10+，在成员虚拟环境从仓库根安装同一发行包，提供两个正式入口：

```sh
python -m pip install ./infra/interface/alethicinsight-cli
# 每次演练新建仓库外隔离目录，不读取或复制任何历史凭据
export ALETHICINSIGHT_CONFIG_DIR="$(python -c 'from pathlib import Path; import tempfile; root = Path.cwd().resolve(); base = Path(tempfile.gettempdir()).resolve(); assert base != root and root not in base.parents; print(tempfile.mkdtemp(prefix="alethicinsight-member-", dir=base))')"
test -n "$ALETHICINSIGHT_CONFIG_DIR" || exit 1
export ALETHICINSIGHT_API_BASE_URL=http://localhost:8902
alethicinsight-cli --help
alethicinsight-cli auth login
alethicinsight-cli auth status --json
alethicinsight-cli projects list --recruiting --limit 20 --json
alethicinsight-cli knowledge search '成员' --json
alethicinsight-cli applications list --mine --limit 20 --json
```

新配置使用 `ALETHICINSIGHT_API_BASE_URL` 与 `ALETHICINSIGHT_CONFIG_DIR`；旧 `ALETHIC_API_BASE_URL`、`ALETHIC_BASE_URL`、`ALETHIC_CONFIG_DIR` 仅兼容。显式服务根不能省略，不依赖旧考试 URL 默认值。`/api/v1/auth/cli/*` 路由不因移动包目录更名。

CLI 随机 request_id，引导可信成员 `/org?cli_request=...`。用户核对 scopes 后取一次性 code，在 CLI 隐藏提示输入；密码、网页 token、code、CLI token 和确认 token 不进 Agent 上下文。默认授权只读，**读取 forms schema 与校验/草稿一样，需要 `applications:create`**，不是只读登录后的可用命令。本人准备申请时先重新明确授权，完成网页同意与 CLI 交换后再查 schema，不放宽服务端权限：

```sh
alethicinsight-cli auth login --scope profile:read:self --scope applications:create --scope applications:read:self --scope projects:read
alethicinsight-cli forms schema project-join --json
```

新的 login scope 列表不会与旧 grant 自动合并；仍需知识查询时显式加 `--scope knowledge:read`。读到当前 schema 后，再准备本地脱敏 `application.json`，project_id/revision 以实际返回为准：

```json
{"kind":"project-join","form_revision":"1","fields":{"project_id":"LOCAL_TEST_PROJECT_ID","motivation":"希望参与本地测试项目并验证加入流程","weekly_hours":4,"notes":"仅本地演练"}}
```

```sh
alethicinsight-cli applications submit --file application.json --dry-run --json
alethicinsight-cli applications draft --file application.json --json
```

`--dry-run` 只 validate；draft 持久化未提交草稿。CLI `applications submit` 也只准备草稿并返回需网页确认（退出码 7），没有 `--confirm`、最终提交或 review。用户必须在可信 `/org?draft_id=...` 重新核对，网页取得一次性 confirmation 后提交 `draft_id + confirmation`，不是再传 fields 或旧嵌套 confirmation_token。

分页只取一页，后续用 `--cursor`，页大小 1..50。`auth logout` 尝试 `DELETE /auth/cli/grants/self` 自撤销并清除本地凭据，只有确认响应才可认定服务端撤销；网络失败时到可信网页撤销或等过期。不能将凭据目录放仓库、共享目录或备份中。

## 4. 本地 stdio MCP 入口

同包安装后已有 MCP，不安装第二发行包。Host 使用刚才授权的同一成员服务根与隔离目录；以下路径需由操作者本地填写，JSON 不展开 shell 变量：

```json
{
  "mcpServers": {
    "alethicinsight": {
      "command": "/absolute/path/to/member-venv/bin/alethicinsight-mcp",
      "args": ["--cli", "/absolute/path/to/member-venv/bin/alethicinsight-cli"],
      "env": {
        "ALETHICINSIGHT_API_BASE_URL": "http://localhost:8902",
        "ALETHICINSIGHT_CONFIG_DIR": "/absolute/path/to/the-new-isolated-directory-from-section-3"
      }
    }
  }
}
```

Windows 使用 `Scripts/alethicinsight-mcp.exe` / `Scripts/alethicinsight-cli.exe`。不要在 env 写 token，不复用默认或历史凭据目录，不把 shell 字符串/非可信程序交给 `--cli`。本地 stdio stdout 专供协议，不启动远程 HTTP MCP。工具只允许发现、读取与准备草稿，不包括确认签发、最终提交、review、SQL、shell 或任意 URL 抓取。

协议自动测试不等于真实 Host 接入。支持版本以本仓包实现、`--help` 和真实握手为准；具体 Host/版本与结果分别记录。参见 [CLI/MCP](agent-interface/docs/05-cli-mcp-skills.md)。

## 5. 受控知识同步

同步仅处理人工审核的本地公开 checkout，不在 HTTP 请求内爬取，不读取私有仓或生产资料。脚本为本仓 `server/scripts/sync_org_knowledge.py`，只操作当前配置的独立成员 DB，不运行考试脚本。使用 `governance-public` allowlist、UTF-8 `.md/.txt`、路径/数量/体积限制，拒绝 restricted、隐藏文件和路径逃逸。

保持第 2 节同一成员 DB 环境，在空开发库完成初始化后：

```sh
python server/scripts/sync_org_knowledge.py --help
python server/scripts/sync_org_knowledge.py --local-dir /path/to/reviewed/governance-public --revision FULL_40_HEX_COMMIT
# 覆盖名单须另行人工审查，再加 --allowlist /path/to/approved-manifest.json
```

```json
{"source_repo":"governance-public","documents":[{"path":"docs/zh-CN/membership.md","title":"成员制度","access_level":"public"}]}
```

来源路径、许可、完整 commit 和 checkout 状态由操作者核实；脚本 revision 格式校验不证明 remote/commit 一致。同步按来源清单 upsert 并删除未列出的旧索引，**空清单会清空该来源**；先备份明确的本地成员库，审阅清单，失败不可部分写入。完成后用成员 CLI 验证来源、版本、可见性；撤除文档不能留下旧摘要。这不是全仓镜像或真实知识导入已完成的声明。

## 6. 测试与验收记录

从本仓根目录、成员开发环境执行，标准成员测试不需要考试源码：

```sh
(cd server && python -m pytest tests -q)
(cd infra/interface/alethicinsight-cli && python -m pip install '.[test]' && python -m pytest -q)
python -m pip wheel --no-deps ./infra/interface/alethicinsight-cli --wheel-dir /path/to/new-temporary-wheel-dir
```

跨产品 HTTP 测试单独在 `integration/test_cross_product_http.py`，另建集成环境安装两边依赖，并显式选择考试 checkout 根；不纳入 `server/tests/`，不猜测邻仓：

```sh
export EXAM_PLATFORM_SOURCE='/absolute/path/to/alethicinsight-platform'
python -m pip install -r server/requirements.txt
python -m pip install -r "$EXAM_PLATFORM_SOURCE/server/requirements.txt"
python -m pytest integration/test_cross_product_http.py -q
```

本页只给操作步骤，不声明本次已运行或通过。来源记录集中在 [migration.md](migration.md)，不继承其通过数。每次验收记录 commit、系统/依赖、实际命令、退出状态、失败/跳过；完整门禁与 PowerShell 命令见 [09 · 验收](agent-interface/docs/09-roadmap-and-acceptance.md)。

- 成员进程无考试业务或维护任务，考试进程无 `/org` 或组织 `/api/v1`。
- 双空库、独立密钥与合成账号同时启动，账号、session/grant 和 admin 权限不跨产品；初始化/维护不改变另一 DB。
- 成员真实 HTTP + 实际 CLI/MCP 子进程闭环、撤权/草稿确认/幂等及权限负例；干净 wheel 安装和两个正式入口。
- YAML 解析、内部 `$ref`、operationId/路径参数、本地 Markdown 链接，以及人工契约对照成员运行时 `/openapi.json`。
- 人工浏览器确认、两种真实 Agent、独立 POSIX、硬件、离机备份/恢复和生产 TLS/反代分别记录，不能由 HTTP 或协议自动测试替代。

## 7. 备份与隔离恢复（未执行的人工方案）

成员备份集合仅含本产品 DB、必要文件、知识来源清单/revision 与受保护配置；考试 DB、考生清洗件/附件/归档另按考试策略备份，不把混合旧库直接复制为成员库。SQLite 使用 backup API/`.backup` 得到一致副本，活跃写入时不得遗漏 WAL；跨文件一致性需要经批准停写窗口。

备份加密、限制访问、离机保留，记录代码/schema、摘要、时间、RPO/RTO 与责任人，不提交 Git、不备份 CLI 明文凭据。恢复到新隔离目录，不覆盖原件，核验 integrity_check、成员权限和事务闭环；不得连到考试库。恢复快照可能复活会话、grant、code 与 confirmation，重新开放前必须按审查过的流程使旧凭据失效并要求重新登录。

真实混合库中的成员身份如何迁出、哪些历史记录可迁、密码与授权如何失效，均须另立数据迁移方案，当前未执行。不提供生产覆盖/DELETE 命令。

## 8. 发布前与回滚（未执行）

先完成独立成员仓标准回归、显式跨仓集成及合成数据验收，再经授权设计成员独立部署、私有网络/域名、TLS、反代、配置/秘密管理、审计与备份。不运行考试部署脚本，不把考试 edge 当成员写入权威，也不以本次文档授权 nginx reload。

源码回退走新的 revert PR；不 reset/clean 或覆盖原脏工作区。运行回退需记录旧成员 release、schema 兼容与一致快照，先停新写入和授权；不能为了代码回退 DROP 表或覆盖考试 DB。不兼容迁移需另审维护窗口、故障态备份、数据损失/补录策略与恢复后凭据撤销。真实数据迁移与生产发布都是独立审批事项，本次未执行。
