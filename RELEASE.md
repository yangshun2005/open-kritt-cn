# 发布 open·kritt

open·kritt 是一个多语言 monorepo（前端、后端、引擎、数据库），以**一个产品**、**一个版本号**的形式发布。

## 版本模型

- **唯一事实来源：** 仓库根目录的 [`VERSION`](VERSION) 文件。它带有一个行尾标记 `# x-release-please-version`，让 release-please 自动更新它（见下文）；工具会忽略该标记。
- 所有组件都固定在同一版本：
  - `frontend/package.json` 和 `backend/package.json`（`version`）
  - `engine/open_kritt_engine/__init__.py`（`__version__`）
  - UI 在构建时注入版本（`frontend/vite.config.js` → `__APP_VERSION__`，从同步后的 `frontend/package.json` 读取，因为仓库根目录的 `VERSION` 不在仅含前端的 Docker 镜像中），因此侧边栏始终显示真实版本。如有需要，可用 `APP_VERSION` 环境变量覆盖。
- **发布时，release-please 会一并更新所有版本**（两个 `package.json` 和引擎通过 `extra-files`，外加 `VERSION`），因此它们不会漂移。
- CI 和 pre-commit hook 会运行 `node scripts/sync-version.mjs --check`，一旦版本漂移就让构建失败。

你几乎不需要手动改版本。如果确实需要（例如手动或非周期性的版本更新），编辑 `VERSION` 中的数字，然后运行 `node scripts/sync-version.mjs` 将其传播到其他组件。

## 稳定性策略（1.0 之前）

在 `0.x` 阶段，项目遵循 SemVer，但有 **1.0 之前的例外：次要版本之间任何内容都可能变化**：

- **破坏性变更**（HTTP API 形态、数据库 Schema、环境变量、工作流/步骤语义）可能出现在**次要**版本更新（`0.x.0`）中，并会在 [CHANGELOG](CHANGELOG.md) 的 **Changed** / **BREAKING** 标题下注明。
- **补丁**版本（`0.x.y`）仅包含新增功能或缺陷修复。
- **数据库迁移只向前、且幂等。** 每次变更都是 `database/init/` 中带 `IF NOT EXISTS` 保护的新增文件，由 `backend/prisma/migrate.js` 应用。重复运行迁移必须始终安全（CI 在每个 PR 上验证这一点）。
- 我们将在发布 `1.0.0` 时给出正式的兼容性保证。

## CI / 分支保护

流水线按事件拆分：

- **CI**（`.github/workflows/ci.yml`）在**针对 `main` 的 Pull Request** 上运行。
- **Release**（`.github/workflows/release.yml`）在**推送到 `main`** 时运行。

为保证只有通过 CI 的代码才能进入 `main`（从而保证发布只从绿色构建中切出），请通过分支保护将 CI 设为**必需状态检查**——这是一次性的仓库设置：

**Settings → Branches → 为 `main` 添加分支规则集（或保护规则）：**
- ✅ 合并前要求 Pull Request
- ✅ 合并前要求状态检查通过 → 选择 CI 任务（`frontend`、`backend`、`engine`、`database`、`version sync`）
- ✅（推荐）合并前要求分支为最新
- ✅（推荐）阻止强制推送

开启后，直接推送到 `main` 会被拒绝；变更只能通过合并 CI 为绿色的 PR 进入——随后触发 Release 工作流。

## 一次性仓库设置（发布 PR 权限）

release-please 使用 Actions token 创建发布 PR，而 GitHub 默认会阻止这一行为（*"GitHub Actions is not permitted to create or approve pull requests."*）。选择**一种**方式：

**方案 A —— 仓库/组织设置。** Settings → Actions → General → Workflow permissions → "Read and write permissions" + "Allow GitHub Actions to create and approve pull requests"。

> 如果该复选框**置灰**，说明策略在**组织**级别强制执行。需要组织所有者在该组织的 Actions 设置中启用——或改用方案 B（无需改动组织设置）。

**方案 B —— 提供 token（推荐用于组织）。** 创建一个限定到本仓库的细粒度 PAT，权限为 **Contents: read/write** + **Pull requests: read/write**（或 GitHub App token），并将其添加为仓库密钥 **`RELEASE_PLEASE_TOKEN`**。`release.yml` 已优先使用该密钥，回退到默认 token，因此无需修改工作流。额外好处：由 PAT/App 创建的发布 PR 也会触发 CI（默认 Actions token 不会）。

## 提交约定

发布由 [Conventional Commits](https://www.conventionalcommits.org/) 自动完成：

- `feat: …` → 次要版本更新（1.0 之前：补丁——见 `release-please-config.json`）
- `fix: …` → 补丁版本更新
- `feat!: …` / `BREAKING CHANGE:` → 次要版本更新（1.0 之前）
- `docs:`、`chore:`、`refactor:`、`test:`、`ci:` → 自身不触发发布

> **Scope 可选，但绝不要用空括号。** `feat():` 是*无效*的 Conventional Commits，release-please 会静默跳过它（`unexpected token ')'`）。写成 `feat:`（无 scope）或 `feat(frontend):`（带 scope）——不要写 `feat()`。

## 切出发布

我们使用 [release-please](https://github.com/googleapis/release-please)（`.github/workflows/release.yml`）。它在**推送到 `main`** 时运行；由于分支保护只允许通过 CI 的 PR 合并进 `main`，发布绝不会从红色构建中切出。

1. 将符合 Conventional Commits 的 PR 合并进 `main`。release-please 会维护一个滚动的 **"chore(main): release x.y.z"** Pull Request，更新所有位置的版本——`backend/package.json`、`frontend/package.json`、`engine/…/__init__.py`，**以及根目录的 `VERSION` 文件**（通过 `x-release-please-version` 标记）——并更新 `CHANGELOG.md`。由于四者同步变更，发布 PR 上的 `versions` CI 检查无需手动步骤即可通过。
2. **合并发布 PR。** release-please 会创建 git 标签（`vX.Y.Z`）和带自动生成说明的 GitHub Release。

> 容器镜像的构建与发布有意**尚未**接入；后续会作为单独的发布步骤添加。

## 手动 / 自动化之前的发布（回退方案）

如果需要手动切出发布：

```bash
# Keep the release-please marker on the line:
echo "0.5.0 # x-release-please-version" > VERSION
node scripts/sync-version.mjs        # propagate to package.json files + engine
# update CHANGELOG.md (move Unreleased → 0.5.0)
git commit -am "chore(release): 0.5.0"
git tag v0.5.0 && git push --tags
```