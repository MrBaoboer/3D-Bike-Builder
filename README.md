<div align="center">

<sub>3D 分步交互说明书</sub>

# 山地车组装指南

**一台全避震山地车，从一根车架装到能骑走**

<p>
<a href="https://build-bike.vercel.app"><b>打开使用</b></a>
&nbsp;&nbsp;·&nbsp;&nbsp;
<a href="#在本地运行">在本地运行</a>
&nbsp;&nbsp;·&nbsp;&nbsp;
<a href="docs/README.md">技术文档</a>
</p>

<img src="docs/shots/01-hero.png" alt="装完的整车">

<p><sub><b>8</b> 章 &nbsp;·&nbsp; <b>29</b> 步 &nbsp;·&nbsp; <b>27</b> 个大件 &nbsp;·&nbsp; <b>7</b> 颗螺丝 &nbsp;·&nbsp; 十来分钟</sub></p>

</div>

<br>

每一件都亲手装上去：顺着方向推到位，绕着轴心拧到底。装错了当场有反馈，并告诉你为什么。

<br>

## 怎么用

打开网页就能用，不用注册，不用安装。浏览器需支持 WebGL 2（Chrome / Edge 111+、Safari 16.4+、Firefox 113+）。

| | |
|---|---|
| **翻页** | <kbd>←</kbd> <kbd>→</kbd>，或画面两侧的箭头。这一步的活还没干完时，「下一步」先替你演一件 |
| **装一件** | 顺着箭头把零件拖到位。方向只有一个，拖错了会被弹回来 |
| **拧一颗** | 按住螺栓绕圈拧，转一圈进一个螺距，到底一声「咔」 |
| **认零件** | 指到哪件，画面上就报哪件的名字 |
| **让它演** | 点「帮我装上」或「帮我拧上」，这一件替你做完 |
| **退出** | <kbd>Esc</kbd> 逐层收起菜单与弹出的卡片 |

<br>

<table>
<tr>
<td width="50%">
<img src="docs/shots/02-exploded.png" alt="二十七个大件摊开"><br>
<sub>开场把二十七个大件按装配顺序倒着摊开，再一件一件装回去。</sub>
</td>
<td width="50%">
<img src="docs/shots/03-cross-tighten.png" alt="按对角顺序拧四颗面盖螺丝"><br>
<sub>面盖四颗按对角顺序拧。跳过不拦，但会提示，并记进结尾自检。</sub>
</td>
</tr>
<tr>
<td>
<img src="docs/shots/04-dark.png" alt="左脚踏反牙 · 深色主题"><br>
<sub>左脚踏是反牙。往正牙方向拧满两圈会发涩停住、退回半圈，再告诉你为什么。</sub>
</td>
<td>
<img src="docs/shots/05-tally.png" alt="出门前自检"><br>
<sub>装完逐条对账，二十七件七颗一条不落。每一行都能点回它所属的那一步。</sub>
</td>
</tr>
</table>

<br>

## 手机与平板

<div align="center">
<img src="docs/shots/06-mobile.png" width="300" alt="手机竖屏">
</div>

竖屏、横屏、平板各有一档版式。界面退到四周，中间留给车。

<br>

## 在本地运行

需要 Node `^22.13 || >=24`。

```bash
npm install && npm run dev
```

打开 <http://localhost:5174>。构建、测试与部署见[技术文档](docs/README.md)。

## 已知限制

- 首屏要读一份 12 MB 的整车模型，第一次打开有十来秒加载条
- 螺纹连接只有七处：面盖四颗、前桶轴、左右脚踏轴。其余大件推到位即算装好
- 进度不存档，刷新即从头开始
- 界面只有中文

## 参与

哪一步走不下去、哪个数不对，[开个 Issue](https://github.com/MrBaoboer/3D-Bike-Builder/issues)。订正工程数据（规格、螺距、装配顺序）尤其欢迎，请附出处。
改代码见 [CONTRIBUTING.md](CONTRIBUTING.md)。另有[行为准则](CODE_OF_CONDUCT.md)、[安全上报](SECURITY.md)与[商业授权](COMMERCIAL.md)。

## 许可

| | 范围 | 许可 |
|---|---|---|
| 代码 | `src/` `tools/` `index.html` 构建配置 | **AGPL-3.0** |
| 内容 | 课程编排、文案、装配清单、程序化螺栓与工具几何、音效配方、设计令牌 | **CC BY-NC-SA 4.0** |
| 整车模型 | `public/models/CarbonFrameBike.glb` 与含它画面的截图 | **CC BY-SA 4.0**，第三方 |

整车模型的署名见 [assets/CREDITS.md](assets/CREDITS.md)。自行部署或商用前先看 [COMMERCIAL.md](COMMERCIAL.md)。
