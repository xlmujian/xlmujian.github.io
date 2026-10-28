# 幕间官网 · 部署说明

站是纯静态的：没有构建步骤、没有依赖、没有后端。三个页面 + 一个 JSON + 一张图。

## 文件

```
index.html          首页
download.html       下载页（按钮直连安装包）
notes.html          更新日志
site.css            三个页面共用的样式
latest.json         版本源：客户端检查更新、下载页版本号、更新页版本号都读它
assets/shot-chat.png 界面截图
download/           放安装包（见下）
```

## latest.json

```json
{
  "version": "0.1.0",
  "url": "https://xuanlin.xin/download/Interlude-Setup.exe",
  "notes": "这一版改了什么",
  "publishedAt": "2026-10-28"
}
```

- 客户端「设置 → 关于 → 检查更新」请求的是 `https://xuanlin.xin/latest.json`
- 版本号比本机大就提示「有新版本」，点击后打开 `url`
- **发版流程**：装好新版本 → 更新 `latest.json` 里的 `version` / `notes` / `publishedAt` → 把新安装包传到 `download/Interlude-Setup.exe`（覆盖旧文件，这样链接永远是最新版）

## 安装包放在哪

两个选择，都不需要服务器：

**A. 跟站放在一起（最简单）**

把 `Interlude-0.1.0-Setup.exe` 改名 `Interlude-Setup.exe`，放进 `download/` 目录一起上传。GitHub Pages 单文件上限 100 MB，**129 MB 的安装包超限了**，所以这条路只在 Cloudflare Pages 上可行（单文件 25 MB 限制也不够）。

→ 实际上两个静态托管都不适合放 129 MB 文件。

**B. 对象存储（推荐）**

Cloudflare R2（免费 10 GB 存储、出站流量免费）或阿里云 OSS：

```
R2 → 自定义域名 dl.xuanlin.xin → download/Interlude-Setup.exe
```

然后把 `latest.json` 里的 `url` 和 `download.html` 按钮的 `href` 都指向它。这样站点和安装包分开，站点可以随便迁。

## 托管选择

| 平台 | 备案 | 国内速度 | 费用 |
|---|---|---|---|
| GitHub Pages | 不需要 | 一般，偶有干扰 | 免费 |
| Cloudflare Pages | 不需要 | 一般 | 免费 |
| 香港轻量服务器 | 不需要 | 好 | 几十/月 |
| 国内服务器 | **需要 ICP 备案** | 最好 | 便宜但备案 1–3 周 |

未备案的域名只能指向境外，这三行前两行都合规。

## GitHub Pages 步骤

1. 新建仓库，**名字必须是 `<用户名>.github.io`**（用户主页仓库）
2. 把本站文件推到仓库根目录（`index.html` 要在根）
3. 仓库 Settings → Pages → Source 选 `main` / `root` → 保存
4. Custom domain 填 `xuanlin.xin` → Save，等证书签发后勾 Enforce HTTPS
5. DNS：`@` 用 A 记录指向 `185.199.108.153 / .109.153 / .110.153 / .111.153`，`www` 用 CNAME 指向 `<用户名>.github.io`

> 根域名用 CNAME 不算标准做法，容易和将来的 MX 记录打架，用 A 记录更稳。

## Cloudflare Pages 步骤

1. 注册 Cloudflare，Add a site → 填 `xuanlin.xin` → 按提示把域名 NS 改到 Cloudflare 给的那两个
2. Workers & Pages → Create → Pages → Upload assets → 拖入整个文件夹
3. Custom domains → 绑定 `xuanlin.xin`

不用改仓库、不用命令行，改完把文件重新拖一次即可。

## 本地预览

直接双击 `index.html` 就行。版本号那种动态读取需要 `file://` 之外的来源，想看效果可以在目录里起个静态服务：

```powershell
cd C:\Users\Administrator\interlude-site
npx --yes serve -l 4173 .
```

## 发版检查清单

- [ ] `package.json` 的 `version` 改成新版本号，重新打包
- [ ] `latest.json` 的 `version` 与安装包一致
- [ ] 新安装包覆盖上传到 `download/Interlude-Setup.exe`
- [ ] `notes.html` 加一条该版本的记录
- [ ] 打开站点点一次「下载最新版」，确认拿到的是新包
