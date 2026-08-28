# 静态站点 · 小耳再造整形科普系列

本目录是 89 篇内容（A 档 80 篇 + C 档 5 章 + B 档 1 篇 + 大纲 1 篇 + SOP 1 篇 + 排期表 1 篇）的静态展示站点。

## 文件结构

```
site/
├── index.html              首页
├── a-popular.html          A 档大众科普（20 篇主题，每篇 4 件衍生）
├── b-training.html         B 档业务培训入口
├── c-review.html           C 档专业综述入口
├── article.html            单篇查看器（按需加载对应分片）
├── build_index.py          构建生成器（修改源 md 后重跑）
├── sitemap.xml             SEO 站点地图
├── robots.txt              抓取规则
├── assets/
│   ├── style.css           样式
│   ├── articles-meta.js    轻量元数据（title/cat/prev/next/back，无正文，26 KB）
│   ├── global_index.js     全局搜索索引（text 缩短为 260 字符，66 KB）
│   ├── global_search.js    搜索交互
│   ├── reading_state.js    已读状态管理
│   ├── resume.js           首页"继续阅读"
│   ├── wechat_share.js     微信分享
│   ├── articles.js         （旧结构，含全量 html，兼容用）
│   ├── slots/              正文分片：A1..A20.js、B.js、C1..C5.js、misc.js
│   │                       article.html 按需加载对应分片，每片约 11-27 KB
│   └── img/
│       ├── logo.png        圆形 logo
│       ├── favicon.png     方形 favicon
│       ├── favicon.svg     SVG favicon
│       └── share-card.png  分享专属横图（1200×630，og:image / 微信分享用）
│   └── fonts/
│       └── NotoSansSC-Regular.otf  中文字体（生成分享图用，非页面加载资源）
└── README.md               本文件
```

## 本地浏览

### 方式 A：用 Python 启动一个临时服务器（推荐）

```bash
cd /home/sooogooo/dsh/tools/site
python3 -m http.server 8765 --bind 127.0.0.1
```

浏览器访问 http://127.0.0.1:8765/

### 方式 B：直接用 file:// 打开

```bash
xdg-open /home/sooogooo/dsh/tools/site/index.html  # Linux
open /home/sooogooo/dsh/tools/site/index.html      # macOS
```

可以工作，但 article.html 在 file:// 下也能正常加载（数据已预渲染到 articles.js）。

## 修改源 md 后

A/B/C 档内容存放在 `/home/sooogooo/dsh/tools/` 顶层。修改任何 .md 后，重跑索引生成器：

```bash
python3 /home/sooogooo/dsh/tools/site/build_index.py
```

该脚本会：

1. 扫描顶层 89 个 md 文件
2. 在 Python 端做一遍 Markdown → HTML 预渲染
3. 生成 `assets/articles-meta.js`（轻量元数据：title/cat/prev/next/back）
4. 生成 `assets/slots/*.js`（正文按 A/B/C 档分片，article.html 按需加载对应分片，大幅减少文章页首屏 JS）
5. 生成 `assets/global_index.js`（搜索索引，text 缩短为 260 字符）
6. 生成 `assets/articles.js`（旧结构兼容，含全量 html）
7. 生成 `sitemap.xml`、`robots.txt`、`assets/meta.json`（SEO 与抓取）

> 注意：`sitemap.xml` / `robots.txt` 已用部署域名 `https://microtia.amcd.westhospital.net` 生成。若日后域名变更，请改 `build_index.py` 顶部 `SITE_BASE` 并重跑。

## 设计要点

- **冷色调医疗克制**：主色 `#0a6cb8`（深蓝），辅色 `#d4a437`（金），与 logo 配色一致。
- **移动优先**：480px / 720px / 980px 三档断点，所有交互在 540px 以下做了触摸友好优化。
- **可访问性**：skip link、语义标签、焦点可见、WCAG AA 对比度、暗色模式（系统偏好）、减弱动效（系统偏好）。
- **零依赖**：无打包器、无外部 JS 库；正文分片 + 轻量 meta，可 file:// 直开。
- **性能**：文章页按需加载单一正文分片（约 11-27 KB），首屏 JS 从约 730 KB 降到约 60 KB。
- **签名**：每页底部"2026 | 内部资料 · 仅供机构内部使用" + "重庆西区医院医学美容中心"。
- **签名**：每页底部"2026 | 内部资料 · 仅供机构内部使用" + "重庆西区医院医学美容中心"。

## 合规

站内文字沿用既有内容的合规红线（不承诺无痕、零风险、一次永久、保证两侧对称；不替患者选材料；不制造身体羞耻）。HTML 与 CSS 文件本身已通过禁用词扫描，无"一刀""落盘""结果""是不是"等违规表达，无中文破折号。
## 微信公众号分享

### 默认行为（无需后端，立即生效）

所有页面 `<head>` 都带了 Open Graph 标签，`og:image` 指向 logo（`assets/img/logo.png`）。因此无论用户在微信里把链接发到聊天还是朋友圈，微信抓取网页时都会自动带出：

- 标题（og:title）
- 简介（og:description）
- 小图标（og:image → logo）

这层不需要任何后端，部署后即生效。对"向聊天和朋友圈分享时带图、logo 做默认小图标"的需求已满足。

### 可选增强（微信内打开时精确控制卡片）

若想精确控制分享卡片的标题、描述、图与链接，需接入微信 JS-SDK 签名。前端脚本 `assets/wechat_share.js` 已就绪，只在微信内置浏览器里加载 SDK，并调用：

- `wx.updateAppMessageShareData`（发送给朋友）
- `wx.updateTimelineShareData`（分享到朋友圈）

后端需在页面渲染时注入签名（**必须由后端以当前页面 URL 计算 jsapi_ticket 签名**，前端无法算出有效签名）：

```html
<script>
  // 示例结构，实际值由后端生成
  window.WECHAT_SIGN = {
    appId: "wx1234567890abcdef",
    timestamp: 1680000000,
    nonceStr: "abcdef123",
    signature: "后端生成的签名"
  };
</script>
```

脚本检测到 `window.WECHAT_SIGN` 即自动初始化；没有签名时静默降级到 og:image 兜底，不报错。

可覆盖项（可选）：

- `window.WECHAT_SHARE_IMG`：自定义分享小图（默认 logo）。
- 文章页（article.html）打开时会自动同步 og:title/description/url，并调用 `window.AMCWechatShare.refresh()` 让 JS-SDK 读取最新值。

### 说明

微信分享签名依赖后端（ticket），本设计的前端部分已完成全部接入与优雅降级。要真正在微信内精确控制分享卡片，请后端按官方文档生成签名并注入 `window.WECHAT_SIGN`。
