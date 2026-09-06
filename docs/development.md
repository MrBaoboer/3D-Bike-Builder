# 开发指南

改接口先改这里，再改代码。

## 环境与命令

Node `^22.13.0 || >=24.0.0`。

```bash
npm install && npm run dev
```

开发服务器在 <http://localhost:5174>。

| 命令 | 做什么 |
|---|---|
| `npm run lint` | ESLint，只开 recommended |
| `npm test` | `node:test`：清单校验器与清单读取层的单测 |
| `npm run verify` | 清单 × GLB 离线逐条对账 |
| `npm run build` | Vite 构建到 `dist/` |
| `npm run check:code` | 以上四条，半分钟。CI 对每个 PR 跑这一条 |
| `npm run smoke` | Playwright 起 `dist/`，宽窄两种画幅各真装一遍，几分钟 |
| `npm run check` | `check:code` + `smoke` |
| `npm run frames` | 逐步量取景：主体在不在舞台中央、整车照贴没贴边 |
| `npm run shots` | 从 `dist/` 实拍 README 的六张截图到 `docs/shots/` |
| `npm run live` | 比对线上两处发的是不是同一份产物 |
| `npm run model` | 打印整车装配树 |

`smoke`、`frames`、`shots` 都从 `dist/` 起页面，先 `npm run build`。每一条查什么、怎么改，见 [testing.md](testing.md)。

页面把共享上下文与引擎挂在 `window.__ctx` / `window.__engine` 上，工具脚本与探针都从这里进。

## 目录

```
assets/    bike.manifest.json 装配清单（唯一事实来源）· CREDITS.md 第三方素材署名
public/    models/CarbonFrameBike.glb 整车模型，12 MB，第三方素材
src/
  main.js    唯一的组装处，也是唯一读清单 JSON 的地方
  styles.css 画布与封面
  core/      bom 清单读取 · state 状态
  render/    stage 舞台 · bike 整车 · bolt 程序化螺丝与工具 · fx 特效
  interact/  slide 推入 · screw 旋入 · pick 指到哪件报哪件
  audio/     sfx 实时合成音效，不加载音频文件
  ui/        hud 界面 · icons 图标 · guides 三维箭头 · styles/ 四份 CSS
  app/       engine 分步引擎 · build 此刻车上该有哪些件
  steps/     acts 步骤表 · util 取景现算与共用铺陈
  util/      tween 补间
tools/     门禁与量尺，见 testing.md
docs/      技术文档 · shots/ README 用的截图
```

`tools/` 回答「有没有坏」，进版本库，CI 与提交前跑。一次性的探针回答「现在是多少」，放 `.analysis/`，不进版本库；写法照 `tools/frame-audit.mjs`：起 `dist/`，打开页面，在页内 `evaluate` 里量，把数打出来。
开发期另有 `/__shot`（`vite.config.js` 的 `dev-shot` 中间件）：页面把画布的 dataURL POST 过来，落到 `.shots/`，给无法直接截屏的环境用。

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

界面层不自带内容。章节名由 `steps/acts.js` 的 `PHASES` 提供，`hud.setChapters(steps, phases)` 的第二个参数必传。

样式的层叠顺序：`ui/styles/tokens.css` 令牌 → `ui/styles/base.css` 重置与无障碍 → `styles.css` 画布与封面 → `ui/styles/chrome.css` 常驻界面 → `ui/styles/surfaces.css` 覆盖层。
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
- `stage.updaters` 的回调收 `(dt, t, slow)`，三个时间量各有各的封顶，见 [camera.md](camera.md#主循环的时间量)。

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

**`installs` 是整份课程的骨架。** `app/build.js` 据此现推「走到第 i 步时车上该有哪些件」：还没轮到的不在场、正在装的在场但由这一步自己摆、装过的回到装配位。
漏声明一件，它就会从头到尾挂在画面上。`installs` 与 `fastens` 也是出门前自检的依据，每一行靠它连回对应的步骤。

**`enter()` 不能返回等用户动手的 Promise。** 引擎 `await` 它，在里面等用户，`engine.busy` 就永远不落，翻页、冒烟、自动路径全部卡死。到位之后要做的事走 `onDone` 回调。

一步装多件时默认逐件拖装（摇臂、刹把）。两件在台面上已连成一体的（直装牙盘锁在右曲柄上），加 `follow: { 从动件: 驱动件 }`：从动件不单独拖、不长箭头，抓哪件都驱动整组、同一个 u 走完全程、坐实一起记账。
前提是两件在清单里同 `dir` 同 `gap`，飞行中相对位置才不变。

`cam` 一律由 `steps/util.js` 现算，不写常量，四个入口见 [camera.md](camera.md#取景的四个入口)。

文案能少则少。常驻文字只有步名与一行旁白；说明卡只写物理原因看不见的那几处：反牙、对角、预紧、直装接口、最小插入线。

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

拧螺丝的步骤，机位要正对螺栓轴、从螺栓头那一侧看。旋入靠指针绕轴画圈读角度；轴躺进屏幕平面时（`|视线·轴| ≤ 0.3`）拖拽平面几乎与视线平行，读角度退化成沿屏幕切向的位移除以半径，手感差一截。
这几步的机位都是手写的（`frameBolts` 的 `az/el`、`shot` 的 `cam`），改的时候守住这一条。

## 翻页：一下演一件

翻页永远不被拦，但**「下一步」在这一步还有活没干完时，先演一件，不翻页**：两条摇臂按两下，四颗面盖螺丝按四下，做完再按一下才走。
一口气演完整步不行，四颗螺丝在一两秒里连着转完，「对角、分两轮」来不及看清。判据见 `engine.pending` 与 `engine.finishPending()`，两者都只认各原语的 `session.pending`。

**上一下还没走完时按的那一下要攒着**（最多四下），忙完一下一下补上。「拆开看看」进场要演一秒多，连按两下方向键很常见。直接跳步（`engine.jump()`）清空攒下的。

空格与方向键的让位规矩不一样：

- **空格**是按钮的激活键，焦点落在任何控件上都得让开。
- **方向键**在按钮上没有默认动作，只让给自己要用方向键的控件（进度轨、菜单、文本框）。它们各自 `preventDefault()`，引擎认这一下就够。

两键按同一条处理的话，用鼠标点完「下一步」之后方向键整个失灵，焦点正在那枚按钮上。
