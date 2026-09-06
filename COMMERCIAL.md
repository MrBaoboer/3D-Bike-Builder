# 商业授权

授权分三层。前两层由维护者授权，第三层是第三方素材，维护者无权转授。

## 一 代码：AGPL-3.0

`src/`、`tools/`、`index.html`、构建配置、`.github/` 下的工作流。

这是一个纯前端应用，部署出去，访问者就是接收者。在 AGPL 下，把改过的版本挂到网上供人访问，
就得把改动后的完整源码一并提供给访问者。`dist/assets/` 里压缩过的产物不算对应源码。

## 二 内容：CC BY-NC-SA 4.0

八章二十九步的课程编排、旁白与界面文案、说明卡里的工程解释、`assets/bike.manifest.json` 这份装配清单、
`src/render/bolt.js` 里程序化生成的螺栓与工具几何、`src/audio/sfx.js` 里的音效配方、设计令牌与界面版式。

NC 禁止商业使用。代码即使完全开源，这一层也不允许拿去做商业产品。

## 三 整车模型：CC BY-SA 4.0，第三方

`public/models/CarbonFrameBike.glb`，以及 `docs/shots/` 下含该模型画面的截图。
建模 Robert Schweier，实时化与动画 Felix Herbst / prefrontal cortex，详见 [assets/CREDITS.md](assets/CREDITS.md)。

- 它不是 NC。CC BY-SA 4.0 允许商业使用，条件是署名与相同方式共享。
- 维护者不是版权人，只是按 CC BY-SA 4.0 的条款在用。从维护者这里取得的任何商业授权都不覆盖这个模型。
- ShareAlike 只传染到模型自身及其改作（重新导出、减面、贴图转码），不传染到代码。代码与模型是聚合而非改编。

要商用，最干净的做法是把模型换掉。清单里所有几何量都写着节点名，换一份自有模型，
改的是 `assets/bike.manifest.json` 里的 `nodes` 与坐标，`npm run verify` 会逐条对账。

## 什么时候需要单独谈

- 想闭源运营改动过的版本
- 想把它整进一个不开源的产品里
- 想去掉署名、换成自己的品牌
- 想把课程编排、文案、装配清单用于商业用途
- 所在组织有政策不接受 AGPL

谈成之后能给的是第一层与第二层的商业授权，其中的外部贡献同样可授权（贡献者在 [CONTRIBUTING.md](CONTRIBUTING.md) 里已同意）。
第三层仍要按 CC BY-SA 4.0 处理，或者换掉模型。

## 怎么联系

在本仓库开一个 Issue 说明用途与范围，或在 GitHub 上找 **@MrBaoboer**。
说清楚：用在什么产品上、是否闭源、是否保留署名、打算怎么处理第三层的模型。
