# open·kritt 文档站点（Mintlify）

本文件夹是一个自包含的 [Mintlify](https://mintlify.com) 文档项目。内容为 Markdown/MDX，即使日后更换工具也能保持可移植性。

## 本地预览

```bash
cd docs-site
npm i -g mint      # Mintlify CLI
npm run dev        # 在 http://localhost:3001 提供服务
npm run check-links    # 无依赖的内部路由与重定向检查
npm run broken-links   # Mintlify 的链接检查器，包含重定向目标
```

仓库的开发脚本将文档固定到端口 **3001**。直接运行 `mint dev` 仍使用 Mintlify 默认端口 **3000**；如需临时使用其他端口，请用 `mint dev --port <n>`。

品牌资源位于 `logo/mark-accent.png`（导航栏 logo，强调色 `#ff5c3d`）和 `favicon.svg`；`style.css` 通过 Mintlify 的 `nav-logo` Hook 放大导航栏 logo。

## 连接与发布

1. 在 [mintlify.com](https://mintlify.com) 注册（**Starter/免费**方案即可——我们不用 AI 助手）。
2. 在 `Kritt-ai/open-kritt` 上安装 **Mintlify GitHub app**，并指向本 `docs-site/` 目录。每次推送到 `main` 都会重新部署。

## 自定义域名 —— `docs.kritt.ai`

1. 在 Mintlify 控制台 → **Settings → Custom domain** 中添加 `docs.kritt.ai`。
2. 在 DNS 中为 `docs.kritt.ai` 添加一条 `CNAME`，指向 Mintlify 提供的目标（`<your-subdomain>.mintlify.site`）。

若想改为托管在 **`kritt.ai/docs`**（同一域名），请在控制台启用 **"Host at /docs"**，并在主站添加反向代理规则（Mintlify 已为 Cloudflare Workers / Vercel / nginx 提供现成配置）。普通 `/docs` 路径在非 Enterprise 版可用；任意子路径（如 `/help`）仅限 Enterprise 版。

## 编辑

- 页面以 `.mdx` 形式存放于此；导航与主题在 `docs.json` 中。
- 重命名的页面必须在 `docs.json` 中保留永久重定向；CI 会拒绝失效的内部链接、缺失的导航页面以及无效的重定向目标。
- 安全指南链接到仓库中的权威文档（`docs/threat-model.md`、`SECURITY.md`）。若我们全面采用 Mintlify，需确定单一事实来源以避免内容漂移。