# DESIGN.md — 衣境 AI · 服饰创作台

> 一间明亮的数字影棚：对话即创作，素材伸手可得，紫罗兰的一点电流让工具有了灵性。

**产品**：面向服饰商家的 AI 图片/视频生成工具（LUI 对话式布局）
**核心区块**：① 对话创作流 + 图片上传 ② 商家资产库 ③ 商家场景库 ④ 公共场景库

---

## 1. Visual Theme & Atmosphere

**Style**: Clean AI Workbench（极简克制骨架 × 温暖商务密度 × 紫罗兰 AIGC 语言）
**Keywords**: 明亮、通透、直接、克制、专业、一点未来感
**Tone**: 高效可信的生产力工具 — NOT 炫技的营销页、NOT 拥挤的电商后台
**Feel**: 像一间白墙影棚，光线均匀，道具分门别类放在手边；唯一的色彩来自窗边那道紫蓝色的极光。

**Interaction Tier**: L2 流畅交互（工具型界面：微交互丰富、无重滚动动效、无 scroll-jacking）
**Dependencies**: CSS + 原生 JS（IntersectionObserver / rAF），零外部动效库

**布局纲领（LUI）**：
- 三栏应用外壳：`左侧导航栏(232px) | 对话创作流(自适应) | 素材面板(324px)`
- 对话流是绝对主角；一切素材（上传图、资产、场景）以 **chip 芯片** 形式进入输入框，作为生成上下文
- 素材面板三个页签：**资产库 / 我的场景 / 公共场景**，可整体收起
- 空状态 = 欢迎语 + 快捷创作卡片；这是唯一允许"氛围感"的时刻

## 2. Color Palette & Roles

```css
:root {
  /* Backgrounds */
  --bg: #F6F6FA;               /* 页面底色（冷调浅灰） */
  --surface: #FFFFFF;          /* 卡片、面板、输入框 */
  --surface-alt: #F1F1F7;      /* 次级填充、代码块、气泡 */
  --surface-hover: #ECECF4;    /* 列表项/图标按钮 hover */

  /* Borders */
  --border: #E8E8F0;           /* 默认边框、分割线 */
  --border-hover: #D6D6E4;     /* 悬停边框 */

  /* Text */
  --text: #17172B;             /* 标题、正文主体 */
  --text-secondary: #55556B;   /* 描述、次要信息 */
  --text-tertiary: #9C9CB0;    /* 占位符、时间戳、标签 */

  /* Accent — 紫罗兰 AI */
  --accent: #6C4DF6;           /* 主强调：选中态、链接、focus */
  --accent-hover: #5938E8;
  --accent-weak: #F0EDFE;      /* 选中底色、强调 chip 背景 */
  --accent-2: #3D7BFD;         /* 渐变终点（蓝） */
  --grad: linear-gradient(120deg, #7B4DFF 0%, #3D7BFD 100%);  /* 仅限：主按钮 / logo / 欢迎语关键词 / 进度条 */

  /* RGB variants for rgba() */
  --bg-rgb: 246, 246, 250;
  --surface-rgb: 255, 255, 255;
  --text-rgb: 23, 23, 43;
  --accent-rgb: 108, 77, 246;
  --accent-2-rgb: 61, 123, 253;

  /* Semantic */
  --success: #16A34A;
  --error: #E5484D;
  --warning: #F5A524;
}
```

**Color Rules:**
- 所有颜色一律通过 CSS 变量引用，**禁止硬编码 hex**
- 渐变 `--grad` 是稀缺资源，**仅允许**出现在：logo、主按钮（新建/发送）、AI 头像、生成进度条、欢迎语关键词；**禁止**用于正文文字、卡片、边框、大面积背景
- 大面积永远是 `--bg / --surface` 的白灰层叠，紫色只做"电流"点缀
- 图片内容自带色彩，容器一律中性色，不与图片抢戏
- 语义色仅用于状态反馈（成功/失败/警告 toast、错误边框），不做装饰

## 3. Typography Rules

**Font Stack:**
```css
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+SC:wght@400;500;700&family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap');

:root {
  --font-sans: "Noto Sans SC", "Inter", "PingFang SC", "MiSans", "Microsoft YaHei", system-ui, sans-serif;
  --font-mono: "JetBrains Mono", "SFMono-Regular", Consolas, monospace;
}
```

| Role | Font | Size | Weight | Line Height | Letter Spacing |
|------|------|------|--------|-------------|----------------|
| 欢迎语 H1 | Sans | 30px | 700 | 1.4 | 0.02em |
| 面板/区块标题 H2 | Sans | 15px | 600 | 1.5 | 0.02em |
| 卡片标题 H3 | Sans | 14px | 600 | 1.6 | 0.02em |
| 正文 Body | Sans | 14px | 400 | 1.7 | 0.02em |
| 标签 Label / eyebrow | Sans | 12px | 500 | 1.5 | 0.04em |
| 参数/种子 Mono | Mono | 12px | 400 | 1.6 | — |

**Typography Rules:**
- 中文字族在前（Noto Sans SC），英文数字回退 Inter；**禁止只配英文字体**
- 正文 ≥ 14px、行高 ≥ 1.7、字距 0.02em（中文可读性红线）
- 层级靠字重 + 颜色（--text / secondary / tertiary）建立，不靠字号跳跃
- **NEVER use**: 衬线体、手写体、Orbitron 类科幻字体、系统默认宋体回退

**Text Decoration:**
- 欢迎语 H1 中的**关键词**（如"衣境"/"图片与视频"）：允许 `--grad` 渐变填充 + 缓慢流动（品牌时刻，全站唯一的文字渐变）
- 其余任何标题/正文：无渐变、无投影（克制风格红线）
- 链接与可点文字：颜色过渡 hover（--text-secondary → --accent），不加下划线动画以外的装饰

## 4. Component Stylings

### Buttons

```css
/* 主按钮（发送键 / 新建创作 / 生成） */
.btn-primary {
  display: inline-flex; align-items: center; justify-content: center; gap: 6px;
  height: 40px; padding: 0 18px; border: none; border-radius: 12px;
  background: var(--grad); color: #fff;
  font: 500 14px/1 var(--font-sans); letter-spacing: 0.02em;
  cursor: pointer; user-select: none;
  transition: transform .15s ease, box-shadow .2s ease, filter .2s ease;
}
.btn-primary:hover  { transform: translateY(-1px); box-shadow: 0 6px 20px rgba(var(--accent-rgb), .32); }
.btn-primary:active { transform: translateY(0) scale(.97); box-shadow: none; }
.btn-primary:focus-visible { outline: 2px solid var(--accent); outline-offset: 2px; }
.btn-primary:disabled { filter: grayscale(.6); opacity: .45; cursor: not-allowed; transform: none; box-shadow: none; }

/* 次级按钮（面板操作 / 结果卡操作） */
.btn-ghost {
  display: inline-flex; align-items: center; gap: 6px;
  height: 34px; padding: 0 12px; border-radius: 10px;
  background: transparent; border: 1px solid var(--border);
  color: var(--text-secondary); font: 500 13px/1 var(--font-sans);
  cursor: pointer; transition: all .18s ease;
}
.btn-ghost:hover  { border-color: var(--border-hover); background: var(--surface-hover); color: var(--text); }
.btn-ghost:active { transform: scale(.96); }
.btn-ghost:focus-visible { outline: 2px solid var(--accent); outline-offset: 2px; }
.btn-ghost:disabled { opacity: .4; cursor: not-allowed; }

/* 图标按钮（上传 / 收起面板 / 更多） */
.btn-icon {
  width: 36px; height: 36px; display: grid; place-items: center;
  border: none; border-radius: 10px; background: transparent;
  color: var(--text-secondary); cursor: pointer; transition: all .18s ease;
}
.btn-icon:hover  { background: var(--surface-hover); color: var(--text); }
.btn-icon:active { transform: scale(.92); }
.btn-icon:focus-visible { outline: 2px solid var(--accent); outline-offset: 2px; }
.btn-icon.active { background: var(--accent-weak); color: var(--accent); }
```

### Cards（快捷创作卡 / 素材卡 / 结果卡）

```css
/* 通用卡片：Spotlight 悬停（鼠标跟随光斑，rAF 节流） */
.card {
  position: relative; overflow: hidden;
  background: var(--surface); border: 1px solid var(--border); border-radius: 16px;
  transition: transform .25s cubic-bezier(.16,1,.3,1), box-shadow .25s ease, border-color .25s ease;
}
.card::before {  /* spotlight 层 */
  content: ''; position: absolute; inset: 0; border-radius: inherit;
  background: radial-gradient(220px circle at var(--mx, 50%) var(--my, 50%),
              rgba(var(--accent-rgb), .07), transparent 65%);
  opacity: 0; transition: opacity .3s ease; pointer-events: none;
}
.card:hover { transform: translateY(-2px); border-color: var(--border-hover); box-shadow: 0 8px 24px rgba(var(--text-rgb), .07); }
.card:hover::before { opacity: 1; }
.card:focus-visible { outline: 2px solid var(--accent); outline-offset: 2px; }

/* 素材缩略卡（资产/场景通用）：图占满，信息浮层 */
.asset-card { aspect-ratio: 3/4; cursor: pointer; }
.asset-card img { width: 100%; height: 100%; object-fit: cover; display: block;
  transition: transform .5s cubic-bezier(.16,1,.3,1); }
.asset-card:hover img { transform: scale(1.05); }
.asset-card .meta {  /* 底部信息条 */
  position: absolute; inset: auto 0 0 0; padding: 20px 10px 8px;
  background: linear-gradient(transparent, rgba(23,23,43,.62));
  color: #fff; font-size: 12px;
}
.asset-card .use {  /* 悬停出现的"使用"按钮 */
  position: absolute; top: 8px; right: 8px; opacity: 0; transform: translateY(-4px);
  transition: all .2s ease;
}
.asset-card:hover .use, .asset-card:focus-within .use { opacity: 1; transform: none; }
.asset-card.selected { border-color: var(--accent); box-shadow: 0 0 0 2px rgba(var(--accent-rgb), .25); }
```

### Composer（输入区 — LUI 的心脏）

```css
.composer {
  background: var(--surface); border: 1.5px solid var(--border); border-radius: 20px;
  box-shadow: 0 2px 8px rgba(var(--text-rgb), .04);
  transition: border-color .2s ease, box-shadow .2s ease;
}
.composer:focus-within {
  border-color: var(--accent);
  box-shadow: 0 0 0 3px rgba(var(--accent-rgb), .12), 0 4px 16px rgba(var(--text-rgb), .06);
}
.composer.dragover {  /* 拖拽上传态 */
  border-style: dashed; border-color: var(--accent); background: var(--accent-weak);
}
.composer textarea {
  width: 100%; border: none; outline: none; resize: none; background: transparent;
  font: 400 14px/1.7 var(--font-sans); color: var(--text); letter-spacing: .02em;
  max-height: 160px;
}
.composer textarea::placeholder { color: var(--text-tertiary); }
```

### Chips（上下文芯片：上传图 / 资产 / 场景）

```css
.chip {
  display: inline-flex; align-items: center; gap: 6px;
  height: 30px; padding: 0 8px 0 4px; border-radius: 8px;
  background: var(--surface-alt); border: 1px solid var(--border);
  font-size: 12px; color: var(--text-secondary);
  animation: chipIn .25s cubic-bezier(.34,1.56,.64,1) both;
}
.chip img { width: 22px; height: 22px; border-radius: 5px; object-fit: cover; }
.chip.scene { background: var(--accent-weak); border-color: transparent; color: var(--accent); }
.chip .remove { width: 16px; height: 16px; border-radius: 50%; display: grid; place-items: center;
  cursor: pointer; color: inherit; opacity: .6; transition: opacity .15s; }
.chip .remove:hover { opacity: 1; background: rgba(var(--text-rgb), .08); }
@keyframes chipIn { from { opacity: 0; transform: scale(.8); } to { opacity: 1; transform: scale(1); } }
```

### Navigation（左侧栏项 / 面板页签）

```css
/* 侧栏项 */
.nav-item {
  display: flex; align-items: center; gap: 10px; height: 40px; padding: 0 12px;
  border-radius: 10px; color: var(--text-secondary); font-size: 14px;
  cursor: pointer; transition: all .18s ease; border: none; background: transparent; width: 100%;
}
.nav-item:hover { background: var(--surface-hover); color: var(--text); }
.nav-item.active { background: var(--accent-weak); color: var(--accent); font-weight: 500; }
.nav-item:focus-visible { outline: 2px solid var(--accent); outline-offset: -2px; }

/* 面板页签：滑动指示条 */
.tabs { position: relative; display: flex; background: var(--surface-alt); border-radius: 12px; padding: 3px; }
.tab { flex: 1; height: 32px; border: none; background: transparent; border-radius: 9px;
  font: 500 13px/1 var(--font-sans); color: var(--text-secondary); cursor: pointer;
  position: relative; z-index: 1; transition: color .2s ease; }
.tab.active { color: var(--text); }
.tab:focus-visible { outline: 2px solid var(--accent); outline-offset: -2px; }
.tab-thumb {  /* 滑动白块 */
  position: absolute; top: 3px; bottom: 3px; border-radius: 9px; background: var(--surface);
  box-shadow: 0 1px 4px rgba(var(--text-rgb), .08);
  transition: left .25s cubic-bezier(.16,1,.3,1), width .25s cubic-bezier(.16,1,.3,1);
}
```

### Links

```css
.link { color: var(--accent); text-decoration: none; position: relative; }
.link::after { content: ''; position: absolute; left: 0; bottom: -1px; width: 0; height: 1px;
  background: var(--accent); transition: width .25s ease; }
.link:hover::after { width: 100%; }
.link:focus-visible { outline: 2px solid var(--accent); outline-offset: 2px; border-radius: 2px; }
```

### Tags / Badges（场景分类 / 视频时长 / 状态）

```css
.tag { display: inline-flex; align-items: center; height: 24px; padding: 0 10px;
  border-radius: 999px; font-size: 12px; letter-spacing: .04em;
  background: var(--surface-alt); color: var(--text-secondary); border: 1px solid transparent;
  cursor: pointer; transition: all .18s ease; }
.tag:hover { color: var(--text); border-color: var(--border-hover); }
.tag.active { background: var(--accent-weak); color: var(--accent); font-weight: 500; }
.badge { position: absolute; padding: 2px 8px; border-radius: 6px; font-size: 11px;
  background: rgba(23,23,43,.55); color: #fff; backdrop-filter: blur(6px); }
```

### Toast / Skeleton / Progress

```css
.toast { position: fixed; top: 20px; left: 50%; transform: translateX(-50%);
  display: flex; align-items: center; gap: 8px; height: 40px; padding: 0 16px;
  background: var(--text); color: #fff; border-radius: 12px; font-size: 13px;
  box-shadow: 0 12px 32px rgba(var(--text-rgb), .18); z-index: 500;
  animation: toastIn .3s cubic-bezier(.34,1.56,.64,1) both; }
@keyframes toastIn { from { opacity: 0; transform: translate(-50%, -12px); }
                     to   { opacity: 1; transform: translate(-50%, 0); } }

.skeleton { position: relative; overflow: hidden; background: var(--surface-alt); border-radius: 12px; }
.skeleton::after { content: ''; position: absolute; inset: 0;
  background: linear-gradient(90deg, transparent, rgba(var(--surface-rgb), .75), transparent);
  transform: translateX(-100%); animation: shimmer 1.4s ease infinite; }
@keyframes shimmer { to { transform: translateX(100%); } }

.progress { height: 4px; border-radius: 2px; background: var(--surface-alt); overflow: hidden; }
.progress > i { display: block; height: 100%; border-radius: 2px; background: var(--grad);
  transition: width .3s ease; }
```

## 5. Layout Principles

**App Shell（无页面滚动，各栏内部独立滚动）:**
```css
.app { display: grid; grid-template-columns: 232px 1fr 324px; height: 100vh; }
/* 素材面板收起时 → 232px 1fr 0；侧栏收起时 → 64px 1fr 324px */
```

**Container:**
- 对话流内容列 max-width: **760px**，水平居中
- Composer 与消息列同宽，吸底（sticky bottom），下方留 24px 呼吸
- 素材面板内部 padding: 16px；素材网格 2 列，gap 10px

**Spacing Scale（8pt 基准）:** 4 / 8 / 12 / 16 / 24 / 32 / 48
- 消息与消息间距: 32px；消息内元素间距: 12px
- 卡片内边距: 16px；面板区块间距: 24px

**Grid:**
```css
.suggest-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 12px; }   /* 快捷创作卡 */
.result-grid  { display: grid; grid-template-columns: repeat(2, 1fr); gap: 8px; }    /* 生成结果 2×2 */
.asset-grid   { display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px; }   /* 素材面板 */
```

**圆角体系:** 面板/大容器 20px · 卡片 16px · 按钮/输入件 10–12px · chip 8px · 图片 12px

## 6. Depth & Elevation

| Level | Treatment | Use |
|-------|-----------|-----|
| Flat | 无阴影，仅 1px `--border` | 侧栏、面板、分割 |
| Subtle | `0 2px 8px rgba(23,23,43,.04)` | Composer 静息、消息卡 |
| Raised | `0 8px 24px rgba(23,23,43,.07)` | 卡片 hover、下拉菜单 |
| Overlay | `0 24px 64px rgba(23,23,43,.16)` | 抽屉、灯箱、移动端面板 |
| Accent Glow | `0 6px 20px rgba(108,77,246,.32)` | 仅主按钮 hover |

规则：同一元素**边框与重阴影不叠加**；层级靠阴影梯度，不靠加深边框。

## 7. Animation & Interaction

**Motion Philosophy**: 反馈即时、入场轻盈、生成过程有仪式感；只动 opacity/transform，一切 ≤ 0.5s，绝不阻挡下一步操作。
**Tier**: L2（工具型适配：无滚动 pin、无视差滥用、无 scroll-jacking）

### Dependencies
```html
<!-- 无外部库：CSS keyframes + IntersectionObserver + rAF -->
```

### Entrance Animation（首屏装载 & 消息入场）
```css
@keyframes fadeInUp { from { opacity: 0; transform: translateY(20px); } to { opacity: 1; transform: none; } }
@keyframes popIn    { from { opacity: 0; transform: scale(.94); }      to { opacity: 1; transform: none; } }

.enter    { animation: fadeInUp .5s cubic-bezier(.16,1,.3,1) both; }
.enter-d1 { animation-delay: .06s; } .enter-d2 { animation-delay: .12s; }
.enter-d3 { animation-delay: .18s; } .enter-d4 { animation-delay: .24s; }

/* 欢迎语 H1：SplitText 式逐字浮入（JS 拆字后套用） */
.greet-char { display: inline-block; opacity: 0; transform: translateY(90%);
  animation: charUp .55s cubic-bezier(.16,1,.3,1) both; }
@keyframes charUp { to { opacity: 1; transform: none; } }
/* JS: 每字 delay = i * 28ms */

/* 欢迎语关键词：全站唯一渐变文字，缓慢流光（GradientText） */
.grad-text { background: linear-gradient(110deg, #7B4DFF, #3D7BFD, #7B4DFF);
  background-size: 200% 100%; -webkit-background-clip: text; background-clip: text;
  -webkit-text-fill-color: transparent; animation: gradFlow 6s linear infinite; }
@keyframes gradFlow { to { background-position: 200% 0; } }
```

### 流式输出（TextType — 助手文字逐字浮现）
```js
function streamText(el, text, speed = 26) {          // LUI 的核心动效
  el.textContent = ''; let i = 0;
  return new Promise(res => {
    (function tick() {
      el.textContent = text.slice(0, ++i);
      i < text.length ? setTimeout(tick, speed) : res();
    })();
  });
}
```

### 生成仪式感（skeleton → 结果 stagger 揭晓）
```js
// 1. 结果卡先渲染 2×2 .skeleton + 渐变进度条（假进度 0→92% 缓动）
// 2. "完成"后：skeleton 淡出，图片以 popIn + 交错 80ms 逐张揭晓
// 3. 操作按钮行 fadeInUp 跟上
```

### Spotlight 卡片（rAF 节流的鼠标跟随光斑）
```js
let raf = null;
document.addEventListener('pointermove', e => {
  if (raf) return;
  raf = requestAnimationFrame(() => {
    document.querySelectorAll('.card:hover').forEach(c => {
      const r = c.getBoundingClientRect();
      c.style.setProperty('--mx', `${e.clientX - r.left}px`);
      c.style.setProperty('--my', `${e.clientY - r.top}px`);
    });
    raf = null;
  });
}, { passive: true });
```

### ClickSpark（发送按钮点击溅出粒子 — 巧思彩蛋）
```js
// 点击发送：8 个 4px 圆点沿圆周飞散，450ms 后自毁；纯 transform+opacity
```

### 面板 / 抽屉滑动
```css
.panel { transition: width .35s cubic-bezier(.16,1,.3,1), opacity .25s ease; }
.drawer { transform: translateX(100%); transition: transform .35s cubic-bezier(.16,1,.3,1); }
.drawer.open { transform: none; }
```

### 氛围背景（仅空状态：双色光晕缓慢漂移，无 blur 滤镜）
```css
.orb { position: absolute; width: 480px; height: 480px; border-radius: 50%; pointer-events: none;
  background: radial-gradient(circle, rgba(var(--accent-rgb), .10), transparent 62%);
  animation: drift 18s ease-in-out infinite alternate; }
.orb--blue { background: radial-gradient(circle, rgba(var(--accent-2-rgb), .08), transparent 62%);
  animation-duration: 22s; animation-direction: alternate-reverse; }
@keyframes drift { from { transform: translate(-6%, -4%) scale(1); } to { transform: translate(8%, 6%) scale(1.12); } }
```

### Hover & Focus States
- 所有可交互元素**必有** hover + `:focus-visible`（2px accent 外圈）
- 列表项/图标按钮：底色浮现（.18s）；卡片：上浮 2px + 阴影 + spotlight；图片：内部 scale 1.05
- 发送按钮 hover 带 accent glow；textarea focus 时 composer 外圈 3px 柔光

### Reduced Motion
```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: .01ms !important; animation-iteration-count: 1 !important;
    transition-duration: .01ms !important;
  }
  .orb { display: none; }
  /* JS 侧：streamText 直接整段落字，跳过逐字 */
}
```

**签名动效清单（L2 六类核查）**：
| 类别 | 落点 |
|------|------|
| Text — H1 | 欢迎语逐字浮入（SplitText 式）+ 关键词渐变流光（GradientText） |
| Text — H2 | 面板页签切换时区块标题 BlurText 式淡入 |
| Text — Body | 助手回复 streamText 逐字流式输出（TextType） |
| Animation — 元素级 | 发送键 ClickSpark 粒子 + accent glow hover |
| Component | SpotlightCard（快捷卡/素材卡鼠标跟随光斑）+ 页签滑动指示条 |
| Background | 空状态双色光晕漂移（radial-gradient，零 blur 滤镜） |

## 8. Do's and Don'ts

### Do
- 一切颜色走 CSS 变量；一切间距走 8pt 梯度
- 素材进入创作 = 生成 chip 芯片进 composer，让"上下文"看得见、可移除
- 生成中用 skeleton shimmer + 渐变进度条，绝不用转圈 spinner
- 图标统一线性风格（lucide 风，stroke 1.75，18/20px），语义直白
- 每个面板都有空状态设计（插图化图标 + 一句引导 + 动作按钮）
- 长列表/网格用 stagger 入场（≤ 0.6s 总时长），首屏一次性，不重复触发
- 触摸目标 ≥ 44×44px；键盘可达（Tab 顺序 = 视觉顺序）
- 文案直接说人话："生成模特图"而不是"开始推理"

### Don't
- ❌ 硬编码 hex / 随意像素值（必须走变量与间距梯度）
- ❌ Emoji 当图标（工具调性红线）
- ❌ 正文/次级标题用渐变或投影（渐变只属于 logo、主按钮、AI 头像、进度条、欢迎语关键词）
- ❌ 在卡片、边框、大面积背景上使用渐变（紫色泛滥 = 廉价感）
- ❌ 运动元素上使用 `filter: blur()`；backdrop-filter > 14px 或覆盖大面积滚动区
- ❌ scroll-jacking / Lenis / 滚动 pin（工具界面滚动必须原生）
- ❌ 纯色块占位图（一律真实图片：用户素材 > Unsplash）
- ❌ 边框 + 重阴影双重描边同一卡片
- ❌ 全局自定义光标；pointermove 不经 rAF 节流
- ❌ 弹窗嵌弹窗；面板抽屉同时打开超过一层遮罩

## 9. Responsive Behavior

**Breakpoints:**
| Name | Width | Key Changes |
|------|-------|-------------|
| Desktop | > 1200px | 三栏全开：侧栏 232 + 对话流 + 面板 324 |
| Tablet | 768–1200px | 素材面板变为右侧滑入抽屉（遮罩 rgba(23,23,43,.32)），侧栏收窄 64px 仅图标 |
| Mobile | < 768px | 单栏：侧栏与面板均为全屏抽屉；composer 吸底全宽；快捷卡 1 列；结果图 2 列 |

**Touch Targets:** ≥ 44×44px（chip 删除钮扩大热区到 32px）
**Collapsing Strategy:** 面板 → 抽屉 → 全屏抽屉；功能不减，只换容器

```css
@media (max-width: 1200px) {
  .app { grid-template-columns: 64px 1fr; }
  .panel { position: fixed; right: 0; top: 0; bottom: 0; width: min(360px, 92vw); z-index: 300; }
}
@media (max-width: 768px) {
  .app { grid-template-columns: 1fr; }
  .sidebar { position: fixed; left: 0; z-index: 300; transform: translateX(-100%); }
  .sidebar.open { transform: none; }
  .suggest-grid { grid-template-columns: 1fr; }
  .chat-col { padding: 0 16px; }
}
```

---

*Motion effects derived from [vue-bits](https://github.com/DavidHDev/vue-bits) by DavidHDev (MIT) 的 SplitText / GradientText / TextType / SpotlightCard / ClickSpark 思路，纯 CSS/JS 实现。*
