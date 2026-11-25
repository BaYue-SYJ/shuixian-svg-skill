# shuixian-svg

> 一个 skill：让 AI 帮你做出可交互的宽幅 SVG 动画场景 —— 单个 HTML 文件，会日夜循环、能点、有一段值得按的「秀」和原创配乐。

这不是一个代码库，而是一套**做法**：把「画风怎么画、秀怎么设计、配乐怎么不跑调、交件前查什么」写成可执行的规则，交给支持 `SKILL.md` 约定的 AI 助手（WorkBuddy / Claude Code 等）加载，再由它产出成品。规则来自 101 组场景作品一路修正后的沉淀。

成品形态固定：一个 HTML 文件，双击就开，1600×600 的宽幅插画，会随时间流动，鼠标移过去有视差，点东西有反应。

## 核心功能

| 能力 | 具体是什么 |
|---|---|
| 单文件交付 | 只用 SVG + 原生 JavaScript，零外部库、零图片、零音频文件，断网也能跑 |
| 日夜循环 | 白天 → 黄昏 → 夜晚 → 清晨，四时段连续过渡，一颗按钮轮播，另有时间拉杆手动定格 |
| 天气系统 | 按当地气候设计多种天气，云量、光线、雨雪粒子、地面反光、人物行为跟着一起变 |
| 可点物件 | 6 个以上，点角色 / 车 / 动物各有专属反应，不额外堆重复按钮 |
| 「秀」 | 1～2 段约 20 秒的小剧场，有开场、高潮、收尾，规模够大；再按一次优雅停止（画面冻成残影淡出，音符与计时器全部收干净） |
| 原创配乐 | Web Audio 实时合成，无音档。旋律强拍锁在和弦音上，钟琴只用整数倍泛音 |
| 20 种画风 | 蓝晒图、3D 说明图、水墨、立体派、线稿、扁平、剪纸层叠、像素、浮世绘、霓虹、极简几何、水彩、印象派、彩绘玻璃、复古石版、黏土定格、刺绣、羊毛毡、纸雕光影、CGI 电影质感 |
| 可达性 | `prefers-reduced-motion` 降级、分页隐藏暂停、键盘 `tabindex` + Enter 可触发、手机宽度控制列换行 |

## 目录结构

```
shuixian-svg/
├── SKILL.md                     工作流程 + 不能妥协的硬规则
├── references/
│   ├── styles.md                20 种画风的具体 SVG 画法（含性能提醒）
│   ├── show-and-motion.md       秀的设计、表演氛围、优雅停止、移动物规则
│   ├── sound.md                 声音架构、音量、配乐和声规则
│   └── checklist.md             交件前逐条自检的 35 项清单
├── templates/
│   └── scene-skeleton.html      可直接打开的最小场景骨架（577 行，含完整交互与示例场景「港边小镇」）
└── tools/
    ├── showstop.js              「表演再按一下优雅停止」共用模块
    ├── showstop_inject.py       把模块 + adapter 注入场景 HTML 的小工具
    └── scene-bgm.js             可换曲谱的常驻背景配乐引擎
```

## 安装

这个目录本身就是一个 skill，安装 = 把整个 `shuixian-svg` 文件夹放进客户端的 skills 目录。没有构建步骤，也没有额外配置。

**WorkBuddy（对本机所有项目生效）**

```bash
cp -r shuixian-svg ~/.workbuddy/skills/
```

**WorkBuddy（只对当前项目生效）**

```bash
cp -r shuixian-svg <项目目录>/.workbuddy/skills/
```

**Claude Code 等其他支持 `SKILL.md` 的客户端**：放进对应的 skills 目录（如 `~/.claude/skills/`），重启客户端即可被识别。

## 使用示例

直接用自然语言说你要什么场景，skill 会先把内容清单列给你确认，再动手写代码：

> 帮我做一组「京都伏见稻荷的千本鸟居」SVG 动画场景，用剪纸层叠风格，秀是夜里鸟居灯笼一路亮上山。

> 做一个「优胜美地隧道景观的四季」，要能看出季节在换，天气多变，用复古石版海报风格。

> 做一个「香港维多利亚港的跨年烟花」，霓虹夜景画风，秀是烟花铺满左右两侧。

**它遵循的流程**

1. 选题与命名 —— 用大众熟悉的地名，名称是「地名 + 画面主角 / 时刻」
2. 选画风 —— 从 20 种里挑一种，整个画面（连人物造型）统一
3. 列内容确认 —— 地标、人物、动物、交通、环境动态、天气、可点物件、秀
4. 复制骨架开工 —— 保留 `body > main.wrap > figure.stage > svg#scene` 结构与控制列
5. 由远到近分层画 —— 天空 → 远景 → 中景地标 → 地面 → 人物物件 → 前景 → 特效 → `#cineFx`
6. 加动态与交互 —— 日夜、天气、点击反应、鼠标视差
7. 加秀与声音 —— 秀注入 ShowStop，配乐照和声规则写
8. 自检 —— 照 `checklist.md` 逐条打勾，浏览器实开确认不卡不报错

## 工具用法

### templates/scene-skeleton.html

复制一份就是起点，直接用浏览器打开就能跑。里面已有一套完整示例场景（港边小镇：山丘、小屋、帆船、海鸥），按钮为 `btnShow` / `btnWeather` / `btnSound` / `btnTime` / `btnAuto`，时间拉杆 `timeRange`，表演氛围图层 `#cineFx`，场景根节点 `svg#scene`。

### tools/showstop_inject.py

给带「秀」的场景注入优雅停止能力。先照 `references/show-and-motion.md` 写一个 adapter，声明怎么判断正在表演、怎么保存和还原用户设置：

```js
ShowStop.init({
  on: () => show.on || null,                     // 多段秀时返回种类，例如 'fire'、'stars'
  btn: { fire: 'btnFire', stars: 'btnStars' },   // 只有一段秀时可直接写 'btnShow'
  save: () => ({ t: T.t, auto: T.auto }),        // 表演前记下用户设置
  restore: s => { T.t = s.t; T.auto = s.auto; applyTime(true); },
  stop: () => { endShow(); setCine(0); }
});
```

然后注入（可重复执行，会自动替换旧版）：

```bash
python3 tools/showstop_inject.py 场景.html adapter.js
```

注入位置：模块放在主程序第一个 `<script>` 之前，adapter 放在主程序最后一个 `})();` 之前（IIFE 内部，能看见 `show` 等变量）。要指定位置就传第三个参数当锚点字符串：

```bash
python3 tools/showstop_inject.py 场景.html adapter.js "/* 动画主循环 */"
```

Windows 环境若没有 `python3` 别名，改用 `python` 调用。

### tools/scene-bgm.js

常驻背景配乐引擎。换曲子只改脚本里的 `SCORE` 音符数组（`"A4"`、`"C#5"`、`"Eb4"` 是音名，`"-"` 是休止，数字是长度格数），对外只暴露这几个接口：

| 接口 | 作用 |
|---|---|
| `window.sceneMusic.start()` | 开始播放 |
| `window.sceneMusic.stop()` | 停止 |
| `window.sceneMusic.toggle()` | 切换开关 |
| `window.sceneMusic.duck(秒)` | 临时让位（秀的音乐响起前压低配乐） |
| `window.sceneMusic.playing` | 只读，当前是否在播 |

页面存在 `#btnMusic` 按钮时自动绑定点击。

## 硬规则（摘要）

完整版在 `SKILL.md`，这些是不能妥协的部分：

- **技术**：单 HTML；`viewBox="0 0 1600 600"`；颜色全部收在开头 CSS 变量，白天夜晚各一组；支持 `prefers-reduced-motion`；代码加中文注释分区
- **画面**：上半部不能空；白天也要热闹；画面里不放作品名称或编号
- **原创**：只画原创内容，不用既有角色、品牌、商标；参考电影游戏只取氛围不取造型
- **操作**：可点的角色不再配重复按钮；时段只做一颗轮播按钮；底部说明最多约 5 条短句
- **表演**：不用上下黑边，改用 `#cineFx` 暗角 + 中央微亮；每段秀 20 秒内；两段秀不能同时进行；再按一次要优雅停止
- **声音**：不合成人声；不自动冒出喇叭或汽笛；旋律强拍必须在和弦音上

## 依赖

- **成品**：现代浏览器即可（Chrome / Edge / Safari / Firefox），无第三方库、无网络请求、无构建
- **工具**：`tools/showstop_inject.py` 需要 Python 3，只用标准库 `re` / `sys` / `pathlib`，无第三方包
- **skill 本身**：无运行时依赖，无安装脚本

## 参考文档

| 文件 | 什么时候读 |
|---|---|
| `references/styles.md` | 定风格时，读对应画风那一段（20 种写法 + 各自的性能陷阱） |
| `references/show-and-motion.md` | 写秀与动画之前：秀的设计原则、ShowStop 接入、移动物朝向与避让 |
| `references/sound.md` | 写声音之前：双轨架构、音量数值、和声规则、音效禁区 |
| `references/checklist.md` | 交件前逐条自检，全过才算完成 |
