# 开发与维护

面向要改这个项目的人。前半是各层之间的契约，后半是维护时绕不开的约束。改接口先改这里。

## 环境与命令

Node `^22.13.0 || >=24.0.0`。

```bash
npm install && npm run dev
```

开发服务器在 <http://localhost:5174>。

| 命令 | 做什么 | 耗时 |
|---|---|---|
| `npm run lint` | ESLint，只开 recommended | 秒 |
| `npm test` | `node:test`：清单校验器与清单读取层的单测 | 秒 |
| `npm run verify` | 清单 × GLB 离线逐条对账 | 秒 |
| `npm run build` | Vite 8（rolldown）构建到 `dist/` | 秒 |
| `npm run check:code` | 以上四条。CI 对每个 PR 跑这一条 | 半分钟 |
| `npm run smoke` | Playwright 起 `dist/`，宽窄两种画幅各真装一遍 | 三五分钟 |
| `npm run check` | `check:code` + `smoke` | 几分钟 |
| `npm run frames` | 逐步量取景：主体在不在舞台中央、整车照贴没贴边 | 一两分钟 |
| `npm run shots` | 从 `dist/` 实拍 README 的六张截图到 `docs/shots/` | 一两分钟 |
| `npm run live` | 比对线上两处发的是不是同一份产物 | 秒 |
| `npm run model` | 打印整车装配树 | 秒 |

`smoke`、`frames`、`shots` 都从 `dist/` 起页面，先 `npm run build`。
`npm run smoke -- --shots` 每一步都截图到 `.shots/smoke/`，不加只截失败那一张；`--headed` 开着浏览器跑。
换画幅量取景：`node tools/frame-audit.mjs 390 844`。

页面把共享上下文与引擎挂在 `window.__ctx` / `window.__engine` 上，工具脚本与探针都从这里进。

## 目录

```
assets/    bike.manifest.json 装配清单（唯一事实来源）· CREDITS.md 第三方素材署名
public/    models/CarbonFrameBike.glb 整车，12 MB，第三方素材
src/
  main.js    唯一的组装处，也是唯一读清单 JSON 的地方
  styles.css 画布与封面
  core/      bom 清单读取 · state 状态
  render/    stage 舞台 · bike 整车 · bolt 程序化螺丝与工具 · fx 特效
  interact/  slide 推入 · screw 旋入 · pick 指到哪件报哪件
  audio/     sfx 实时合成音效，不加载音频文件
  ui/        hud 界面 · icons 图标 · guides 三维箭头 · styles/ 四份 CSS
  app/       engine 分步引擎 · build 此刻车上该有哪些件
  steps/     acts 八章二十九步 · util 取景现算与共用铺陈
  util/      tween 补间
tools/     门禁与量尺：check-manifest · unit.test / bom.test · smoke · frame-audit
           shots · live-check · serve · glb-* 与 tex-audit 模型分析
docs/      本文件 · shots/ README 用的截图
.analysis/ 开发期探针，不进版本库
```

**门禁进版本库，量尺不进。** `tools/` 回答「有没有坏」，CI 与提交前跑；`.analysis/` 回答「现在到底是多少」，随查随写。
新探针照 `tools/frame-audit.mjs` 的样子写：起 `dist/`、打开页面、在页内 `evaluate` 里量、把数打出来。
开发期还有一条 `/__shot`（`vite.config.js` 的 `dev-shot` 中间件）：页面把画布的 dataURL POST 过来，落到 `.shots/`，给无法直接截屏的环境用。

## 分层

只允许向下依赖。模块之间只认下面的 ctx 契约，不互相 import 单例。

```
core/      清单、状态          不碰 three（只借 Vector3 这一个纯数学类型）
render/    舞台、整车、螺丝与工具、特效
interact/  把指针变成装配动作
audio/     声音                不依赖其他层
ui/        界面组件            认识 DOM 与 three 的向量，不认识自行车
app/       分步引擎            只认识「一步长什么样」
steps/     步骤内容            通过 ctx 取用以上全部
```

**界面层不自带内容。** 章节名由 `steps/acts.js` 的 `PHASES` 提供，`hud.setChapters(steps, phases)` 的第二个参数必传。

**样式的层叠顺序**：`ui/styles/tokens.css` 令牌 → `ui/styles/base.css` 重置与无障碍 → `styles.css` 画布与封面 →
`ui/styles/chrome.css` 常驻界面 → `ui/styles/surfaces.css` 覆盖层。
颜色只在 `tokens.css` 里定义，组件只引用语义令牌。同名令牌一旦有第二处取值，谁赢由 import 顺序决定。

## ctx：全片共享的那一个对象

| 键 | 是什么 | 常用的 |
|---|---|---|
| `stage` | 舞台 | `setRecommended({az,el,dist,target,fit,ease})` · `snapToRecommended()` · `setSafeArea({top,bottom,left,right})` · `hold(bool)` · `setTheme()` · `updaters` |
| `bike` | 整车 | `get(name)` · `has(name)` · `boundsOf(name)` · `setVisible(name,bool)` · `highlight(names,color,strength)` · `clearHighlights()` |
| `bom` | 清单 | `part(id)` · `fastener(id)` · `nodesOf(partId)` · `groupOf(idOrGroup)` · `crossPairs(group)` · `crossMate(group,id)` · `order()` · `parts` · `fasteners` · `counts` |
| `bolts` | 程序化螺丝与工具 | `spawn(idOrFastener)` · `useTool(kind)` · `hideTools()` · `remove(id)` · `clear()` |
| `slide` | 一自由度推入 | `begin({partId,onSeat,onAll,wrongHint,sound,glow,follow})` · `park(partId,u)` · `burst(partId,世界位移)` · `cancel()` · `autoSeat(partId?)` |
| `screw` | 旋入 | `begin({fastenerId,onProgress,onTight,onWrongWay})` · `beginGroup({group,onEach,onAll})` · `cancel()` · `autoRun(id?)` · `autoRunNext()` |
| `pick` | 指到哪件报哪件 | `begin({ids?,fallback?,onHover})` · `cancel()`。只问不改，从不夺走轨道控制。由引擎统一挂，步骤只在不想要时声明 `noPick: true` |
| `hud` | 界面 | `setCue()` · `setNote()` · `setAlts()` · `toast()` · `tag()` · `dock()` · `sheet()` · `guide()` · `addSpot()` · `setBoltRow()` · `setChapters()` · `setStep()` · `readyNext()` |
| `sfx` | 声音 | `play(name,{gain,pitch,delay})` · `setEnabled()`。只有四种：`THREAD_TURN` `SNUG_CLICK` `SEAT_IN` `WRONG`；`WHEEL_SEAT` `POST_SEAT` 是 `SEAT_IN` 的轻重档别名 |
| `guides` | 三维方向箭头 | `set([{pos,dir,len}])` · `clear()` |
| `fx` | 粒子与提示环 | `ring(pos,axis,{r1})` · `spark(pos)` |
| `state` | 状态 | 直接读写。只有偏好（主题、声音、看没看过操作说明）落盘，键 `bike.v1.state`；进度刷新即归零 |
| `engine` | 引擎自身 | `next()` · `back()` · `go(i)` · `jump(i)` · `goToStep(id)` · `restart()` · `done()` · `pending` · `steps` |
| `build` | 此刻车上该有哪些件 | `plan(steps)` · `applyAt(i,{all})` · `stepOf(partId)` |
| `tier` | 画质档 | `'low' \| 'mid' \| 'high'` |

### 容易写反的签名

- `screw` 的 `onProgress` 收的是一整个对象 `{id, depth, turns}`，不是一个数。步骤脚本不自接这个钩子，走 `steps/util.js` 的 `fasten()` / `fastenGroup()`。
- `bom.crossPairs(group)` 发的是 **id 对**，不是紧固件对象。要问「谁是谁的对角」走 `crossMate(group, id)`。发成对象而调用方拿它跟 id 字符串比，恒不相等，对角配对会静默失效。
- `bolts.spawn()` 收 id 或紧固件对象都行。
- `slide.park(partId, u)` 的 `u`：0 是预备位，1 是装配位。
- `fx.ring()` 的默认半径是给 M5 螺栓头的，标整只轮子要按件的尺度给 `r1`。
- `stage.updaters` 的回调收 `(dt, t, slow)`，三个时间量各有各的封顶，见[主循环的时间量](#主循环的时间量)。

## 一步长什么样

```js
{
  id: 'D2', phase: 3,                        // phase 是 PHASES 的下标
  installs: ['handlebar'],                   // 这一步装上哪些件
  fastens: ['stem-face-a', 'stem-face-b'],   // 这一步拧哪些紧固件
  showAll: false,                            // true：不管装配计划，整车全在场
  noPick: false,                             // true：这一步不挂「指到哪件报哪件」
  title: '四颗面盖螺丝',
  cam: frameBolts(ctx, 'stem-face', { az: 170, el: 26 }),
  cue: '按对角顺序拧，四颗分两轮',            // 一行纯文字，没有图标也没有 HTML
  note: { title: '为什么必须对角', spec: [['工具', '4 mm 内六角']], body: '…' },
  enter(ctx, engine) { … },
  exit(ctx) { … },
}
```

**`installs` 是整份课程的骨架。** `app/build.js` 据此现推「走到第 i 步时车上该有哪些件」：
还没轮到的不在场、正在装的在场但由这一步自己摆、装过的回到装配位。
漏声明一件，它就会从头到尾挂在画面上。`installs` 与 `fastens` 也是出门前自检的依据，每一行靠它连回对应的步骤。

**`enter()` 不能返回等用户动手的 Promise。** 引擎 `await` 它，在里面等用户，`engine.busy` 就永远不落，翻页、冒烟、自动路径全部卡死。到位之后要做的事走 `onDone` 回调。

一步装多件时默认逐件拖装（摇臂、刹把）。两件在台面上已连成一体的（直装牙盘锁在右曲柄上），
加 `follow: { 从动件: 驱动件 }`：从动件不单独拖、不长箭头，抓哪件都驱动整组、同一个 u 走完全程、坐实一起记账。
前提是两件在清单里同 `dir` 同 `gap`，飞行中相对位置才不变。

**文案能少则少。** 常驻文字只有步名与一行旁白；说明卡只写物理原因看不见的那几处：反牙、对角、预紧、直装接口、最小插入线。

### 取景的四个入口

`cam` 一律由 `steps/util.js` 现算，不写常量。按这一步在讲什么挑：

| 入口 | 用在哪 | 框住什么 |
|---|---|---|
| `shot(ctx, parts, o)` | 推入一件的步骤 | 这几件从预备位到装配位扫过的整段行程 |
| `frameBolts(ctx, group, o)` | 拧螺丝的步骤 | 那几颗紧固件本身，不是它们长在的那个大件 |
| `frameWhole(ctx, o)` | 整车四张 | 全车逐网格的包络；`bare` 只量光车架，`burst` 量摊开态 |
| `viewFor(dir, o)` | `shot` 内部 | 只定机位，不定取景 |

`shot()` 先按装配方向反推机位（`viewFor`），再按那个机位量取景（`frameOf`）。顺序不能反：半跨度是在相机基底里量的。

`shot()` 还收 `near`（上一步站在哪个方位）。一步装一对镜像件、或者件顺着车身长轴推进来时，
站左站右同样成立，挑离上一步近的那一侧。`steps/acts.js` 里的 `near` 游标顺着列表往下传，手写机位的步骤用 `at()` 也要带上。

## 交互原语的共同约定

1. **只有一个合法方向。** 错误方向阻尼回弹并说明为什么，不开放自由 6DoF。
   回弹必须看得见：往错方向使劲时件要先给出一点（推入侧顶歪 `K.LEAN`，即 gap 的 8%；旋入侧照转不进给），到头顶住再弹回来。
2. **按下即夺权，松手才交还**轨道控制。中途设回 `controls.enabled = true`，剩下半程会变成转镜头。
3. **每个原语都有自动路径**（`autoSeat` / `autoRun`），与手拖共用同一条代码，该看到、该听到的一样不少。
   「帮我装上」在推入侧连败三次才摆出来；「帮我拧上」在旋入侧一进去就摆着，一颗要绕八圈，等到第三次失败太晚。

三处必须让手指记住的手感：

| | 在哪 | 要点 |
|---|---|---|
| 左脚踏反牙 | `pedal-left-spindle`，`thread: "left"` | 允许拧错。往正牙方向拧满两圈才发涩、停住、回退半圈，再说明「左边是反的」。只有左牙记 `state.wrongThread`：右牙倒转两圈是「退不下去了」，不是拧错 |
| 面盖对角顺序 | `stem-face` 四颗，`order: "cross"` | 不拦，只把 `orderOk` 交给步骤脚本，并记进 `state.crossOrderOk`，结尾自检要提。判定走 `bom.crossMate()` |
| 拧到底 | 全部 | 绕圈拧，转一圈进一个螺距，转满 `turns` 圈到底，一声「咔」。没有扭矩读数，也没有滑丝 |

拧螺丝的步骤，机位要正对螺栓轴、从螺栓头那一侧看。旋入靠指针绕轴画圈读角度；
轴躺进屏幕平面时（`|视线·轴| ≤ 0.3`）拖拽平面几乎与视线平行，读角度退化成沿屏幕切向的位移除以半径，手感差一截。
这几步的机位都是手写的（`frameBolts` 的 `az/el`、`shot` 的 `cam`），改的时候守住这一条。

## 翻页：一下演一件

翻页永远不被拦，但**「下一步」在这一步还有活没干完时，先演一件，不翻页**：
两条摇臂按两下，四颗面盖螺丝按四下，做完再按一下才走。
一口气演完整步不行，四颗螺丝在一两秒里连着转完，「对角、分两轮」来不及看清。
判据见 `engine.pending` 与 `engine.finishPending()`，两者都只认各原语的 `session.pending`。

**上一下还没走完时按的那一下要攒着**（最多四下），忙完一下一下补上。
「拆开看看」进场要演一秒多，连按两下方向键很常见。直接跳步（`engine.jump()`）清空攒下的。

空格与方向键的让位规矩不一样：

- **空格**是按钮的激活键，焦点落在任何控件上都得让开。
- **方向键**在按钮上没有默认动作，只让给自己要用方向键的控件（进度轨、菜单、文本框）。
  它们各自 `preventDefault()`，引擎认这一下就够。

两键按同一条处理的话，用鼠标点完「下一步」之后方向键整个失灵，焦点正在那枚按钮上。

## 清单

`assets/bike.manifest.json` 是唯一事实来源。装配方向、行程、螺距、依赖顺序都写在这里，代码里不写第二份。改了跑 `npm run verify`。

- **`install.kind`** 分 `slide`（推入）与 `thread`（旋入）。`thread` 的件意味着紧固件就长在件上（脚踏轴），
  `interact/screw.js` 据此不另画一颗程序化螺栓，否则画面上会多出一颗现实中不存在的螺母，还跟着件一起被拖走。
- **`point` 必须落在螺栓头真正贴合的那张面上。** 埋进网格里螺栓整颗看不见，而 `npm run verify` 查不出这一条（它不做光线求交）。
  复查走 `.analysis/p-seat-surface.js`：沿拧入方向打一条射线，报出每颗埋了多少毫米。
  把立面盖那四颗尤其要看：`Vorbau_Hope_FR_35mm` 是一整块焊死网格，螺丝没被单独建模，这四颗由 `render/bolt.js` 按清单里的 `stem-face-*` 程序化生成。
- **清单存的是原始 GLB 节点名**（含空格，如 `"Lenker 1"`）。`GLTFLoader` 加载时过一道 `PropertyBinding.sanitizeNodeName`
  （空白转下划线，`[ ] . : /` 删掉），运行时叫 `Lenker_1`；查表前由 `Bike.sanitize()` 做同样的净化。
  清单改成净化名的话，`tools/check-manifest.mjs` 就拿 GLB 对不上账，「模型改名」这类走散会一路漏到线上。

## 模型

`public/models/CarbonFrameBike.glb`，12 MB，第三方素材（CC BY-SA 4.0，署名见 `assets/CREDITS.md`）。

选未压缩的 `glTF-Binary`，不选上游同目录 3.24 MB 的 `glTF-Draco-KTX2`：后者的 KTX2 走 ETC1S 档，码本按感知颜色优化，
而这份资产 11 张贴图有 7 张是法线贴图，压方向向量会在曲面上留下着色不连续。要减体积应另用 UASTC 自行转制，`tools/tex-audit.mjs` 能分辨两种模式。

加载时三件事不可省，都在 `render/bike.js`，顺序固定：

1. **摘掉 `Shadows` 组。** 那是给 iOS AR QuickLook 垫的假阴影贴片，网页里是糊在车底的黑面。
2. **扶正前把：绕转向轴转 −40.212°。** 上游静止姿态里前端绕转向轴打了一个角。
   扶正后前轮心 Z 由 31.94 mm 归到 −0.7 mm，前轴由 [−0.585, 0.274, −0.764] 变成 [0.0004, 0.0009, −1]。
   旋转要在父空间里绕过支点的轴做。直接写 `node.rotation` 只绕节点自身原点转，前轮会甩出去。
3. **烘焙蒙皮网格。** 上游把四根刹车油管绑在悬挂骨架上，蒙皮顶点的真身在骨骼变换之后，
   而 Box3、拾取、取景量的都是裸顶点 × matrixWorld，对油管会差出近一米。
   动画永不播放，骨骼就是常量：把当前姿态烘进顶点、换成普通网格。必须排在扶正之后，扶正会动骨骼的祖先。

模型自带的 `Holobike_Loop`（12.02 s，356 通道）是给镜头看的爆炸展开，一帧也不播，也不能用来反推装配方向：
座管在其中的脱离方向是 [0.210, 0, −0.978]（几乎纯横向），而立管轴是 [0.514, 0.858, 0]；左右脚踏也朝同一侧飞出。
装配方向一律取几何轴（立管轴、转向轴、前轴、曲柄轴）。

### 坐标与单位

- Y 轴向上（glTF 规范），+Z 是车的左侧，−X 是车头
- 1 单位 = 1 米；整车底面压到 y = 0，前把已扶正
- 清单里的坐标就是这套世界坐标

标定以 Hope 200 mm 刹车碟为主标尺（前后两片实测 200.22 mm），花鼓 150 / 110 mm 交叉验证。
不用轮胎标称尺寸标定：实测后胎 683.9 mm 对标称 707 mm 差 3%。

### 世界方向 ≠ 局部方向

清单给的 `install.dir`、`pivot.axis`、紧固件 `axis` 都是**世界方向**，而模型里每个件的父级基底都不是单位阵：
前轮挂在 `Federung → Lenker` 下，脚踏挂在 `Kurbel_X01_DH → Pedale` 下，座管的父级是一个 90° 旋转。

把世界向量直接加到 `object.position` 上，件会沿一个无关方向跑掉。换算只在 `interact/slide.js` 的 `#rig()` 里做，
**步骤脚本一律走 `ctx.slide.park(partId, u)`**，不自己拿 `install.dir` 加减。

「装配位」只认节点第一次被看到时的位置（记在 `userData.homePos`）。
拿「现在在哪儿」当装配位，一个已经退到预备位的件再被备一次，记下的就是预备位，每备一次再偏一个 gap。

## 取景：现算，不写常量

四个入口见[上文](#取景的四个入口)。七条判据，少一条就有步骤看不清：

0. **量之前先刷矩阵**（`nodeBoxes` 的第一句 `updateWorldMatrix(true, true)`）。少了它，第 1 条会静默失效，画面看上去只是「构图有点怪」。详见下节。
1. **量整段行程**，不是件的静态包围盒。只框装配位，件在起手那一头会飘到画面外（前轮的行程包络比它自己高 34%）。
2. **在相机基底里量。** `fit` 的 `{r,h}` 被当成水平半径与垂直半高独立处理，而相机是斜着看的，包围盒的对角线比任何一条边都长。
   拿世界 XYZ 凑，会成片出现下缘裁掉的步骤。
3. **补上主体的半深** `fit.d`。近景里主体深度与机位距离同量级：主转点轴那一步主体长 211 mm 而机位只有 190 mm 远，
   最靠近相机的角投影出来比按中心算的宽四成。判据要落在离相机最近的那一层上。
4. **逐网格量，不要先并成一个包围盒。** 自行车又扁又空，整车并集的角落大半是空气，按并集算的竖向半跨度比车真正的半高大 25%。
5. **近景有最近工作距离**（`MIN_SPAN`，260 mm）。只按主体包络定距，小件会把镜头拽到贴脸，满屏碳纹里一颗小螺栓认不出是车上的哪儿。
6. **锚点落在预备位**（`ANCHOR_TO_SEAT` = 0.10），不是整段行程的中点。件常常比行程还小，对准中点它停在行程哪一头都贴着画幅边；
   而进场那一刻件停在预备位，那才是「初始视角」的那一眼。代价是画幅大一点，半跨度由 `(行程+件长)/2` 变成 `行程+件长/2`。
7. **默认正中，要偏就明写。** 主体锚点落在舞台正中；构图上确实要偏的，由步骤自己写 `off: [右为正, 上为正]`
   （以取景半跨度计，车把那一步是 `[-0.13, 0.14]`）。不要加「一律往整车形心偏一档」的自动补偿：
   面盖四颗只占画面 6% 宽，被推出去 0.16 个半跨度就是「没在中间」。

`npm run frames` 的「主体 x,y」那一栏盯着第 7 条，偏心超过 0.30 当场退 1。

### 量几何之前先自己刷矩阵

**`Box3.setFromObject()` 不替你刷矩阵。** 它内部走 `updateWorldMatrix(false, false)`：只重算自己那一格，既不回头刷祖先，也不往下刷子树。
而取景代码跑在首帧渲染之前，那时全场没有任何人刷过矩阵。两种走法都会错：

- **祖先旧。** 前面的步骤刚挪过摇臂、曲柄、轮组，挂在它们底下的件拿到的祖先矩阵是旧的。
  这份模型里节点原点离自己的几何常常很远（油管那四段差一米），矩阵一旧，量出来的盒退化到原点那一带，机位随之对错地方。
- **子树旧。** 清单登记的二十七个节点里二十六个是组节点（只有油管是网格本身），`slide.park(id, u)` 改的是组节点的 `position`，
  而盒是逐子网格量的，子网格拿到的还是旧 `matrixWorld`。park 的位移一点也量不到，「量整段行程」整个失效。

所以 `steps/util.js` 的 `nodeBoxes()` 与 `render/bike.js` 的 `boundsOf()` 都在量之前先 `updateWorldMatrix(true, true)`。
新写量几何的地方照做。两种都不报错，只有 `npm run frames` 的「主体 x,y」那一栏抓得出来。

## 换步的运镜

二十九步是一件事，不是二十九张图，全靠「同一台车，镜头挪过去」串起来。
步骤之间**一律走一段运镜，没有跳切**。机位过渡由 `stage` 统一排，步骤只声明「停在哪儿」。

- 在**轨道坐标**里插值（目标点 / 方位角 / 仰角 / 距离），不在世界坐标里。世界坐标直线插值走的是弦不是弧，左右脚踏那两步隔着一百多度，相机会穿过车身。
- 方位角走最短的一边；距离按**等比**推拉；中段把距离往外鼓一点（转得越多鼓得越高），避开中途蹭到车身。
- 缓动用 `Ease.smoother`（五次平滑，两头速度与加速度都为零）。`inOutCubic` 中段速度是平均值的三倍，一段一百度的环绕会甩到 180 °/s。
- 时长按「这一趟有多远」现算，0.45–1.7 s。步骤可用 `cam.ease` 拉长（`ease: 0.55` 约两倍，「拆开看看」用它与两秒六的摊开同步）。
- 用户一碰画面，这一趟当场作废，转到哪儿停在哪儿。

**克制。** 镜头一动，看的人会以为「有什么变了」而重新找一遍画面：

- 屏幕上看不出来的位移（`TINY`）直接落位，一帧都不动画。按定义它在屏幕上不可见，不算跳切。
- 中段外扩只留给真要绕过去的那几趟（`BULGE_FROM` 40°）。
- 界面自己引起的重新取景（说明卡摊开、读数出现、转屏）走快档 0.32 s、不外扩：它是纠版式，不是叙事。
  但**不掐短正在走的那一趟**：动手的步骤在 `enter()` 里摆出那一排螺丝，安全区当场变一次，不防的话一趟一秒四的环绕会被压成 0.32 秒。

减少章与章之间的甩动，办法不是把运镜调慢，而是让相邻两步站得近：

| | 改法 |
|---|---|
| 开箱三步 | A1 / A2 / A3 共用 `BURST_VIEW`，镜头不动，变的只有车 |
| 镜像对 | 一步装一对镜像件时（摇臂、刹把、把套）挑离上一步近的那一侧（`shot` 的 `near`） |
| 传动四步 | 牙盘锁上右曲柄连成一件（`follow`），四步里三步都在传动侧；只有左曲柄要过到左侧 |
| 车把 | 件顺着车身长轴推进来时两边等价，`viewFor` 报出 `alt` 让 `shot` 挑 |
| 左右脚踏 | 对称站到各自那一侧的前四分之三位，车头留在画面里（这一步讲的就是「朝车头方向拧」） |
| 出门前自检 | 回到开箱那一个机位，首尾同一个视角 |

改机位之后拿 `npm run smoke` 的「运镜」那一条对：它正着倒着各走一遍全程，断言每一趟都排了时长、分毫不差地落在该到的机位上，
且转过六十度以上的那几趟走的路程长于两点直线。

## 摊开那一步

`burstOffset(ctx, partId, k)` 给出每一件相对装配位的世界位移，`slide.burst()` 负责摆位。两股叠加：

1. **在相机的屏幕平面里以整车形心为中心各向放大**（`BURST_H` / `BURST_V`），件本身不放大。
   这是一次二维仿射放大，屏幕上分得开的两件，放大之后只会更分得开。
   不能按世界径向摊：放大量里有一大截落在视线方向上，那个方向的位移在画面上等于零。
2. **按「从哪一侧装进来」朝屏幕斜着分开**（`BURST_SIDE` / `BURST_SIDE_V`）。二十七件里十五件是侧装件，两两成镜像，在侧影里严丝合缝地重合。
   方向是斜的不是纯左右：镜头站在车的左前方，右侧那几件全在远端，纯横向推正好撞上近端那一排。

机位与位移共用 `BURST_VIEW` 的 az/el，两边必须一致，否则算的是另一个平面。

时间上**按装配顺序倒着走**（`BURST_STAGGER` 占总时长六成）：最后装上的先飞出去，车架最后剩下。一口气全炸开只是一次位移，看不出层次。

复查走 `npm run smoke` 里的「摊开」那一条：逐件比对「有它 / 没它」两张渲染的像素差，差出来的就是它露在最前面的部分。
改了上面任何一条常量，`npm run frames` 前后各跑一遍对着看，只看截图会漏掉「小了一成」这种量级的退化。

## 界面遮住多少画面

`ui/hud.js` 每次布局变化都量一次顶栏、底栏与右列，写进 `--bar-h`，并把**四条边**报给 `stage.setSafeArea()`，三维据此让位、退远、把主体挪进可用区正中。

- 算「底部占了多少」有两道判据，缺一不可：**贴底**（下沿离屏幕底不超过一条底部条）与**挡道**（与画面横向中间那一半有重叠）。
  少了前者，摆在右上角的读数区会被算成「底部占了几百像素」，主体被顶出画面；少了后者，缩在右下角的读数会让整车无谓上移。
- 算「右边占了多少」同理用几何：贴着右缘、且高得能挡住主体，才算数。让位量按「卡片高 ÷ 可用画面高」折算，右上角一张小卡只让它真正占着的那一档。
  窄屏上这一列横铺在底部、左沿落在屏幕左半边，这一条自然不成立，它已经算进 bottom 里，再算一次会让车横着缩一半。
- 横向的挪动用**相机自己的右向量**。世界 X 在斜看时既有横向分量也有纵深分量。
- 可用区有下限 `0.46`：界面再厚也不能把主体挤成一枚邮票。这一档由手机上拧面盖那一步定住，竖屏 844 px 里界面实测占掉 440 px。

`stage.hold(true)` 冻住「界面高度变了就重新取景」，但**晚冻两帧**：
那一排螺丝是这一步 `enter()` 里才摆出来的，而 hold 在同一个 enter 里同步调，立刻冻上这一条就永远算不进取景。

## 主循环的时间量

`stage.start()` 每帧发三个时间量，各有各的封顶：

| | 封顶 | 给谁 |
|---|---|---|
| `dt` | 50 ms | 特效与三维箭头。逐帧积分，一帧跳太多会把状态走过头 |
| `slow` | 250 ms | 补间与运镜。纯插值，跳多远都不会走过头 |
| `t` | 无 | 累计时间 |

补间**不能**跟着 `dt` 走 50 ms 封顶：低帧率下每秒推进不足一秒的量，动画整段拖长，越卡的机器动画越长。
无头浏览器是软件渲染，这个场景下只有约 3 fps，逐帧量相机角速度会得出几百度每秒的「跳变」，那是封顶在低帧率下的正确行为。
要量运镜质量，看 `npm run smoke` 的「运镜」断言与 `npm run frames` 的输出。

## 不投影

场景里没有影子，也没有接影的地面（`render/stage.js` 不开 `shadowMap`，`render/bike.js` 不设 `castShadow` / `receiveShadow`）。
自行车投在地上的影子是一大片辐条、链条、叉腿交织的噪点，面积常常比主体还大，近景里摊在主体旁边抢视线。

两处量纲跟着这一条走：

- 向下那一面全靠半球光的下半色交代形体：下半色 `0x8a8f96`，浅色主题强度 0.6，深色主题收到 0.34。
- **取景余量比有影子时紧。** 影子原本占着车底下那一块，同样的余量在没有影子时读起来就是「车缩在中间一小团」，整车那几张的 `pad` 收到 0.96。

## 冒烟走查

`npm run smoke` 不是「翻页看标题」，它必须真的把车装一遍。每一步查：可达、有标题、声明了 `cam.fit`、
这一步瞄的那个点投影后落在界面没遮住的那块画面里、画面不是纯色。
然后**从头到尾按用户那条路把整台车装完**（一步步按「下一步」，每一步把剩下的活按「一下一件」演完），再对账：

- 每一件精确落回装配位（偏差 < 0.01 mm），二十七件七颗都记上账，自检三十四行报「全部到位」
- 在场件数一路只增不减，每一件进场时它声明的前置件都已在车上
- 四颗面盖要按四下，第五下才翻页；按对角拧算通过，拧相邻的当场记下不合格
- 左脚踏往正牙方向拧：转得动、拧满两圈停住、回退半圈再说明白；右牙倒转两圈照样顶住回弹，但不记成拧反
- 侧着推顶得住：件歪出去一点点就到头，松手摆正并说明为什么
- 整车那四张成品照完整落在画幅内；摊开那一步二十七件每一件都露得出来，指到哪件报哪件的名字
- 换步一律走一段运镜；深浅两套主题都换得动；方向键能前进后退；全程控制台没有报错与 4xx/5xx

**凡是有对错之分的交互，正例反例都要断言。** 只查「拧过了」不够，还要查「拧对了没有」；只查「步骤可达」不够，
四个交互原语都得真的走一遍，`enter()` 一进去就抛错的步骤，「可达」照样报绿。
`slide` / `screw` 的自动路径不走负角与侧向，「错方向」那半边只有真发指针事件才测得到：
用 `page.mouse` 真拖，抓点必须落在件身上的真实采样像素，拿包围盒形心会落进空处。

改 `tools/smoke.mjs` 之前先读这两条：

- 判「画面不是纯色」不能直接 `drawImage` WebGL 画布。没开 `preserveDrawingBuffer` 时那块缓冲在合成后就清了，读回来永远全黑。
  要在同一个任务里先 `render()` 再取样。
- 断言「镜头没对着空处」要投影**步骤声明的 `cam.target`**，不能投 `controls.target`：后者是让位之后的机位目标，按定义就在画幅正中。

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

## 部署

产物纯静态，`base: './'`，放任何子路径下都不用改配置。线上两处发的是同一份产物：

| | 地址 | 怎么来的 |
|---|---|---|
| 主 | <https://build-bike.vercel.app> | 推到 `main` → Vercel 出一次生产部署 |
| 备 | <https://mrbaoboer.github.io/3D-Bike-Builder/> | `.github/workflows/deploy.yml` → GitHub Pages |

**主地址必须是项目域名表里的那一条，不要用部署别名。** 别名指向某一个具体部署，不跟着生产走，每发一版就掉队一次，
而且照样返回 200、标题一样。判断办法：`vercel domains add <名字>.vercel.app <项目>` 能成，它就是项目域名；
报 `alias_conflict` 就换个名字，`vercel.app` 前缀全局先到先得。

**核对办法**：`npm run live`（`tools/live-check.mjs`）比对两处首页里带内容哈希的入口 JS 文件名，一样才是同一份产物，对不上当场退 1。
只看「打得开 / 返回 200」不够。

生产产物注入一条 CSP（`vite.config.js`），策略内容与理由见 [SECURITY.md](../SECURITY.md)。
改它之后**必须重跑 `npm run smoke`**：`connect-src` 少放 `blob:` 的话，GLTFLoader 取不到贴图，而页面看上去还是「能开」。
CSP 只进构建产物，不进 `index.html`：开发期 Vite 靠动态 `<style>` 注样式。

`vercel.json` 另发一组安全响应头。缓存分两档：`/assets/` 里的产物带内容哈希，给一年 immutable；
`/models/` 下那份 GLB 文件名固定、没有哈希，只给七天。标成 immutable 的话，换了模型也刷不掉浏览器里那份旧的。
