# 显式跨仓 HTTP 组合验收

成员平台与考试平台是独立仓库、独立服务。`test_cross_product_http.py` 不属于普通成员测试；只有明确提供考试平台源码 checkout 时才运行。CLI/MCP 仍为成员仓内的内部接口包，不需要第三个仓库。

## 前置条件

- 使用已具备成员仓 `server/requirements.txt`、考试仓 `server/requirements.txt` 依赖的 Python 环境；先核对现有环境，不要为此测试盲目安装依赖。
- 普通成员测试包含 Node.js 驱动的前端检查，需要 Node.js；CLI wheel 验收需要现有 `setuptools>=68`、`wheel`、`pip` 和 Python `venv` 支持。
- `EXAM_PLATFORM_SOURCE` 必须指向考试平台（`recruitment-exam-platform`）的独立源码 checkout **根目录**，该目录下应存在 `server/app/main.py`、`server/app/routes/campaigns.py` 和 `server/app/routes/exam.py`。不能指向成员仓或考试仓的 `server` 子目录。
- checkout 只用来读取/运行源码，不使用实际服务、账号、数据库或生产密钥，也不执行部署。

以下命令都从本成员仓根目录运行。

## 普通成员与 CLI/MCP 验收

这两组测试不要求 `EXAM_PLATFORM_SOURCE`，也不默认寻找其他仓库：

```sh
python -m pytest -q server/tests
python -m pytest -q infra/interface/alethicinsight-cli/tests
```

成员测试默认使用本仓 `infra/interface/alethicinsight-cli`。不要在普通验收中设置 `MEMBER_PLATFORM_TEST_CLI_SOURCE` 指向其他 checkout。

CLI 全套测试已包括 wheel 独立安装：将包源码复制到临时目录，使用 `pip wheel --no-index --no-deps --no-build-isolation` 构建，再在全新 venv 中使用 `pip install --no-index --no-deps` 安装 wheel。从源码目录之外运行 `alethicinsight-cli`、`alethicinsight-mcp` 两个 console entry，检查元数据、缺少配置时的错误、MCP 协议和真实 loopback HTTP 请求，不借用源树导入。可单独重跑已安装 wheel 相关用例：

```sh
python -m pytest -q infra/interface/alethicinsight-cli/tests/test_m36_org_mcp.py -k installed
```

## 跨仓真实 HTTP 验收

Git Bash / POSIX shell 示例（将路径换成自己的考试 checkout）：

```sh
EXAM_PLATFORM_SOURCE='/path/to/recruitment-exam-platform' \
  python -m pytest -q integration/test_cross_product_http.py
```

PowerShell 示例：

```powershell
$env:EXAM_PLATFORM_SOURCE = 'C:/path/to/recruitment-exam-platform'
python -m pytest -q integration/test_cross_product_http.py
Remove-Item Env:EXAM_PLATFORM_SOURCE
```

未设置、空值或错误的 `EXAM_PLATFORM_SOURCE` 会明确报错，不会 skip，也不会回退搜索相邻仓库。直接运行仓根的无参数 `pytest` 会收集 integration，因此应选择上述独立测试路径；若要一次运行所有测试，必须显式配置考试源码路径。

测试在 pytest 临时目录中创建两套合成数据库和随机密钥，通过各自独立的进程、工作目录、`PYTHONPATH` 和环境变量启动两个 loopback HTTP 服务。子进程不继承调用者的业务环境配置。验收覆盖账号初始化、首次修改密码、成员账号管理、跨产品 token 拒绝、路由边界、数据库表隔离，以及停止/重启任意服务不影响另一服务。测试结束时终止子进程；不会读取 checkout 中的数据目录或 `.env`。
