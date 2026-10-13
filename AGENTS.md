# AGENTS.md

面向在 open·kritt 仓库中工作的 AI/代码代理（以及想快速了解要点的真人）的指南。这是权威的代理指南；`CLAUDE.md` 指向此处。

完整的贡献者流程见 [`CONTRIBUTING.md`](CONTRIBUTING.md)——本文件是快速版，只讲该做和不该做。

## 这是什么

open·kritt 是一个 AI 驱动的安全漏洞扫描器。它是一个由多个协作服务组成的**多语言 monorepo**：

| 路径             | 内容                                          | 技术栈                         |
| ---------------- | --------------------------------------------- | ----------------------------- |
| `frontend/`      | Web UI                                        | React 18 + Vite (npm)         |
| `backend/`       | HTTP API                                      | Node + Express + Prisma (npm) |
| `engine/`        | 运行 AI 测试框架的扫描 worker                  | Python 3.11+ (Ruff)           |
| `database/`      | Postgres 镜像 + `init/*.sql` 迁移              | Postgres 16                   |
| `executor-view/` | Executor 查看器                               | Python                        |
| `scripts/`       | 仓库工具和 CLI 实现                           | Node                          |

整个产品以**单一版本**发布——即仓库根目录的 [`VERSION`](VERSION) 文件。

## 运行 / 构建 / 测试

最快的全栈启动方式：`cp .env.example .env && docker compose up --build`
（加上 `-f docker-compose.dev.yml` 可获得人类可读的后端日志）。

如需引导式本地入门，运行 `./kritt` 打开全屏方向键菜单，或直接使用
`./kritt setup` 和 `./kritt start`（需要 Node.js 20+；CLI 无需 npm install）。

各组件：

```bash
# frontend
cd frontend && npm install
npm run dev | npm run build | npm run lint | npm run format | npm test   # Vitest

# backend  (needs DATABASE_URL; see .env.example)
cd backend && npm install
npx prisma generate && npm run migrate     # apply database/init/*.sql (idempotent)
npm run dev | npm run lint | npm run format | npm test                   # node:test

# engine
cd engine && pip install -r requirements.txt
ruff check . | ruff format . | pytest
python -m open_kritt_engine                # run the worker
```

## 内部规则（该做）

- **保持改动聚焦。** 不要把无关修改混进 diff；行为或配置变更时，同步添加或更新测试和文档。
- **每次提交都使用 Conventional Commits**：`feat:`、`fix:`、`docs:`、`refactor:`、
  `test:`、`chore:`、`ci:`。Scope 可选（`feat(frontend): …`），但空 scope 无效：用 `feat:` 或 `feat(frontend):`，绝不用 `feat():`。破坏性变更用 `!` 或 `BREAKING CHANGE:` 页脚标记。发布影响为：
  - `feat` → 次版本发布（项目处于 1.0 之前时为补丁版本）
  - `fix` → 补丁版本发布
  - 破坏性变更 → 1.0 之前为次版本发布
  - `docs`、`chore`、`refactor`、`test`、`ci` → 单独不触发发布
- **提交改动前先 lint 和格式化**（JS 用 ESLint/Prettier，Python 用 Ruff）。遵循现有风格；不要重新格式化无关代码。
- **绝不提交密钥或本地数据：** 凭据、令牌、`.env`、服务商登录数据或扫描输出。使用提供的 `gitleaks` pre-commit Hook。
- **数据库变更必须是增量、幂等、仅向前的**：新建 `database/init/NNN_*.sql`，使用 `CREATE ... IF NOT EXISTS` / `ADD COLUMN IF NOT EXISTS`，然后更新 `backend/prisma/schema.prisma` 保持一致，并运行迁移两次。CI 也会运行 `migrate.js` 两次以验证幂等性。
- **一个版本，一个事实来源**：正常贡献不升级版本。如果明确要求手动改版本，修改 [`VERSION`](VERSION)，然后运行 `node scripts/sync-version.mjs`。绝不要手动编辑组件的版本。
- **固定 Docker 镜像标签**（绝不用 `:latest`）。

## 贡献流程

开始之前：

- 提交 bug 或功能请求前，先搜索现有 Issue。
- 问题、想法和较大提案请使用 GitHub Discussions，避免重复工作。
- 绝不公开报告漏洞。使用 [`SECURITY.md`](SECURITY.md) 中的私密报告流程；如果不可用，开一个不含任何漏洞细节的公开 Issue 并请求私密渠道。
- 外部贡献者应 Fork 仓库并在分支上工作。`feat/my-change` 这类分支名只是示例，不是强制命名规则。

提出改动前，对每个涉及的领域运行检查：

| 领域 | 必需的本地检查 |
| ---- | --------------------- |
| 前端 | `cd frontend && npm install && npm run lint && npm run format:check && npm test && npm run build` |
| 后端 | `cd backend && npm install && npx prisma generate && npx prisma validate && npm run lint && npm run format:check && npm test` |
| 引擎 | `cd engine && pip install -r requirements.txt && ruff check . && ruff format --check . && pytest` |
| CLI | `node --test scripts/kritt.test.mjs scripts/kritt-ui.test.mjs scripts/kritt-headless.test.mjs` |
| 文档 | `cd docs-site && npm run check-links` |

引擎测试目前仅在本地运行；CI 不运行它们。跨组件边界的改动，还需运行推荐的全栈冒烟测试：

```bash
docker compose up --build
```

向 `main` 提交 Pull Request。保持小而完整，并填写 Pull Request 模板：说明改了什么、为什么改，链接相关 Issue（例如 `Closes #123`），标明变更类型和受影响的组件，列出已运行的检查，为可见 UI 变更附上截图，指出破坏性变更，并留下有用的审查者备注。PR 类型应与 Conventional Commit 类型一致。合并前 CI 必须通过。

仓库提供可选的 pre-commit 自动化：

```bash
pipx install pre-commit
pre-commit install
pre-commit run --all-files
```

它检查常见文件卫生、合并冲突标记、超过 512 KiB 的文件、YAML/JSON、LF 行尾、密钥、Ruff、Prettier、ESLint 和版本同步。

## 贡献条款（提交前请阅读）

任何有意向官方仓库提交纳入的已认证提交——包括 Pull Request、提交、补丁或 Issue——均构成对具有法律约束力的 [`CONTRIBUTION_TERMS.md`](CONTRIBUTION_TERMS.md) 的同意。无需 DCO 签署、CLA 尾注、单独签名或特殊法律提交语法。

简言之，贡献者将其可转让的贡献权利共同转让给 Harel Rom 和 Gabriel Balko，并授予/放弃这些条款中描述的额外权利。即使贡献随后被拒绝、关闭或撤回，提交仍然有效。只提交你有权贡献的作品；不要提交由雇主、客户、大学或其他协议拥有或限制的作品。标明所有第三方材料、其来源和许可证。贡献随项目以 GNU AGPL v3.0 分发。本摘要不能替代阅读条款。

## 注意事项（别踩坑）

- **Prisma 引擎是平台特定的。** 在 macOS 上生成的 `node_modules` 无法在 Linux 上运行 Prisma（反之亦然）。不要用为其他操作系统构建的客户端运行后端/数据库代码；在目标平台上用 `npx prisma generate` 重新生成。
- **前端原生可选依赖。** Rollup 和 esbuild 按操作系统、架构和 libc 分别发布包。只从干净的依赖图（没有旧 lockfile 或 `node_modules`）重新生成 `frontend/package-lock.json`，以便 npm 记录每个平台包。CI 同时运行 `npm ci` 和 Alpine/musl 镜像冒烟测试，以捕获被平台裁剪的 lockfile。
- **构建时的前端版本**来自 `frontend/package.json`（与 `VERSION` 保持同步），而非 `../VERSION`——仓库根目录不在前端 Docker 镜像中。
- **引擎测试目前在 CI 中禁用。** 有几个假设 `/root` 主目录可写（`~/.codex`、`~/.claude`），还有一个断言已过时；它们在 CI runner 上会失败。修复前引擎仅做 lint。
- **扫描/富化结果中的保留键模式**是有意义的，不是随意的：`_reserved_report` / `_reserved_poc`（渲染为 markdown 标签页）和 `_chip_*`（脚本后 chip）。不要重命名或删除它们。
- **自定义、无依赖的 Markdown 渲染器**位于 `frontend/src/components/Markdown.jsx`（转义不受信任的报告/PoC 文本）。优先使用它，而不是添加 markdown 依赖。
- **这是一个安全工具。** 它在不受信任的代码上运行 AI 代理，并持有服务商 API 密钥 / `GITHUB_TOKEN`。绝不提交密钥（`gitleaks` pre-commit Hook 有帮助），并小心任何改变离开本机数据的改动。

## CI / 合并流程

- **CI** 在向 `main` 提交的 PR 上运行（`frontend`、`backend`、`engine`、`database` 和版本同步）。这是必需检查——`main` 只接收 CI 通过的代码。
- **发布**通过 release-please 在推送到 `main` 时运行（见 [`RELEASE.md`](RELEASE.md)）。
- Lint/format 步骤目前为 `continue-on-error`，直到一次性清理 PR；保持新代码干净，以便它们能成为硬性门禁。

## 去哪里找

- 产品/开发流程：[`CONTRIBUTING.md`](CONTRIBUTING.md)
- 发布与版本管理：[`RELEASE.md`](RELEASE.md)
- 安全策略与威胁模型：[`SECURITY.md`](SECURITY.md)