# 遐想之地 · ReverieLand

一个用 Jekyll + GitHub Pages 搭建的免费个人网站。收录小说、古诗、古词、现代诗、对联、散文、技术遐想、弦外之音。

> 不为无益之事，何以遣有涯之生。

## 特点

- **完全免费**：托管在 GitHub Pages，无服务器费用
- **国内访问快**：配合 Cloudflare CDN（可选）后访问速度不错
- **专注写作**：用 Markdown 写就行，不用碰代码
- **中式美学**：衬线字体、宣纸色调，看起来有书卷气

## 目录结构

```
.
├── _config.yml          # 站点配置
├── index.html           # 主页
├── about.md             # 关于页
├── novel.html           # 小说列表页
├── gushi.html           # 古诗列表页
├── guci.html            # 古词列表页
├── xiandaishi.html      # 现代诗列表页
├── duilian.html         # 对联列表页
├── sanwen.html          # 散文列表页
├── _layouts/            # 页面模板
│   ├── default.html
│   ├── work.html
│   └── genre.html
├── assets/css/
│   └── style.css        # 样式
└── works/               # 作品存放目录
    ├── novel/           # 小说
    ├── gushi/           # 古诗
    ├── guci/            # 古词
    ├── xiandaishi/      # 现代诗
    ├── duilian/         # 对联
    └── sanwen/          # 散文
```

## 本地预览（可选）

如果想在本地先看效果：

```bash
# 1. 安装 Ruby（https://rubyinstaller.org/ 上有 Windows 安装包）
# 2. 安装 Jekyll
gem install jekyll bundler

# 3. 在项目目录下启动本地服务器
cd D:\AI_Project\Novel_website
bundle install   # 第一次需要
bundle exec jekyll serve

# 4. 浏览器打开 http://127.0.0.1:4000
```

## 部署到 GitHub Pages（5 步搞定）

### 第一步：在 GitHub 创建仓库

1. 注册/登录 [github.com](https://github.com)
2. 右上角 `+` → `New repository`
3. 仓库名填：`你的用户名.github.io`（比如 `zhangsan.github.io`）—— 这是用户主页
   - 或者随便起个名也行，比如 `novel-website`，那就是项目主页（访问地址会变成 `用户名.github.io/novel-website`）
4. 选 `Public`，点 Create

### 第二步：上传代码

如果你装了 Git：

```bash
cd D:\AI_Project\Novel_website
git init
git add .
git commit -m "init: 遐想之地开张"
git branch -M main
git remote add origin https://github.com/你的用户名/仓库名.git
git push -u origin main
```

如果没装 Git，最简单的方式：
1. 在 GitHub 仓库页点 `uploading an existing file` 链接
2. 把 `D:\AI_Project\Novel_website` 里的所有文件和文件夹**直接拖**到网页里
3. 点 Commit changes

### 第三步：开启 Pages

1. 进仓库页面，点上面的 `Settings` 标签
2. 左边菜单找 `Pages`
3. Source 选 `Deploy from a branch`，Branch 选 `main` / `(root)`
4. 等 1-2 分钟，刷新页面会显示一个绿色的网址：`https://你的用户名.github.io`

### 第四步：清掉示例作品

示例作品 `works/` 下的诗词、对联、散文是我临时写的演示，删掉就行：

```
works/gushi/2026-05-12-shan-zhong-ou-de.md
works/guci/2026-04-18-qing-yu-an-ye-gui.md
works/xiandaishi/2026-02-04-li-chun-ri.md
works/duilian/2026-01-15-shu-fang-lian.md
works/sanwen/2026-03-22-lao-jie-de-xia-wu.md
```

散文「开篇散文」传统：如果想留一篇站名记之类的散文，可以新建 `_sanwen/<日期>-<标题>.md` 写一下。

**小说保留**——`works/novel/hui-sheng-ji-yuan.md` 和 `works/novel/deng-ta-xie-yi.md` 是你的真实作品，连同 `works/novel/files/` 下的原文 PDF/HTML 一起保留。这些文件会被 Jekyll 自动复制到网站上，访客可以从介绍页下载。

### 第五步：开始写你自己的作品

参考下面的「如何添加新作品」。

## 如何添加新作品

以古诗为例：

1. 在 `works/gushi/` 下新建文件，文件名用日期开头：`2026-07-01-jiang-nan-chun.md`
2. 文件内容：

```markdown
---
title: 江南春
subtitle: 一首七绝
date: 2026-07-01 10:00:00 +0800
author: 你的名字
notes: 这首是去年在杭州西湖边写的，最近才定稿。
---

<div class="poem" markdown="1">

千里莺啼绿映红，
水村山郭酒旗风。
南朝四百八十寺，
多少楼台烟雨中。

</div>
```

3. 提交到 GitHub：

```bash
git add works/gushi/2026-07-01-jiang-nan-chun.md
git commit -m "新增：江南春"
git push
```

4. 等 1 分钟刷新网站就有了。

## 其他体裁的模板

### 小说（多段落）

```markdown
---
title: 故事标题
date: 2026-07-01 10:00:00 +0800
author: 你的名字
---

## 第一章

正文……

## 第二章

正文……
```

### 古词

```markdown
---
title: 浣溪沙·秋思
date: 2026-07-01 10:00:00 +0800
---

<p class="ci-title">浣溪沙 · 秋思</p>

<div class="poem" markdown="1">

山色青青雨亦奇，
一帘秋意入书帷。
灯前独坐写新词。

</div>
```

### 对联

```markdown
---
title: 春联
date: 2026-02-09 10:00:00 +0800
---

<p class="duilian-shanglian">爆竹声中除旧岁</p>
<p class="duilian-xialian">梅花香里报新春</p>
<p class="duilian-horizontal">万象更新</p>
```

### 现代诗 / 散文

直接用 Markdown 写就行，段落之间空一行。

## 自定义

- **改站名**：编辑 `_config.yml` 里的 `title`、`subtitle`
- **改样式**：编辑 `assets/css/style.css`，顶部有配色变量，改 `--bg` `--ink` `--accent` 三个就能换风格
- **加新体裁**：参考 `_config.yml` 里的 `collections` 配置，加一个目录 + 一个列表页就行

## 国内访问优化（可选）

GitHub Pages 在国内访问偶尔会慢。简单办法：用 [Cloudflare](https://www.cloudflare.com/) 给自定义域名做 CDN 加速，或者直接用 jsDelivr + Gitee Pages 镜像。

最简单的方案：注册一个自己的域名（比如 `shimozhai.com`，几十块一年），绑到 Cloudflare，访问速度就很稳了。等你玩熟了再说，先把内容写起来最重要。

---

有问题随时来问我～ 写作愉快！