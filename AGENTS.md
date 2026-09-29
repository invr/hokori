# AGENTS.md — hokori (Astro 博客主题 "Chiri")

## 开发者命令（来自 package.json）

| 命令 | 作用 |
|---|---|
| `pnpm dev` | 启动开发服务器 (`tsx scripts/update-link-metadata.ts && astro dev`) |
| `pnpm build` | **必须按顺序运行**：typecheck → update-link-metadata → astro build |
| `pnpm typecheck` | 运行 `tsc --noEmit` 后再运行 `astro check` |
| `pnpm typecheck:ts` | 仅 TypeScript (`tsc --noEmit`) |
| `pnpm lint` | 在整个仓库运行 ESLint |
| `pnpm lint:fix` | 带自动修复的 ESLint |
| `pnpm format` | Prettier 整个仓库自动格式化 |
| `pnpm format:check` | 检查格式（无修改） |
| `pnpm new <title>` | 通过 `tsx scripts/new-post.ts` 创建新文章 |
| `pnpm update-link-metadata` | 从远程 URL 刷新 link-card 元数据 |
| `pnpm update-theme` | 获取并合并上游主题变更 (git) |

**构建顺序很重要**：`pnpm build` 运行 `typecheck` → `update-link-metadata` → `astro build`。切勿重新排序。

## 单包 / 目录边界

- 单包（无单包工具管理）。所有代码在根目录。
- 关键目录：
  - `src/` — Astro 源代码
  - `src/content/` — MDX 内容集合（文章、关于页）
  - `src/pages/` — Astro 页面入口（包括 `[...slug].astro`、 `rss.xml.ts`、 `open-graph/`）
  - `scripts/` — 实用脚本（new-post、update-*、theme）
  - `assets/` — 静态资源（截图、来自 guizang-ppt-skill 的模板）

## TypeScript & Lint

- `tsconfig.json` 扩展 `astro/tsconfigs/strict`，带 `@/` 路径别名 → `./src/*`
- ESLint 配置 (`eslint.config.js`) 强制执行：
  - `no-console` 仅允许 `warn`、`error`
  - `@typescript-eslint/no-unused-vars` 警告，忽略以下下划线前缀的参数
  - `astro/no-set-html-directive` 禁用
  - Prettier 作为 ESLint rule 集成
- **永远不要在 `pnpm build` 之前跳过 `pnpm format:check` 或 `pnpm lint`**。CI 按 format → lint → typecheck → build 的顺序运行。

## 内容与文章

- `src/content/posts/` 下的 MDX 文章，前置数据包括：`title`、`pubDate`、`image?`
- `src/content/about/` 下的关于页
- **创建新文章**：运行 `pnpm new "<title>"` —— 这会在 `src/content/posts/` 生成带有前置数据（title + 今天日期）的 `.md` 文件。不要手动创建文件；脚本会处理命名和日期。
- Link card 元数据位于 `src/data/link-card-metadata.json`。通过 `pnpm update-link-metadata` 更新。在 MDX 中使用 `::link` 指令，格式如 `{::link {url="https://..."}}`。

## Astro 配置

- `astro.config.ts` 设置：
  - MDX 插件（math、directive、embedded media、TOC、reading time、katex）
  - Sitemap 集成
  - Vite 别名 `@` → `src/`
  - 通过 sharp 的图片服务
  - Dev toolbar 禁用
- **除非添加新集成，否则不要修改 `astro.config.ts` 插件**。插件列表是内容处理的事实依据。

## 关键坑 & 备忘录

1. **无测试套件**——仓库中不存在测试文件。验证通过 `pnpm build` 进行构建，手动预览请使用 `pnpm dev` 然后在浏览器打开。
2. **`pnpm build` 运行 `update-link-metadata` 会抓取远程 URL**——确保有网络访问，否则脚本可能会挂起/超时（每个 URL 10 秒）。使用 `--force` 标志重新抓取现有条目。
3. **主题配置在 `src/config.ts`**——控制站点-wide 设置（title、description、文章选项）。在配置文件中编辑，而非在模板文件中。
4. **`pnpm update-theme`** 使用 `git merge upstream/main --allow-unrelated-histories`。若有本地 git 冲突请勿运行。
5. **图片服务**在 `astro.config.ts` 中通过 sharp 配置——更改图片端点前必须先更新配置。
6. **`no-console` lint 规则**仅允许 `warn` 和 `error`。移除或重命名任何非这两者的 `console.*` 调用。
7. **TypeScript strict mode**——`strictNullChecks`，不允许在没有警告的情况下使用 `any`。尽量少使用类型断言。

## 参考（现有指引来源）

- `README.md`——高层入门、功能、命令
- `.github/workflows/ci.yml`——CI 流水线（format → lint → typecheck → build）
- `package.json`——所有 npm/pnpm 脚本和依赖
- `astro.config.ts`——Astro 配置和插件列表
- `src/content.config.ts`——内容集合 schemas（posts、about）
- `src/config.ts`——主题配置
- `references/`——（如有）任何设计/布局参考

## 陷入困境时

- 运行 `pnpm build` 以验证 typecheck + lint 通过。
- 运行 `pnpm new "<title>"` 创建文章——不要手动编辑 `src/content/posts/` 的命名。
- 首先检查 `src/config.ts` 进行站点-wide 更改。
- 如果链接/元数据看起来陈旧，运行 `pnpm update-link-metadata`。
- 如果主题看起来过时，运行 `pnpm update-theme`。