# 部署

产物纯静态，`base: './'`，放任何子路径下都不用改配置。线上两处发的是同一份产物：

| | 地址 | 怎么来的 |
|---|---|---|
| 主 | <https://build-bike.vercel.app> | 推到 `main` → Vercel 出一次生产部署 |
| 备 | <https://mrbaoboer.github.io/3D-Bike-Builder/> | `.github/workflows/deploy.yml` → GitHub Pages |

GitHub Pages 首次使用前要在仓库 Settings → Pages 里把 Source 选成 GitHub Actions。

## 主地址

主地址必须是项目域名表里的那一条，不要用部署别名。别名指向某一个具体部署，不跟着生产走，每发一版就掉队一次，而且照样返回 200、标题一样。
判断办法：`vercel domains add <名字>.vercel.app <项目>` 能成，它就是项目域名；报 `alias_conflict` 就换个名字，`vercel.app` 前缀全局先到先得。

## 核对

`npm run live`（`tools/live-check.mjs`）比对两处首页里带内容哈希的入口 JS 文件名，一样才是同一份产物，对不上当场退 1。只看「打得开 / 返回 200」不够。

## CSP 与响应头

生产产物注入一条 CSP（`vite.config.js`），策略内容与理由见 [SECURITY.md](../SECURITY.md)。
改它之后**必须重跑 `npm run smoke`**：`connect-src` 少放 `blob:` 的话，GLTFLoader 取不到贴图，而页面看上去还是「能开」。
CSP 只进构建产物，不进 `index.html`：开发期 Vite 靠动态 `<style>` 注样式，`style-src 'self'` 会让 `npm run dev` 白屏。

`vercel.json` 另发一组安全响应头。缓存分两档：`/assets/` 里的产物带内容哈希，给一年 immutable；`/models/` 下那份 GLB 文件名固定、没有哈希，只给七天。
标成 immutable 的话，换了模型也刷不掉浏览器里那份旧的。
