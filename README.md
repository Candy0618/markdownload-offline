# MarkDownload Offline

谷歌商店的插件[MarkSnip - Markdown Web Clipper](https://chromewebstore.google.com/detail/marksnip-markdown-web-cli/kcbaglhfgbkjdnpeokaamjjkddempipm?hl=zh-CN&utm_source=ext_sidebar)能满足该fork的需求，本仓库不再更新。



这是一个基于 [deathau/markdownload](https://github.com/deathau/markdownload) 的中文 fork，用于将网页内容裁剪并保存为 Markdown 文档，并尽量把图片一并离线化保存。项目目标不是做网页快照，而是提取可读正文后导出为适合本地笔记、归档和二次编辑的 Markdown。

当前仓库代码版本对应扩展清单中的 `3.4.1`，在原版 `3.4.0` 基础上额外补充了一些离线图片相关修复，重点包括：
- 为图片预下载请求补充 `Referer` / `Origin`，改善部分防盗链站点的图片下载成功率。
- 在预下载阶段检查 HTTP 状态码和返回的 MIME，避免把 HTML/XML/JSON 错误页误存成图片文件。
- 在改写 Markdown 中的图片地址时跳过 fenced code block，避免误改正文中的代码示例。

项目依然不能保证适配所有网站。页面结构极端复杂、内容懒加载严重、站点有严格反爬或防盗链策略时，导出结果仍可能需要人工检查。

## 功能概览

- 提取网页正文并转换为 Markdown。
- 支持只裁剪当前选择文本。
- 弹出窗口中可预览并小幅编辑导出的 Markdown。
- 支持直接下载 `.md` 文件。
- 支持将图片与 Markdown 一起保存，用于离线阅读。
- 支持将图片嵌入为 Base64 Data URL。
- 支持 Obsidian 风格图片链接与 Obsidian 集成。
- 支持 frontmatter / backmatter 模板。
- 支持模板变量、日期格式化、自定义标题和下载路径。
- 支持上下文菜单和键盘快捷键。

## 工作方式

扩展的大致流程如下：

1. 读取当前页面 DOM。
2. 使用 `Readability.js` 提取主要正文。
3. 使用 `Turndown` 将简化后的 HTML 转换为 Markdown。
4. 根据设置对标题、模板、图片、链接和下载路径做进一步处理。
5. 如果启用了图片下载，则先预下载图片，再把 Markdown 中对应的图片引用改写为本地路径或 Base64。

这意味着导出的不是原始网页 HTML，而是“提取后的正文 Markdown”。如果你的目标是纯归档网页原貌，这个项目并不等价于浏览器网页存档或单文件快照工具。

## 安装

这个 fork 仓库目前主要面向本地构建和加载。你可以按下面的方式使用：

### Firefox

1. 安装依赖。
2. 在 `src/` 目录执行 `npm run start:firefoxdeveloper`，或者使用 `web-ext run` 加载。
3. 也可以在 Firefox 的调试扩展页面临时加载 `src/manifest.json`。

### Chromium / Chrome / Edge

1. 打开浏览器的扩展管理页。
2. 启用“开发者模式”。
3. 选择“加载未打包的扩展程序”。
4. 选择 `src/` 目录。

### Safari

仓库中保留了 Safari Web Extension 包装工程，位于 `xcode/MarkDownload - Markdown Web Clipper/`。如果你需要打包 Safari 版本，需要使用 Xcode 打开并处理该工程。

## 使用方法

最常见的使用方式：

1. 打开你要保存的网页。
2. 点击扩展图标。
3. 在弹出的编辑器里检查生成的 Markdown。
4. 需要时手动修改标题或正文。
5. 点击下载按钮保存为 `.md` 文件。

如果页面上先选中了一段文字，扩展可以只导出选中部分。

## 离线图片说明

这个 fork 的重点是“尽量把文章和图片一起离线保存”，但这里有几个边界需要明确：

- 启用 `downloadImages` 且下载模式为 `downloadsApi` 时，扩展会先请求图片，再决定最终写入 Markdown 的本地图片路径。
- 对部分有防盗链的网站，扩展现在会在图片请求里注入来源页的 `Referer` / `Origin`。
- 如果服务端返回的不是图片，而是 HTML、XML、JSON 或文本错误页，扩展会跳过该图片，避免生成伪图片文件。
- fenced code block 中看起来像图片链接的文本不会被改写，这样可以保留文章中的代码示例原貌。

需要注意的是：

- 并不是所有站点的图片都能成功离线化，尤其是登录态资源、短时签名 URL、跨域受限资源或强校验防盗链资源。
- 即使 Markdown 已生成，图片预下载失败时，最终文档里仍可能出现本地图片路径但对应文件不存在的情况。这个问题在当前代码中还没有完全兜底。
- 如果你的首要目标是“百分之百可还原的网页离线档案”，建议把这个工具和网页快照类工具区分开来。

## 主要选项

常用配置包括：

- `includeTemplate`：是否启用 frontmatter / backmatter 模板。
- `downloadImages`：是否下载图片。
- `imageStyle`：图片写入 Markdown 的方式，可选普通 Markdown、Obsidian 风格、Base64、不包含图片等。
- `imagePrefix`：图片保存路径前缀。
- `title`：导出文件名模板。
- `mdClipsFolder`：Markdown 下载目录。
- `saveAs`：下载时是否始终弹出另存为。
- `contextMenus`：是否启用右键菜单。
- `turndownEscape`：是否保留 Turndown 默认转义行为。

默认情况下，图片下载模式使用 `downloadsApi`。在不支持下载 API 的环境下，代码会回退到 `contentLink`。

## 模板变量

标题、frontmatter、backmatter、图片目录等字段支持模板变量替换。常见变量包括：

- `{pageTitle}`
- `{title}`
- `{baseURI}`
- `{origin}`
- `{host}`
- `{hostname}`
- `{port}`
- `{protocol}`
- `{pathname}`
- `{search}`
- `{byline}`
- `{excerpt}`
- `{keywords}`
- `{date:YYYY-MM-DD}`

支持的大小写/命名风格变体包括：

- `:lower`
- `:upper`
- `:kebab`
- `:mixed-kebab`
- `:snake`
- `:mixed_snake`
- `:camel`
- `:pascal`
- `:obsidian-cal`

例如：

```text
{pageTitle:kebab}
{pageTitle:mixed_snake}
{date:YYYY-MM-DD}
```

## 快捷键与菜单

根据 `src/manifest.json` 当前定义，扩展支持以下命令：

- `Alt+Shift+M`：打开弹窗。
- `Alt+Shift+D`：将当前标签页保存为 Markdown。
- `Alt+Shift+C`：复制当前标签页 Markdown 到剪贴板。
- `Alt+Shift+L`：复制当前标签页 URL 为 Markdown 链接。
- 另外还支持复制选区为 Markdown、复制所选多个标签页链接、发送到 Obsidian 等命令，具体可在浏览器扩展快捷键页面中自定义。

## Obsidian 集成

项目保留了与 Obsidian 的集成能力，依赖社区插件 [Advanced Obsidian URI](https://vinzent03.github.io/obsidian-advanced-uri/)。基本步骤如下：

1. 安装并启用 `Advanced Obsidian URI`。
2. 打开扩展设置页。
3. 启用 Obsidian 集成。
4. 配置 Vault 名称和目标文件夹。
5. 使用右键菜单或相关命令将内容发送到 Obsidian。

## 权限说明

当前版本在 `src/manifest.json` 中声明了这些核心权限：

- `<all_urls>`：读取网页内容，并允许处理不同站点的页面与图片。
- `activeTab`：访问当前活动标签页内容。
- `downloads`：下载 Markdown 文件和图片。
- `storage`：保存扩展设置。
- `contextMenus`：提供右键菜单操作。
- `clipboardWrite`：复制 Markdown 到剪贴板。
- `webRequest` / `webRequestBlocking`：为图片预下载请求注入请求头，改善离线图片保存成功率。

如果你不需要防盗链图片处理，这两个 `webRequest` 相关权限就是这次 fork 相比上游更值得注意的权限变化。

## 开发

源码主要位于 `src/`。常用命令定义在 `src/package.json`：

```bash
cd src
npm install
npm run build
npm run start:firefoxdeveloper
```

当前仓库没有配置自动化测试脚本，修改后更适合通过真实网页做手动验证，重点验证：

- 普通文章导出。
- 仅导出选区。
- 下载图片和 Base64 嵌入。
- 防盗链图片站点。
- 含代码块、数学公式、表格的页面。
- Obsidian 集成路径和文件名模板。

## 依赖库

项目使用的主要第三方库：

- [Readability.js](https://github.com/mozilla/readability)：提取网页正文。
- [Turndown](https://github.com/mixmark-io/turndown)：将 HTML 转为 Markdown。
- [Moment.js](https://momentjs.com)：格式化模板中的日期。

## 版本说明

这个 fork 当前代码清单版本为 `3.4.1`。上游 README 中的 `3.4.0` 历史说明仍可参考，完整历史见 `CHANGELOG.md`。

如果后续继续围绕“纯离线 Markdown”演进，建议优先关注：
- 图片下载失败后的 Markdown 回退策略。
- 并发下载同一图片 URL 时的 Referer 跟踪隔离。
- 更明确的图片失败提示与导出报告。

## 致谢

原项目作者为 [Gordon Pedersen / deathau](https://github.com/deathau)。

Markdown 标志图标来自 https://github.com/dcurtis/markdown-mark
