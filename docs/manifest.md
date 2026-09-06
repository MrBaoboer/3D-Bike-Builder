# 清单与模型

## 清单

`assets/bike.manifest.json` 是唯一事实来源。装配方向、行程、螺距、依赖顺序都写在这里，代码里不写第二份。改了跑 `npm run verify`。

长度一律用模型单位（1 单位 = 1 米，与 GLB 坐标同一套），`pitch` 用 mm。坐标全是 `render/bike.js` 加载完成后的世界坐标：前把已扶正，整车底面已压到 y = 0。

| 字段 | 含义 |
|---|---|
| `parts[].nodes` | 该件对应的 glTF 节点名，存原始名（含空格） |
| `parts[].install.kind` | `slide` 推入；`thread` 旋入，紧固件长在件上（脚踏轴），`interact/screw.js` 据此不另画一颗程序化螺栓 |
| `parts[].install.dir` | 装配方向，世界单位向量，从预备位指向装配位 |
| `parts[].install.gap` | 预备位沿 `-dir` 退开的距离 |
| `parts[].install.snap` | 吸附阈值，必须小于 `gap` |
| `parts[].fasten` | 这一件要拧的紧固件 id |
| `parts[].needs` | 必须先在车上的件 id。只写硬约束，同时就绪的按书写顺序装 |
| `parts[].pivot` | `kind` 为 `thread` 时必填：轴上一点 `point` 与轴向 `axis` |
| `fasteners[].tool` | `hex-N` 内六角、`wrench-N` 扳手，人话对照在 `steps/util.js` 的 `TOOL_NAME` |
| `fasteners[].thread` | `left` / `right`。左脚踏必须是 `left` |
| `fasteners[].spec` `pitch` `turns` | 规格串、螺距、拧到底的圈数。`pitch` 要与 `spec` 一致 |
| `fasteners[].axis` `point` | 拧入方向（世界单位向量）与螺栓头中心（世界坐标） |
| `fasteners[].group` `order` | 同组一起拧。`cross` 对角交叉、`seq` 按书写顺序、`any` 随意 |

`npm run verify`（`tools/check-manifest.mjs`）不依赖 three 与浏览器，只解 GLB 的 JSON 块，逐条对账，对不上退 1：
id 互不相撞；引用的节点名在 GLB 里存在；`dir`、`axis` 是单位向量；`needs`、`fasten` 指向的东西存在，没有无人使用的紧固件；依赖图无环；`turns × pitch` 在 3–30 mm 之间；`thread` 合法且左脚踏为 `left`；`cross` 组偶数颗且不少于 4 颗；`0 < snap < gap`；`pitch` 与 `spec` 一致。

它查不出的两条，改清单时自己守：

- **`point` 必须落在螺栓头真正贴合的那张面上。** 埋进网格里整颗螺栓看不见，而校验器不做光线求交。复查办法是沿拧入方向从 `point` 打一条射线求交，看每颗埋了多少毫米。
  把立面盖四颗尤其要看：`Vorbau_Hope_FR_35mm` 是一整块焊死网格，螺丝没被单独建模，这四颗由 `render/bolt.js` 按清单里的 `stem-face-*` 程序化生成。
- **节点名存原始 GLB 名。** `GLTFLoader` 加载时过一道 `PropertyBinding.sanitizeNodeName`（空白转下划线，`[ ] . : /` 删掉），`"Lenker 1"` 运行时叫 `Lenker_1`；查表前由 `Bike.sanitize()` 做同样的净化。
  清单改成净化名，校验器就拿 GLB 对不上账，模型改名这类走散会一路漏到线上。

## 模型

`public/models/CarbonFrameBike.glb`，12 MB，第三方素材（CC BY-SA 4.0，署名见 [assets/CREDITS.md](../assets/CREDITS.md)）。

选未压缩的 glTF-Binary，不选上游同目录 3.24 MB 的 glTF-Draco-KTX2：后者的 KTX2 走 ETC1S 档，码本按感知颜色优化，而这份资产 11 张贴图有 7 张是法线贴图，压方向向量会在曲面上留下着色不连续。
要减体积应另用 UASTC 自行转制，`tools/tex-audit.mjs` 能分辨两种模式。

### 加载时的三件事

都在 `render/bike.js`，顺序固定：

1. **摘掉 `Shadows` 组。** 那是给 iOS AR QuickLook 垫的假阴影贴片，网页里是糊在车底的黑面。
2. **扶正前把：绕转向轴转 −40.212°。** 上游静止姿态里前端绕转向轴打了一个角。扶正之后前轮心回到中垂面（Z ≈ 0），前轴成为纯 −Z。
   旋转要在父空间里绕过支点的轴做；直接写 `node.rotation` 只绕节点自身原点转，前轮会甩出去。
3. **烘焙蒙皮网格。** 上游把四根刹车油管绑在悬挂骨架上，蒙皮顶点的真身在骨骼变换之后，而 Box3、拾取、取景量的都是裸顶点 × matrixWorld，对油管会差出近一米。
   动画永不播放，骨骼就是常量：把当前姿态烘进顶点、换成普通网格。必须排在扶正之后，扶正会动骨骼的祖先。

模型自带的 `Holobike_Loop` 是给镜头看的爆炸展开动画，一帧也不播，也不能用来反推装配方向：座管在其中几乎纯横向脱离，而立管轴是斜向上的；左右脚踏朝同一侧飞出。
装配方向一律取几何轴（立管轴、转向轴、前轴、曲柄轴）。

### 坐标与单位

- Y 轴向上（glTF 规范），+Z 是车的左侧，−X 是车头
- 1 单位 = 1 米；整车底面压到 y = 0，前把已扶正
- 清单里的坐标就是这套世界坐标

标定以 Hope 200 mm 刹车碟为主标尺（前后两片实测 200.22 mm），花鼓 150 / 110 mm 交叉验证。不用轮胎标称尺寸标定：实测后胎 683.9 mm 对标称 707 mm 差 3%。

### 世界方向 ≠ 局部方向

清单给的 `install.dir`、`pivot.axis`、紧固件 `axis` 都是**世界方向**，而模型里每个件的父级基底都不是单位阵：前轮挂在 `Federung → Lenker` 下，脚踏挂在 `Kurbel_X01_DH → Pedale` 下，座管的父级是一个 90° 旋转。

把世界向量直接加到 `object.position` 上，件会沿一个无关方向跑掉。换算只在 `interact/slide.js` 的 `#rig()` 里做，**步骤脚本一律走 `ctx.slide.park(partId, u)`**，不自己拿 `install.dir` 加减。

「装配位」只认节点第一次被看到时的位置（记在 `userData.homePos`）。拿「现在在哪儿」当装配位，一个已经退到预备位的件再被备一次，记下的就是预备位，每备一次再偏一个 gap。
