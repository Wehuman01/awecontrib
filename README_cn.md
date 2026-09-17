<div align="center">
  <h1>awecontrib</h1>
  <p><strong>每个仓库一条 verify 命令、一处版本真相。</strong></p>
  <p>
    <a href="./README.md">English</a> ·
    <strong>简体中文</strong>
  </p>
  <p>
    <img src="https://img.shields.io/pypi/v/awecontrib?style=flat-square&color=7C3AED" alt="Version">
    <img src="https://img.shields.io/badge/python-%E2%89%A53.9-0EA5E9?style=flat-square" alt="Python">
    <img src="https://img.shields.io/badge/license-MPL--2.0-22C55E?style=flat-square" alt="License">
  </p>
</div>

> `awecontrib install` 往仓库里放一个 `verify` 文件和一份极简 CI。本地和 CI 从此跑同一条命令，不会再漂移。

## 为什么做

仓库的漂移方式都一样：本地开发跑 `pytest`，release workflow 里跑的却是 `unittest discover`；有测试但 CI 从来不调；版本号散落在两个文件里；`egg-info`/`__pycache__` 悄悄进了 git。解法不是加流程，是每个仓库只留一个入口，你和 CI 都调它。

awecontrib 从 [awe 系列](#实际使用-awe-系列)长出来——那是一批 CLI 和桌面工具，逐仓成长时把上面每个坑都踩了一遍。这个工具留下了教训，去掉了家族绑定：它写进仓库的文件不依赖 awecontrib，也没有命名约定，任何 Python 或 Node 仓库都能用。

## 安装

```bash
pip install awecontrib
```

### Agent 技能

仓库自带配套技能 `resources/skills/awecontrib`，告诉 coding agent 什么时候该用这个 CLI：

```bash
aweskill install Webioinfo01/awecontrib
```

## 命令

### `awecontrib install`

在仓库根目录运行。自动识别 Python（pyproject.toml）和 Node（package.json）；两者都在时用 `--python` 或 `--node` 指定。

- Python 仓库：写入可执行的 `verify`（pytest，仓库配了 ruff 才加 `ruff check .`）、一份只调 `./verify` 的 6 行 `.github/workflows/ci.yml`，并向 `.gitignore` 追加垃圾文件模式。
- Node 仓库：用 package.json 里已有的 typecheck/lint/test 脚本拼出 `scripts.verify`，写入 CI workflow（按 lockfile 识别 npm/pnpm/yarn），追加 `.gitignore` 条目。

已存在的 `verify` 或 CI 文件不会覆盖，除非 `--force`。

### `awecontrib bump <版本号>`

只改版本唯一真相那一处——pyproject 静态 `version`、动态版本的 `src/*/__init__.py` 里的 `__version__`、或 package.json——并在 `docs/CHANGELOG.md` 顶部插入 `## v<版本>` 条目（正文用 `--note`）。不做 git 提交和打 tag，那归你的 release 流程管。

### `awecontrib hygiene [--fix]`

git 里跟踪了垃圾文件（`egg-info`、`__pycache__`、`.pytest_cache`、`.DS_Store`、`*.pyc`、`node_modules`）就报错退出。`--fix` 只打印应执行的 `git rm -r --cached` 命令，不替你删。

## verify 文件

自包含 bash，CI 不需要安装 awecontrib：

```bash
#!/usr/bin/env bash
set -e
cd "$(dirname "$0")"

PY=".venv/bin/python"        # 优先用仓库 venv；CI 里回落到 python3
[ -x "$PY" ] || PY="python3"

"$PY" -m pytest

# hygiene: tracked junk files fail the build
JUNK_RE='(\.egg-info/|__pycache__/|\.pytest_cache/|\.DS_Store$|\.pyc$|^node_modules/)'
...
```

随时可以手改，它是你的文件。如果 release workflow 里还在跑 `unittest discover`，把那一步换成 `./verify`，发布就和开发用同一套检查。

## 实际使用：awe 系列

[awe 系列](https://github.com/Webioinfo01)的每个仓库都跑同一条 `./verify` 门禁——这个工具先在那批仓库上维护，每个项目都是活案例：

| 仓库 | 做什么 | 技术栈 |
|---|---|---|
| [awerouter](https://github.com/Webioinfo01/awerouter) | 把 coding agent 流量在 flash/pro 模型间分流的 LLM 代理 | Python |
| [awecompress](https://github.com/Webioinfo01/awecompress) | 把旧对话轮冻结成缓存摘要的上下文压缩代理 | Python |
| [awewarm](https://github.com/Webioinfo01/awewarm) | 按计划给 AI 编程订阅窗口保温 | Python |
| [aweswitch](https://github.com/Webioinfo01/aweswitch) | 切换 Claude Code、Codex、OpenCode、ZCode 的 API profile | Python |
| [awescholar](https://github.com/Webioinfo01/awescholar) | AI 可操作的学术文献检索与策展 CLI | Python |
| [aweshelf](https://github.com/Webioinfo01/aweshelf) | 给 AI 编程会话打书签、分类、恢复 | Python |
| [aweskill](https://github.com/Webioinfo01/aweskill) | 把 agent 技能投影到 47+ agent 目录的包管理器 | Node/TypeScript |
| [awehitch](https://github.com/Webioinfo01/awehitch) | 通过 MCP 把 ChatGPT 网页当本地编程 agent 的规划大脑 | Node/TypeScript |
| [aweshare](https://github.com/Webioinfo01/aweshare) | 共享闲置 Ollama/vLLM 算力的本地优先中继 | Node/TypeScript |
| [awefork](https://github.com/Webioinfo01/awefork) | 在任意回合 fork AI 编程会话的桌面工作台 | Electron/TypeScript |
| [awedot](https://github.com/Webioinfo01/awedot) | 打书签并恢复编程会话的桌面悬浮球 | Tauri/Rust + TypeScript |

## v1 不做

badge 改写（PyPI/npm badge 本来就是动态的）、git 自动化、批量仓库扫描、monorepo 支持。

## 许可

MPL-2.0
