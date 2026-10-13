# 为 open·kritt 做贡献

感谢你帮助改进 open·kritt。参与时请遵守我们的
[行为准则](CODE_OF_CONDUCT.md)。

## 开始之前

- 报告 bug 或请求新功能前，先搜索[现有 Issue](https://github.com/Kritt-ai/open-kritt/issues)。
- 提问、提想法和较大的提案请用 [Discussions](https://github.com/Kritt-ai/open-kritt/discussions)。先在那里讨论可避免重复劳动。
- 切勿公开报告漏洞。请按 [SECURITY.md](SECURITY.md) 私下提交。

## 搭建项目

你需要 Git、带 Docker Compose 的 Docker，以及 Node.js 20 或更高版本。Fork 仓库，然后：

```bash
# 1. Get the code
#    External contributors: fork on GitHub, then clone your fork:
git clone https://github.com/<you>/open-kritt && cd open-kritt

# 2. Configure model access and bring the whole stack up
./kritt setup
./kritt start

# 3. Make changes on a branch and use Conventional Commits
git checkout -b feat/my-change
git commit -m "feat(frontend): add X"

# 4. Push and open a PR against main. CI must pass before it can merge.

```

这些命令会配置环境并启动整个技术栈。详见
[安装指南](https://docs.kritt.ai/getting-started/installation-and-setup)。

## 修改并验证

每次改动保持聚焦。行为变化时补充或更新测试和文档，然后对你触及的每个区域运行检查：

| 区域 | 检查 |
| --- | --- |
| 前端 | `cd frontend && npm install && npm run lint && npm run format:check && npm test && npm run build` |
| 后端 | `cd backend && npm install && npx prisma generate && npx prisma validate && npm run lint && npm run format:check && npm test` |
| 引擎 | `cd engine && pip install -r requirements.txt && ruff check . && ruff format --check . && pytest` |
| CLI | `node --test scripts/kritt.test.mjs scripts/kritt-ui.test.mjs scripts/kritt-headless.test.mjs` |
| 文档 | `cd docs-site && npm run check-links` |

引擎测试目前只在本地运行，未在 CI 中启用。跨组件边界的改动建议做全栈冒烟测试：

```bash
docker compose up --build
```

## 项目规则

- 切勿提交凭据、令牌、`.env` 或本地扫描数据。
- 数据库变更必须在 `database/init/` 中新增编号 SQL 文件。迁移必须增量、只向前，且可重复运行；同步更新
  `backend/prisma/schema.prisma`，并运行两次 `npm run migrate`。
- `VERSION` 是产品版本的唯一来源。普通贡献不应改动它。若确需改版本，编辑 `VERSION` 并运行
  `node scripts/sync-version.mjs`。
- Docker 镜像标签必须固定。
- 改动依赖锁文件、Prisma 配置、结果键或 Markdown 渲染前，先阅读仓库专属的
  [代理与贡献者注意事项](AGENTS.md)。

## 提交改动

每次提交必须使用 [Conventional Commits](https://www.conventionalcommits.org/)：

```bash
git commit -m "fix(frontend): describe the change"
```

常见类型有 `feat`、`fix`、`docs`、`refactor`、`test`、`chore` 和 `ci`。scope 可选；用 `feat:` 或 `feat(frontend):`，不要用 `feat():`。

## 贡献条款

有意提交 Pull Request、提交、补丁或其他贡献，即表示你同意[贡献条款](CONTRIBUTION_TERMS.md)，包括将贡献中可转让的权利共同转让给
[Harel Rom 和 Gabriel Balko](OWNERSHIP.md)。请勿提交雇主、客户、大学或其他方拥有或限制的工作成果。

无需单独签名、贡献者协议、表单、复选框、登记或特殊提交语法。提交贡献这一普通行为即为唯一的同意步骤。

## 发起 Pull Request

推送分支并向 `main` 发起 Pull Request。在 Pull Request 中：

- 说明改了什么、为什么改，并关联相关 Issue。
- 列出你运行的检查。
- UI 可见变化请附截图。
- 无关改动不要放进 diff。
- 填写 Pull Request 模板并确保 CI 通过。

贡献将作为项目的一部分，按仓库的
[GNU AGPL v3.0 许可证](LICENSE)分发。