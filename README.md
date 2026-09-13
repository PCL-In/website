# PCL-In 下载站

PCL-In 官方下载站的源码仓库，通过 GitHub Pages 部署在 <https://pclin.astras.cc>。

启动器本体在另一个仓库：[PCL-In/desktop](https://github.com/PCL-In/desktop)。

## 目录说明

- `index.html` — 主页面（版本面板 / 下载 / 上游对比 / 功能 / 系统要求 / FAQ）
- `assets/css/style.css` — 样式
- `assets/js/app.js` — 从启动器仓库的 GitHub Releases 自动拉取版本信息、校验值与引导逻辑
- `assets/img/` — Logo 与站点图标（取自启动器自身的 `Plain Craft Launcher 2/Images/icon.png`）
- `CNAME` — 自定义域名 `pclin.astras.cc`
- `.nojekyll` — 禁用 Jekyll 处理，避免特定目录 / 文件名被干扰

## 界面与主题

站点采用 **Soft UI（新拟态）** 风格：用柔和的凸起 / 凹陷阴影表达层级，配色克制
（主色 `#4f6ef7`），不使用渐变文字、发光装饰与 emoji。图标全部取自
[Lucide](https://lucide.dev)（ISC 许可），以**内联 SVG sprite** 形式写在 `index.html` 里，
运行时不请求任何外部资源。

站点 Logo 与 favicon 取自启动器的 `Plain Craft Launcher 2/Images/icon.png`，
裁成圆角透明 PNG 后存放于 `assets/img/`：`logo.png`（256×256，顶栏 / 页脚 / 高清 favicon）、
`favicon-32.png`、`apple-touch-icon.png`。

提供 **明亮 / 暗色** 两套主题：默认跟随系统 `prefers-color-scheme`，点击右上角按钮可手动切换，
选择记录在 `localStorage` 的 `pclinTheme`（在 `<head>` 的内联脚本里应用，避免首屏闪烁）。

## 两种下载模式

页面提供**普通模式**与**友好模式**；**默认进入友好模式**，切换状态记录在
`localStorage` 的 `pclinDownloadMode`。

- **普通模式**：以表格列出全部下载文件（x64 / ARM64，含 GPG 签名与 SHA256）。
- **友好模式**：先选择设备类型（Windows x64 / ARM64）卡片，再在弹出的配置对话框中
  依次选择 **版本 → 处理器架构 → 安装包类型**，实时显示**匹配文件**并可直接下载或查看 SHA256。
  页面会通过 User-Agent / `navigator.userAgentData` 识别当前设备，给匹配的卡片打上
  「适合此设备」标记；移动端设备则标记为「不适合此设备」。

## 下载加速（可选）

每个「下载」按钮旁边都有一个「代理」小按钮：点击后所有下载链接（普通模式的 `.exe` 与 `.asc`，
以及友好模式向导里的下载按钮）都会加上 `https://ghproxy.net/` 前缀，通过第三方代理转发 GitHub 直链，
适合直连较慢的情况。开启时按钮呈凹陷高亮态。

页面上的所有「代理」按钮共享同一个状态（普通模式两行 + 向导里的匹配文件），
记录在 `localStorage` 的 `pclinUseProxy`。代理只改变下载链路的走向，文件内容不变，
**SHA256 校验值仍然适用**，开启后页面上方也会给出对应提示。

## 数据来源

下载链接、版本号、SHA256、文件大小均自动从启动器仓库的 GitHub Releases 获取
（读取 `https://api.github.com/repos/PCL-In/desktop/releases`，友好模式的版本下拉框会列出最近 20 个版本）。
当无法联网或接口不可用时，页面会回退到内置的当前版本信息，因此地址始终可用。

## 部署

`.github/workflows/pages.yml` 在每次推送到 `main` 时，把仓库根目录（排除 `.git` / `.github` / `_site`）
发布为 GitHub Pages 站点，并在部署前后清理历史的 `github-pages` artifact（避免「multiple artifacts」报错）。

- 仓库 **Settings → Pages** 中 Source 需为 **GitHub Actions**；
- 自定义域名 `pclin.astras.cc` 已写入 `CNAME`；
- DNS 侧把该域名指向 GitHub Pages（本站经 Cloudflare 代理）。

## 更新版本

启动器仓库发布新的 GitHub Release 后，只要包含 `PCL-In-x64.exe` / `PCL-In-arm64.exe`
（及其 `.asc` / `.sha256`），本站的普通模式与友好模式都会自动展示新版本，无需改代码。

唯一需要手动跟的是**离线回退数据**：`assets/js/app.js` 顶部的 `STATIC`（版本号、日期、
两个安装包的 size 与 sha256）以及 `index.html` 里 `vX.Y.Z` 形式的兜底链接，
它们只在浏览器无法访问 GitHub API 时才会用到；不改不影响正常访问，但离线打开时页面会显示旧版本号。

## SEO

站内能自动做的部分都已经自动，每次推送到 `main` 都会生效：

| 项目 | 位置 | 说明 |
| --- | --- | --- |
| `robots.txt` | 站点根目录 | 全站可抓取，末尾给出 sitemap 地址 |
| `sitemap.xml` | 站点根目录 | 目前只有一个 URL；**新增页面时要手动加进去** |
| canonical / OG / Twitter | `index.html` `<head>` | 统一指向 `https://pclin.astras.cc/`，带分享大图 |
| 结构化数据（JSON-LD） | `index.html` `<head>` | `WebSite` + `Organization` + `SoftwareApplication` + `FAQPage`；软件版本由 `app.js` 取到 Release 后同步 |
| IndexNow | `.github/workflows/pages.yml` + 根目录的 `<key>.txt` | 部署完成后自动把首页提交给 IndexNow，Bing / Yandex 几分钟内来抓；失败只打日志，不影响部署 |
| `404.html` | 站点根目录 | 自定义 404，带 `noindex, follow` |

### 只有站长能做的一次性操作

1. **Bing 网站管理员工具**（对 Bing 收录影响最大）：<https://www.bing.com/webmasters>
   添加 `https://pclin.astras.cc/`，用「HTML meta 标记」验证——把后台给的验证码填进
   `index.html` 里 `msvalidate.01` 那一行并取消注释，然后提交一次 `sitemap.xml`。
   之后 IndexNow 的收录情况也能在该后台看到。
2. **Google Search Console**（可选）：<https://search.google.com/search-console>
   同样添加站点并提交 `https://pclin.astras.cc/sitemap.xml`。
   Google 不支持 IndexNow，只能靠 sitemap + 抓取。
3. **Cloudflare 不要拦搜索引擎**：确认 `Security → Bots` 没有开启会拦截已验证爬虫的模式
   （Bot Fight Mode 会挡 Bingbot / Googlebot）。本站经 Cloudflare 代理，这一步容易被忽略。
