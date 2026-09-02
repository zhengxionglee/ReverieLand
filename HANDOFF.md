# 遐想之地 · ReverieLand 智能体交接文档

> 给接手这个项目的下一个 AI 助手看的。所有"为什么这么设计"、"哪里容易踩坑"、
> "用户在意什么"，都尽量在这里讲清楚。

---

## 0. 30 秒速览

- **项目**：个人诗集 / 文学站，主站名 **遐想之地 · ReverieLand**
- **技术栈**：Jekyll（Kramdown GFM）+ GitHub Pages，完全静态、零成本
- **在线**：`https://zhengxionglee.github.io/ReverieLand/`
- **仓库**：`github.com:zhengxionglee/ReverieLand.git`（main 分支即部署分支）
- **作者**：自写诗词/散文/对联/小说/歌词赏析的站长，**zhengxionglee**
- **体裁**：小说、古诗、古词、现代诗、对联、散文、弦外（歌词赏析）
- **Slogan**：不为无益之事，何以遣有涯之生
- **风格关键词**：衬线字体、宣纸色调（`#f8f5ee`）、朱红小印章、留白多

---

## 1. ⚠️ 安全告警（先看这个）

### Git remote 里塞了明文 token

```
$ git remote -v
origin  https://zhengxionglee:ghp_████████████████████████████（明文已脱敏，勿入库）@github.com/zhengxionglee/ReverieLand.git
```

**这个 token 已经在历史里，会跟着 git log 永久存在。** 优先级建议：

1. **先去 GitHub Settings → Developer settings → Personal access tokens → 立刻 Revoke 掉这个旧 token**（一旦我或别人发现了会先去做的）
2. 重新生成一个新 token，**scope 只勾 `public_repo`**（这个仓库是 public，最低权限就行）
3. **不要把新 token 直接写进 remote URL**，改用 `gh auth login` 或 SSH key（`git@github.com:zhengxionglee/ReverieLand.git`）
4. **如果只是临时操作**，可以用 `git -c credential.helper= cmd` 走交互输入；或者 `git credential-store` + 设置 Windows Credential Manager

**不要把带 token 的 git 命令贴到任何对外的地方**（包括截图、日志、AI 提示词）。

> 这个 token 是被前一个 AI 助手嵌进 remote URL 的（看 commit 7 之前的命令习惯）。接手后第一件事是清理。

---

## 2. 目录结构

```
Novel_website/  （D:\AI_Project\Novel_website）
├── _config.yml                # 站点配置（必读）
├── index.html                 # 主页（hero + 栏目卡片 + 最近更新）
├── about.md                   # 关于页
│
├── novel.html / gushi.html / guci.html / xiandaishi.html /
│   duilian.html / sanwen.html / overtones.html
│                            # 7 个板块的列表页（permala 一一对应）
│
├── _layouts/                  # 模板
│   ├── default.html           # 站点骨架（head + header + main + footer）
│   ├── work.html              # 通用详情页（小说/对联/散文/弦外共用）
│   ├── gushi.html             # 古诗详情页（带"诗"印章）
│   ├── ci.html                # 古词详情页（带"词"印章）
│   ├── poem.html              # 现代诗详情页（带"诗"印章）
│   ├── genre.html             # 通用列表页模板（小说/古词/散文/弦外复用）
│   ├── collection.html        # 现代诗合集详情页
│   ├── gushi-collection.html  # 古诗合集详情页
│   └── duilian-collection.html# 对联合集详情页
│
├── _includes/
│   └── prev-next.html         # 上一篇/下一篇 导航（核心 include）
│
├── _gushi/                    # 古诗作品（共 16 首）
├── _xiandaishi/               # 现代诗作品（共 14 首）
├── _guci/                     # 古词作品（共 17 阙）
├── _duilian/                  # 对联作品（共 13 副）
├── _sanwen/                   # 散文作品（共 1 篇 + 一个 files/ 目录——但 files 目录实际不存在，别被 README 误导）
├── _novel/                    # 小说作品（共 2 部）
├── _overtones/                # 弦外（歌词赏析，共 1 篇）
│
├── _gushi_collections/        # 古诗合集（2 本：《怀思集》《行路集》）
├── _xiandaishi_collections/   # 现代诗合集（《行路难》《科幻》）
├── _duilian_collections/      # 对联合集（《缀玉集》）
│
├── assets/
│   ├── css/style.css          # 1300+ 行的所有样式（必读）
│   └── novel/                 # 小说附件（PDF + 公众号排版 HTML）
│       ├── echo/
│       └── lighthouse/
│
├── echo_v9.pdf                # 根目录散落的小说 PDF（历史遗留，可以清理）
├── LighthouseProtocol.pdf     # 同上
├── echo_wechat.html           # 同上
├── novel_wechat.html          # 同上
│
├── _sass/                     # 空目录
├── works/                     # 空目录（**README.md 写错了这个路径，真实是 _<coll>/**）
│
├── README.md                  # ⚠️ 内容**过期**（见 §11）
├── .gitignore                 # 标准 Jekyll + token 忽略
└── HANDOFF.md                 # 本文档
```

### 几个容易误会的地方

- **README.md 写的是 `works/gushi/...`，但实际目录是 `_gushi/...`**。README 是初版写的，后来改过目录结构但 README 没更新。**以实际为准。**
- `_sanwen/files/` 这个路径在 README 里被提到过，但 `files/` 目录实际并不存在。**不要按 README 建它。**
- 根目录散落的 `echo_v9.pdf` / `LighthouseProtocol.pdf` / `echo_wechat.html` / `novel_wechat.html` 是历史遗留，**实际引用的是 `assets/novel/echo/` 和 `assets/novel/lighthouse/`**。根目录那 4 个文件是重复品，可以安全删。
- `_sass/` 和 `works/` 都是空目录。`_sass/` 是 Jekyll 默认找 SCSS 的地方，保留无害；`works/` 是 README 里的过时引用，可以删。

---

## 3. 核心配置 `_config.yml`

```yaml
title: 遐想之地
subtitle: ReverieLand
slogan: 不为无益之事，何以遣有涯之生
slogan_en: Do useless things. Dream out loud. Pass the time.
url: "https://zhengxionglee.github.io"
baseurl: "/ReverieLand"
permalink: /:collection/:year-:month-:day-:title/  # 标准 Jekyll 永久链接
markdown: kramdown
kramdown: { input: GFM, syntax_highlighter: rouge }  # GFM 必开，行末两空格 = 换行
```

### Collections（共 7 + 3 个合集）

| collection         | 输出目录             | 默认 layout        | 默认 genre |
| ------------------ | -------------------- | ------------------ | ---------- |
| `novel`            | `/novel/...`         | `work`             | 小说       |
| `gushi`            | `/gushi/...`         | `gushi`            | 古诗       |
| `gushi_collections`| `/gushi-collections/`| `gushi-collection` | 古诗集     |
| `guci`             | `/guci/...`          | `ci`               | 古词       |
| `xiandaishi`       | `/xiandaishi/...`    | `poem`             | 现代诗     |
| `xiandaishi_collections` | `/xiandaishi-collections/` | `collection` | 诗集     |
| `duilian`          | `/duilian/...`       | `work`             | 对联       |
| `duilian_collections` | `/duilian-collections/` | `duilian-collection` | 对联合集 |
| `sanwen`           | `/sanwen/...`        | `work`             | 散文       |
| `overtones`        | `/overtones/...`     | `work`             | 弦外       |

> **新增体裁 = 三件事**：
> 1. `_config.yml` 的 `collections` 加一段，`defaults` 也加一段
> 2. 根目录新建 `<板块>.html`（用 `layout: genre` 或自己写）
> 3. 仓库根建 `_新板块/` 目录
>
> 主页 `index.html` 的"栏目卡片"是手写的，**新加体裁记得去 index.html 同步加一张 genre-card**

---

## 4. 模板体系

### default.html
站点骨架。包含：
- 中文字体链：**Noto Serif SC + 霞鹜文楷（LXGW WenKai Screen Web Font）**，免费可商用
- 顶部导航（**改导航必须改这里**）：小说 / 古诗 / 古词 / 现代诗 / 对联 / 散文 / 弦外 / 关于
- 底部 footer + slogan

### 详情页家族
| layout         | 用于                   | 关键 class                   | 题图印章 |
| -------------- | ---------------------- | ---------------------------- | -------- |
| `work.html`    | 小说/对联/散文/弦外    | `.work-body`                 | 联/文    |
| `gushi.html`   | 古诗                   | `.gushi-body` 卷轴           | 诗       |
| `ci.html`      | 古词                   | `.ci-body` 卷轴              | 词       |
| `poem.html`    | 现代诗                 | `.poem-body` 卷轴            | 诗       |

所有详情页都会自动 `{% include prev-next.html %}`。

### 列表页家族
- `genre.html` 是**通用模板**（sanwen/guci/novel/overtones 用它）
- `gushi.html` / `xiandaishi.html` / `duilian.html` 是**手写的**（因为要展示合集卡片 + 平铺列表）

### 印章系统
每个体裁的标题旁有个旋转 -4° 的朱红小方块，里面一个字：
- 古诗 → "诗"
- 古词 → "词"
- 现代诗 → "诗"
- 对联 → "联"
- 小说/散文/弦外 → "文"

CSS 在 `.gushi-work .work-title::after`、`.ci-work .work-title::after`、`.poem-work .work-title::after`、`.work-duilian/novel/sanwen/overtones .work-title::after` 各自定义。

---

## 5. 写作模板

### 通用 front matter
```yaml
---
title: 标题
subtitle: 副标题（可省）
date: 2024-05-14 12:00:00 +0800
author: 你的名字           # 几乎所有现存作品都写"你的名字"——其实是占位符
genre: 古诗                # 大多时候 _config.yml defaults 会自动填
collection_title: 怀思集   # 关键！属于哪个合集（决定 prev-next 范围）
notes: 创作手记            # 可省，填了会在底部显示一个黄底色手记框
poem_count: 6              # 仅合集页用
duilian_count: 13          # 仅对联合集页用
poem_number: 1             # 诗集内编号（可省）
status: 已完结             # 可省
---
```

### 古诗（最简单）
```markdown
---
title: 独居怀旧
subtitle: 选自《怀思集》
date: 2024-05-14T12:00:00 +0800
author: 你的名字
collection_title: 怀思集
---

惘然瞻前梦，仙踪去复来。
心藏风月镜，魂断水花台。
冷眼常逢鬼，长情不染埃。
何时黄鹤去，何处白莲开？
```

- **不需要任何 HTML 标签**——`.gushi-body` 已经做了居中、自动分行的卷轴样式
- **段间用 `<hr>`**：因为 Kramdown GFM 的 `---` 在 GFM 模式里**不**产生水平线，而是被吃掉了或者产生空段落（见 §9）
- **押韵不押韵、对仗不对仗是站长的事，AI 不用管**——但写完帮忙检查错别字

### 古词
```markdown
---
title: 望江南
subtitle: 酒局
date: 2023-03-11T12:00:00 +0800
author: 你的名字
---

拼一醉，席上且频斟。
染尽尘沙风肃穆，
仰天大笑月昏沉。
终是酒国人。
```

上下阕之间用 `<hr>`。

### 现代诗
```markdown
故乡里父辈的事业欣欣向荣，

儿时的朋友带着孩子散步，

他说她在我们的学校教书。

<hr>

又做了一个梦回到高中，

重温了一遍背诵过的诗歌。
```

- 段内每行末尾**两空格**（GFM 强制换行）—— 这是 GFM 模式的标准做法
- 段间用 `<hr>`，CSS 已经把它做成 30% 宽、淡雅的朱红细线
- **不要用 `---`**（GFM 里它不产生 hr，见 §9）

### 对联
```markdown
---
title: 缀玉 其一
subtitle: 选自《缀玉集》
date: 2015-05-28T12:00:00 +0800
author: 你的名字
collection_title: 缀玉集
---

<div class="duilian-shanglian">一曲离殇，几度凝望，如镜的湖面不再流淌。</div>
<div class="duilian-xialian">情倦魂伤，心驰神往，依旧的歌声还在回荡。</div>
```

- 上下联分别用 `<div class="duilian-shanglian">` / `<div class="duilian-xialian">` 包
- 如果有横批（甚至不严格的"结句"），用 `<div class="duilian-horizontal">`，会自动加 "横批：" 前缀
- **副标题"选自《XX》"是占位文本**——README 里这么写，模板里也都这么写。但实际是来自 front matter 的 `collection_title`，副标题文字可以自由发挥

### 散文
```markdown
有天晚上十二点上床，第二天五点不到就醒了，醒来洗漱一番前往工位，
搞搞原创研究，意义不明，但目的明确，一切为了完成毕业任务。
不知道真理到底在哪里，把故事讲好比什么都重要。

今年三次出差的报销单很久都没有报下来，由于各种各样的原因被打回来。
```

- **直接写段落**，段落间空一行，Kramdown 自动转 `<p>`
- 散文里嵌入歌词用 `<div class="lyrics-sm" markdown="1">...</div>`（见下）

### 弦外（歌词赏析）
```markdown
---
title: 前程似锦
subtitle: 许嵩第九张专辑第三首 · 歌词赏析
date: 2026-06-26 10:49:00 +0800
author: 你的名字
genre: 弦外
status: 已发布
---

许嵩第九张专辑中第三首《前程似锦》。

---

## 原歌词

<div class="lyrics" markdown="1">

天色已微亮 清冷的广场  
座椅有点脏 将就躺躺  
...

</div>

---

## 续写版本

> 很不错的续写版本：

<div class="lyrics" markdown="1">

晨风吹拂 拂去离别的哑  
...
</div>

**听一版**：[SUNO V5.5] 《前程似锦》... [↗ B站](https://...)
```

- 歌词主体用 `<div class="lyrics" markdown="1">` 包
- `markdown="1"` 是关键——告诉 Kramdown 把 div 里的内容当 Markdown 渲染
- 歌词行末**两空格** GFM 换行
- 段落间用 `---` + 空行（Kramdown 标准 hr），不要用 `<hr>`，因为这里有 `## 二级标题`，风格跟现代诗不同
- 链接、B 站外链等用普通 Markdown 即可

### 小说
```markdown
---
title: 回声纪元
subtitle: 长篇科幻 · 已完结
date: 2026-06-10 20:00:00 +0800
author: 林深
status: 已完结
genre: 小说
---

## 简介

2198 年的深冬，神经工程师林深在第七码头旁那栋无窗的大楼里工作。

> 「其实第一次校准就已经足够精准……」

**下载：**

- 📄 [完整版 PDF]({{ "/assets/novel/echo/echo_v9.pdf" | relative_url }})
- 📱 [公众号排版版 HTML]({{ "/assets/novel/echo/wechat.html" | relative_url }})

**创作手记**：...

---

## 正文

## 一、再见

2198年的深冬，雪像从天幕深处漏下来的灰烬。
...
```

- **PDF/HTML 附件必须放在 `assets/novel/<作品简称>/` 下**，引用用 `relative_url` Liquid 过滤器
- 注意"简介"标题用 `## 二级标题`（不要用 `#`）
- "## 正文" 后再起 `## 一、再见` 这种章名

### 合集页（gushi_collections / xiandaishi_collections / duilian_collections）
```yaml
---
title: 怀思集
subtitle: 念故人，思旧事
date: 2016-08-08T12:00:00 +0800
author: 你的名字
poem_count: 6
permalink: /gushi-collections/huai-si-ji/   # 重要！指定永久链接
---
```

正文 markdown 自由发挥（一般用 `## 简介` 开头，写一段散文式介绍）。

---

## 6. CSS 速查（`assets/css/style.css`）

### 颜色变量（改风格只动这 9 个）
```css
:root {
  --bg: #f8f5ee;          /* 页面底色（宣纸） */
  --bg-card: #fffdf7;     /* 卡片/卷轴底色 */
  --ink: #2a2620;         /* 主文字 */
  --ink-soft: #5b5347;    /* 副文字 */
  --ink-faint: #8a8275;   /* 弱化文字（日期、meta） */
  --accent: #8b3a2f;      /* 朱红（链接、印章、强调） */
  --accent-soft: #c8978a; /* 淡朱红（hover、装饰） */
  --line: #e5ddc9;        /* 浅米色分隔线 */
  --highlight: #f5e9c8;   /* 创作手记底色 */
}
```

### 关键 class 速查

| class                    | 作用                                              |
| ------------------------ | ------------------------------------------------- |
| `.work-body`             | 所有详情页正文容器（1.15rem，line-height 2.3）    |
| `.work-body p`           | 默认首行缩进 2em、margin-bottom 1.5rem            |
| `.work-body .poem`       | 居中、无缩进、行高 2.4                            |
| `.work-body .lyrics`     | 歌词：居中、1.2rem、行高 2.2、字距 0.1em          |
| `.work-body .lyrics-sm`  | 歌词（小字号，散文里嵌入用）：0.95rem、不抢戏     |
| `.work-body .ci-title`   | 词题（小灰字）                                    |
| `.work-body .duilian-shanglian/xialian` | 上下联（1.4rem，居中）             |
| `.work-body .duilian-horizontal`        | 横批（自动加"横批："前缀）         |
| `.work-body .download-list`             | 小说下载链接列表                  |
| `.work-body .status-badge`              | 状态徽章（默认朱红，.done 是绿色） |
| `.work-body hr`          | 段间分隔线（在诗/词页里被改造成 30% 宽朱红细线）  |
| `.prev-next`             | 上一篇/下一篇 导航（响应式：<640px 单列）         |
| `.work-notes`            | 创作手记黄底框                                    |
| `.collection-intro`      | 合集页简介段（首行缩进 + 居中）                   |
| `.poem-list-flat`        | 列表页平铺行（古诗/现代诗/对联列表用）            |
| `.work-header`           | 详情页头部（居中 + 下边虚线）                     |
| `.work-nav`              | 详情页底部"返回XX"链接                            |

### 印章
- 朱红 `#8b3a2f` 背景，1.7rem × 1.7rem，旋转 -4°，右上角 -2.4rem
- 每个体裁一个字（诗/词/联/文）
- 修改/新增印章：编辑对应 layout（如 `.gushi-work .work-title::after { content: '诗' }`）

### 响应式断点
- `@media (max-width: 640px)`：导航居中、字体缩小、prev-next 单列

---

## 7. 重要决策与坑（接手的必读）

### 7.1 上一篇 / 下一篇 逻辑（commit `072fd95`）

**当前逻辑**（按用户的明确要求调换过一次）：
- 列表页（`gushi.html` 等）按 `date desc` 排序：**A2025 / B2023 / C2022 / D2016**
- 详情页的 **"上一篇"**（页面上显示 ← 上一篇）= 列表中**下一项** = 时间上**更旧**的
- 详情页的 **"下一篇"**（页面上显示 下一篇 →）= 列表中**上一项** = 时间上**更新的**

举例（打开 B2023 详情页）：
- 上一篇 ← C2022（旧）
- 下一篇 → A2025（新）
- 打开 D2016 详情页时，**上一篇环形回到 A2025**（循环到最旧），**下一篇 → C2022**

**关键代码在 `_includes/prev-next.html`（32-33 行）**：
```liquid
{%- assign prev_idx = idx | plus: 1 | modulo: siblings.size -%}
{%- assign next_idx = idx | minus: 1 | modulo: siblings.size -%}
```

> **如果用户再想换回"上一篇=更新的"语义**：把那两行的 `+1` 和 `-1` 互换即可（变成更直观的"上下"关系）。
>
> **重要细节**：
> - prev-next 是通过 `page.path` 推断 collection 名（**不是用 `page.collection.label`，那个在 include 里是空的**——是踩过坑的，见 commit `d840fb1`）
> - 如果有 `collection_title`，则**在同合集内循环**；没有则在整板块循环
> - 集合只有 1 篇时不显示（`siblings.size > 1`）

### 7.2 Kramdown GFM 的双空行行为（已记入我的 memory）

Kramdown GFM 模式下：
- 一个空行 = 段落分隔
- **连续两个空行不会产生更宽的间距**，只会"保留"前一段的"段落内有空行"的 markdown 习惯
- `---` 在 GFM 模式里**不**产生水平线（GFM 把它当成 setext 标题的下划线或单纯吃掉）

→ **诗/词/现代诗里的段间分隔，**统一用 HTML 标签 `<hr>`**，不用 markdown `---`**
→ CSS 里有专门处理 `.gushi-body hr` / `.ci-body hr` / `.poem-body hr`，把它们改造成朱红细线

**踩坑历史**（commit `f39aa9a` `25c5574` `8089959` 等）：早期用 `---`，在 GFM 下没产生 hr，我改了好几次才确认要换成 `<hr>` 标签。

### 7.3 Windows PowerShell + UTF-8

> 这是我自己 mavis agent 反复踩过的坑，不是项目本身的 bug

- PowerShell 5.1 默认 ANSI 解码（GBK），cat/ls 中文文件名会乱码
- 解决方案：读文件用 Read 工具（直接 UTF-8）；写文件用 Write/Edit 工具
- **不要用 `Get-Content | Set-Content` 改文件**——会破坏中文编码
- 跑 git 命令前可以先 `chcp 65001` 切换到 UTF-8 代码页

### 7.4 GFM 行末两空格

每行末尾**两空格** = 强制换行（`<br>`）—— **不要**用 `\\` 或 `  \n`（html 里那是个空格加换行符，不是真换行）
**整篇诗/词/歌词基本都是这个写法**，保持一致。

### 7.5 列表页与详情页"上一篇"的视觉连续性

**当前状态下**（commit `072fd95` 之后）：
- 列表页（倒序）里 A2025 是**最上面**，从 B2023 详情页点"上一篇"会跳到 C2022（更旧的）—— **不是从列表的"上一行"继续**

**这是个设计取舍**（用户明确要求的），AI 不用去主动改。
**如果用户问"为什么列表里 B 的上一项是 A，但点 B 详情页的'上一篇'却到 C 了？"**——就答：按你的要求，"上一篇"是**时间上更旧的**，"下一篇"才是时间上更新的。

### 7.6 GitHub Pages 缓存

**强刷一定要 `Ctrl + F5`**（或 `Cmd + Shift + R`）。GitHub Pages 的浏览器缓存有时候很顽固，单纯刷新看不到新内容。

### 7.7 README.md 内容过期

**重要**：README.md 里有几处是错的：
1. 写的是 `works/gushi/...`，实际是 `_gushi/...`
2. 没提到 `弦外（overtones）` 板块
3. 没提到 `gushi_collections` / `xiandaishi_collections` / `duilian_collections` 三种合集
4. "如何添加新作品" 里的 front matter 例子写的是 `subtitle: 一首七绝` 这种，**实际是 `subtitle: 选自《XX》` 更常见**
5. 提到的 `_sanwen/files/` 不存在

→ **改 README 是接手后第一个可以做的小事**（见 §12 待办）

---

## 8. 部署与协作流程

### 用户本地操作（PowerShell）
```powershell
cd D:\AI_Project\Novel_website
git add .
git commit -m "feat(古诗): 新增《XX》"
git push
```

### 部署链路
1. `git push` → GitHub
2. GitHub Pages 自动 build（Jekyll 静态生成）→ 1-2 分钟
3. 用户 `Ctrl + F5` 强刷即可看到

### 重要：没有 CI / build log
- 看不到 GitHub Actions 的 build log（Pages 自动 build 不会发邮件）
- 如果 push 后没生效，**先去 GitHub 仓库页 → Settings → Pages 看 Build 状态**
- 极少数情况 Jekyll build 会失败，错误信息藏在 build log 里——用户不一定会自己看，需要 AI 帮忙

### 协作风格
- **几乎所有 commit 都是 AI 做的**（git log 里 36 个 commit，提交信息都很规范：`feat(x): ...` / `fix(x): ...` / `chore:`）
- 用户的"工作"是：写内容（写诗、写散文）+ 提需求（"把歌词居中"、"换印章文字"）——**不要替用户改文字内容**（除非用户明确说"帮我润色"或"我这里写错了"）

### 唯一可改的"创作"边界
- 修错别字、修正 Markdown 格式（少空格、多空格、`<hr>` 位置）→ 主动修
- 改写诗的内容、添加内容、润色文笔 → **先问用户**（这是用户创作的核心）

---

## 9. 与原作者（用户）的协作偏好

> 这是我从多次交互里总结的"性格画像"，AI 协作时请务必遵守

### 性格
- **决策果断**：想清楚了就立刻做，不喜欢"反复来回"
- **审美讲究**：字体、间距、印章、颜色都很在意，每个细节都要"看起来对"
- **爱琢磨**：会反复调"上一篇/下一篇"的逻辑、调印章的角度、调留白多少
- **讨厌来回折腾**：经历过 `删 <hr> → 清理 BOM 破坏内容 → 回退` 的循环，对**批量改动**敏感
- **沟通简短**：一两句回复就够，不要长篇分析

### 工作风格
- **改动谨慎**：网站改任何东西都希望先**小范围验证**，不要一次性大改
- **批量改动前先试点**：宁可分 2~3 步走
- **不可逆操作先确认**：`git reset`、强制 push、删文件 → 必须先说
- **第三方服务（OAuth、API key）部署前先确认前置条件**

### AI 协作偏好
- **回复简短直接**——别啰嗦
- **直接给方案和结论**——别给一长串 "Pros & Cons" 让用户选
- **做完给个一句话总结**就行
- **截图/示例不需要**——直接说"Ctrl+F5 看效果"
- **可以提建议但不要 push**：不要主动建议"加 Giscus 评论"、"加搜索"——**用户之前折腾 Giscus 30 分钟配不上，立刻决定撤掉**。所以**不要主动推新功能**

### 已知的"不要再做的事"
- ~~批量改 front matter 字段名~~（之前踩过坑）
- ~~改 commit 历史（rebase / amend）~~（用户明确说不要）
- ~~加评论系统~~（明确拒绝了 Giscus）
- ~~加分析/统计~~（没问过，但符合"克制的工具"原则）
- ~~push 之前没确认就 push~~（要 commit 之前把 diff 给用户看）

---

## 10. 关键 commit 速查（git log 节选）

```
c67ecc8  feat(散文): 新增《减法》，含《皮下》歌词小字号嵌入
d11c6fa  feat(弦外): 歌词加 .lyrics 容器样式（居中+不缩进+紧凑）
072fd95  fix(prev-next): 调换上一篇/下一篇指向（旧↔新）   ← 重要
02d02c1  feat(弦外): 新增《前程似锦》许嵩歌词赏析 + 续写版本
1558c62  feat(印章): 对联/小说/散文/弦外 详情页加题图印章（联/文/文/文）
f79bcdb  古词 · 新建 17 首（2022-2024，词牌名+主题，仿现代诗卷轴+印章'词'）
ab4f28b  对联 · 隐藏'上联/下联'小标签（按用户要求）
4c40dc6  古诗 · 新建两个诗集《怀思集》(6首) +《行路集》(10首)
c73b612  现代诗 · 新增诗集《科幻》（5首，2021.10-2026.03）
8b9f545  小说 · 回声纪元替换为完整原文（按段分章），作者改林深；灯塔协议作者改林岬
8edfa10  style: 引入霞鹜文楷字体 (LXGW WenKai)
8ccdb15  rebrand: 拾墨斋 → 遐想之地 / ReverieLand
```

完整 36 个 commit 在 `git log --oneline`。

---

## 11. 快速上手 Checklist（接手后第一周建议做的事）

1. **[优先级 P0 · 安全]** Revoke 那个 git remote 里的明文 token（见 §1）
2. **[P1]** 读一遍 `_config.yml` + `assets/css/style.css` 顶部 200 行（颜色变量、关键 class）
3. **[P1]** 浏览每个 `_layouts/*.html` 和 `_includes/prev-next.html`，理解模板体系
4. **[P2]** 修 README.md（路径 `works/` → `_gushi/` 等、补充弦外板块、补合集说明）
5. **[P2]** 删根目录那 4 个散落的小说文件（`echo_v9.pdf` `echo_wechat.html` `LighthouseProtocol.pdf` `novel_wechat.html`）—— 用 `mavis-trash` 走回收站
6. **[P2]** 删空目录 `works/`（保留 `_sass/`，Jekyll 默认会找它）
7. **[P3]** 在本地起一个 Jekyll 服务跑一遍，确保改动不破构建：
   ```bash
   gem install jekyll bundler
   bundle install
   bundle exec jekyll serve
   # 浏览器打开 http://127.0.0.1:4000/ReverieLand/
   ```
8. **[P3]** 跑一遍 `git remote -v`，把带 token 的 URL 换成 SSH 或 `gh auth` 方式
9. **[P4]** 整理 git 提交历史（可选）—— 把"调试用 DBG"那类 commit squash 掉（**先问用户**）

---

## 12. 给下一个人（也是给我自己）的提醒

- **新加板块要同步改 4 个地方**：
  1. `_config.yml` 的 `collections` 和 `defaults`
  2. 根目录新建 `<板块>.html`（可能用 `layout: genre`）
  3. `_layouts/` 看是否需要新模板（如果跟现有 work/poem/ci 类似就复用，不需要新模板）
  4. `index.html` 的"栏目卡片"和 `default.html` 的导航

- **新加印章**：
  - 在对应 layout 的 CSS 里加 `.work-<新板块> .work-title::after { content: 'X' }`
  - 或者直接复用 `.work-novel .work-title::after` 的规则集（用"文"）

- **字体想换**：改 `default.html` 里的 `<link>` + `assets/css/style.css` 里的 `body` 字体栈。当前是 LXGW WenKai + Noto Serif SC，免费可商用

- **改主题色**：改 CSS 顶部 `:root` 的 9 个变量，三个就够（`--bg` `--ink` `--accent`）

- **不要轻易动的文件**：
  - `_includes/prev-next.html`（调换过语义，再调会绕回去）
  - `_config.yml` 的 permalink 格式（改了会让所有旧链接 404）
  - 现有作品的 front matter（`subtitle: 选自《XX》` 占位文本看着是占位，**别主动去 "规范化" 它们**——这是用户故意保留的简单格式）

- **常见误解**：
  - 看到 "subtitle: 选自《怀思集》" 别以为是冗余——它跟 front matter 里的 `collection_title` 是两回事，前者给人看、后者给 prev-next 找范围
  - `author: 你的名字` 是占位文本，**别主动替换成什么"你的真名"**——用户没告诉过
  - 看到一个 md 文件的 `subtitle` 是空的别补，**留空是合法的**

---

**完成日期**：2026-07-01
**最后 commit**：`c67ecc8`（散文《减法》+《皮下》歌词）
**在线预览**：https://zhengxionglee.github.io/ReverieLand/

—— Mavis
