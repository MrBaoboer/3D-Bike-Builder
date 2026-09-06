# 参与进来

欢迎的：修 bug、改错别字、补浏览器兼容、补无障碍、订正工程数据（规格、螺距、装配顺序）。
订正工程数据请附出处，Issue 模板里有一栏。

先开 Issue 聊过再动手的：加运行时依赖（目前只有 three）、换框架、加新的交互原语、大改课程编排。这几件改动面大，先对齐方向。

## 先跑起来

```bash
npm install && npm run dev
```

动手之前读一遍 [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md)。分层与 ctx 契约、清单、取景、运镜各有一节，改哪一块看哪一节。

## 提交之前

```bash
npm run check:code
```

lint、单测、清单对账、构建。CI 对每个 PR 跑这一条。

- 动了三维、步骤、界面版式的，本地再跑 `npm run smoke`。CI 只在合并到 `main` 之后跑它。
- 改了取景的，`npm run frames` 前后各跑一遍对着看。
- 改了外观的，`npm run build && npm run shots` 重拍截图，附在 PR 里。

不手改 `dist/`。

## 提交与 PR

提交信息与 PR 标题用中文，一句话说清做了什么，不用前缀与 emoji。
PR 正文三段：改了什么、为什么、怎么验的。第三段写上一节里你真正跑过的那几条。

## 授权

提交贡献即表示：你保留自己的著作权，并授予维护者在 AGPL-3.0（代码）与 CC BY-NC-SA 4.0（内容）框架下使用的权利，包括商业授权。
不同意请在 PR 里说明。三层授权的划分见 [COMMERCIAL.md](COMMERCIAL.md)。
