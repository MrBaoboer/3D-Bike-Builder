# 验证

`tools/` 里的脚本分两类：门禁回答「有没有坏」，CI 与提交前跑；量尺回答「现在是多少」，改了相关代码前后各跑一遍对着看。

| 命令 | 类别 | 抓什么 |
|---|---|---|
| `npm run lint` | 门禁 | ESLint recommended |
| `npm test` | 门禁 | `tools/unit.test.mjs` 校验器纯函数的边界；`tools/bom.test.mjs` 对角配对、拓扑排序、向量冻结。这些出错都不在浏览器里报错，只表现为画面异常 |
| `npm run verify` | 门禁 | 清单 × GLB 对账，查什么见 [manifest.md](manifest.md#清单) |
| `npm run smoke` | 门禁 | 浏览器里才现形的错：契约漂移、取景对空、原语没跑起来 |
| `npm run frames` | 量尺 | 逐步取景：主体偏心、整车照贴边 |
| `npm run shots` | 量尺 | 重拍 README 的六张截图 |
| `npm run live` | 量尺 | 线上两处是不是同一份产物，见 [deployment.md](deployment.md) |

`smoke`、`frames`、`shots` 都用 `tools/serve.mjs` 起 `dist/`，先 `npm run build`。

## 冒烟走查

`npm run smoke` 不是「翻页看标题」，它必须真的把车装一遍。每一步查：可达、有标题、声明了 `cam.fit`、这一步瞄的那个点投影后落在界面没遮住的那块画面里、画面不是纯色。
然后**从头到尾按用户那条路把整台车装完**（一步步按「下一步」，每一步把剩下的活按「一下一件」演完），再对账：

- 每一件精确落回装配位（偏差 < 0.01 mm），二十七件七颗都记上账，自检三十四行报「全部到位」
- 在场件数一路只增不减，每一件进场时它声明的前置件都已在车上
- 四颗面盖要按四下，第五下才翻页；按对角拧算通过，拧相邻的当场记下不合格
- 左脚踏往正牙方向拧：转得动、拧满两圈停住、回退半圈再说明白；右牙倒转两圈照样顶住回弹，但不记成拧反
- 侧着推顶得住：件歪出去一点点就到头，松手摆正并说明为什么
- 整车那四张成品照完整落在画幅内；摊开那一步二十七件每一件都露得出来，指到哪件报哪件的名字
- 换步一律走一段运镜，转过六十度以上的那几趟走的路程长于两点直线；深浅两套主题都换得动；方向键能前进后退；全程控制台没有报错与 4xx/5xx

`npm run smoke -- --shots` 每一步都截图到 `.shots/smoke/`，不加只截失败那一张；`--headed` 开着浏览器跑。CI 比开发机慢一个量级，所有等待都要过 `tmo()`。

**凡是有对错之分的交互，正例反例都要断言。** 只查「拧过了」不够，还要查「拧对了没有」；只查「步骤可达」不够，四个交互原语都得真的走一遍，`enter()` 一进去就抛错的步骤，「可达」照样报绿。
`slide` / `screw` 的自动路径不走负角与侧向，「错方向」那半边只有真发指针事件才测得到：用 `page.mouse` 真拖，抓点必须落在件身上的真实采样像素，拿包围盒形心会落进空处。

改 `tools/smoke.mjs` 之前先读这两条：

- 判「画面不是纯色」不能直接 `drawImage` WebGL 画布。没开 `preserveDrawingBuffer` 时那块缓冲在合成后就清了，读回来永远全黑。要在同一个任务里先 `render()` 再取样。
- 断言「镜头没对着空处」要投影**步骤声明的 `cam.target`**，不能投 `controls.target`：后者是让位之后的机位目标，按定义就在画幅正中。

无头浏览器里别逐帧测相机速度，理由见 [camera.md](camera.md#主循环的时间量)。

## 取景对账

`npm run frames`（`tools/frame-audit.mjs`）逐步量两把尺子：**画面**是全部非背景像素的外接框，**主体**是这一步声明的 `installs` / `fastens` 按进场那一刻投影的位置，偏心以可用画面的半宽 / 半高为 1。
只量画面抓不出主体跑偏：主体偏出一个 gap 时，整幅非背景外接框几乎不动。主体走投影不走像素：主体常与车身同色同质（黑油管贴黑碳纤维），逐像素分不出哪一块是它。

默认画幅 1440 × 900，换画幅跟两个参数：`node tools/frame-audit.mjs 390 844`。主体偏心超过 0.30、整车照贴边，都退 1。
改了取景常量，前后各跑一遍对着看，只看截图会漏掉「小了一成」这种量级的退化。

## 截图

`npm run build && npm run shots` 从构建产物实拍六张到 `docs/shots/`，1.5 倍像素密度：整车门面（宽幅、不带界面）、拆开看看、面盖对角拧到一半、出门前自检、深色主题下的左脚踏反牙、手机竖屏。
改了外观就重跑，截图随 PR 一起提交。

## 模型分析

不进门禁，改模型或清单时手动跑：

| 命令 | 做什么 |
|---|---|
| `npm run model` | 装配树概览，折叠叶子，按组汇总面数与件数。`node tools/glb-tree.mjs <glb> [深度]` 可指定展开深度 |
| `node tools/glb-inspect.mjs <glb>` | 只解 GLB 的 JSON 块：节点、网格、材质、贴图、扩展 |
| `node tools/glb-parts.mjs <glb>` | 从 accessor min/max 取每个网格节点的包围盒，按位置归类 |
| `node tools/glb-shells.mjs <glb>` | 三角形按共享顶点并查集分组：焊死的网格只有一两个巨块，能不能程序化拆件看这一条 |
| `node tools/tex-audit.mjs <glb…>` | 逐张贴图列出尺寸与格式；KTX2 变体另报 ETC1S / UASTC 模式 |

## CI

`.github/workflows/check.yml` 分两条作业：

| 作业 | 什么时候跑 | 跑什么 |
|---|---|---|
| 工具链 | 每个 PR、推到 `main` | `npm run check:code`，Node 22.13 / 24 / 26 三格，替 `engines` 那句话作证 |
| 冒烟 | 推到 `main` 之后、手动触发、Dependabot 的 PR | `npm run smoke`，失败时把 `.shots/smoke/` 传成产物 |

冒烟不对普通 PR 跑，一次浏览器走查要等十几分钟。Dependabot 例外：依赖里唯一会真出事的是 three，它的渲染回归只有冒烟看得见。
作业 id 必须是 ASCII，中文 id 会让整个工作流在解析阶段失败，报的是「workflow file issue」。

`.github/dependabot.yml` 把 GitHub Actions 与 npm 合成同一条月度 PR。
CI 上冒烟正常十来分钟；卡在 `playwright install` 那一步远超这个量级是 runner 单机卡死，取消后只重跑那一格即可。
