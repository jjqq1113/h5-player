# 中转播放页 · 部署包

`index.html` 就是全部内容（单文件、无依赖、无外部请求）。把它放到**任意公网可访问的
HTTPS 地址的根路径**，那个地址就是"中转域名"。

## 它干什么

家长扫码 → 打开 `https://你的地址/?u=<视频直链>` → 页面用 `<video>` 播。
页面头部有 `<meta name="referrer" content="no-referrer">`，所以视频请求**不带 Referer**。

为什么必须这样：`hsyx-videos.oss-cn-beijing.aliyuncs.com` 的防盗链白名单只有
`*.shiguangshufang.com`。
- 空 Referer → **206，能播**（已实测）
- 微信域 Referer（`servicewechat.com` / `mp.weixin.qq.com`）→ **403**（已实测）

微信打开 `aliyuncs.com` 的链接会套一层风险中间页，点"继续"之后 Referer 就变成微信域，
于是 403。我们的页面把 Referer 抹掉，OSS 就放行。

## 三种落地方式（按省事程度）

### 1. Netlify Drop（最快，无需账号也能试）

1. 打开 https://app.netlify.com/drop
2. 把本目录整个拖进去
3. 拿到形如 `https://随机名.netlify.app` 的地址 —— 就是中转前缀

> 域名是 `*.netlify.app`，公网可访问；长期用建议注册账号以免站点被回收。

### 2. GitHub Pages

```bash
# 在本目录
git init && git add -A && git commit -m "h5 player"
# 建一个公开仓库（网页上建也可以），然后：
git remote add origin https://github.com/<你的账号>/<仓库名>.git
git branch -M main && git push -u origin main
# 仓库 Settings → Pages → Source 选 main / root → 保存
```

地址形如 `https://<账号>.github.io/<仓库名>/`（仓库名若是 `<账号>.github.io` 则无子路径）。
`.nojekyll` 已放好，避免 Jekyll 处理。

#### 不想敲命令：网页点选版

1. github.com → 右上角 `+` → **New repository** → 名字填 `h5-player` → **Public** → Create
2. 进仓库 → **Add file → Upload files** → 把本目录的 `index.html`、`README.md`、`.nojekyll`
   一起拖进去（macOS 里 `⌘⇧.` 显示隐藏文件才能选到 `.nojekyll`）→ Commit changes
3. **Settings → Pages** → Source 选 `Deploy from a branch` → Branch 选 `main` / `/(root)` → Save
4. 等 30–90 秒，页面顶部会出现 `Your site is live at https://<账号>.github.io/h5-player/`

#### 想全自动（推荐）

```bash
# 1) 建 token：https://github.com/settings/tokens → Generate new token (classic) → 勾 repo
# 2) 存到本地（不要贴在聊天里）：
echo ghp_你的token > ~/.gh_token && chmod 600 ~/.gh_token
# 3) 一键发布 + 等构建 + 打印地址：
bash publish_github_pages.sh
```

脚本会：建仓库（已存在就复用）→ 上传 3 个文件 → 开启 Pages → 每 3 秒探一次，
出现 `PAGES_URL=...` 就是可用地址。改页面后重跑同一条命令即可更新。

### 3. 已有服务器 / 对象存储静态托管

把 `index.html` 传到站点根目录即可。要求：
- **HTTPS**（微信里 http 会被拦）
- 根路径能直接打开（不需要登录、不需要跳转）
- 不要加 `X-Frame-Options: DENY` 之类（无所谓，但别改成下载）

## 接进去

1. 把地址写进技能目录 `h5_base.txt`（一行一个候选，可写多条）：

       https://你的地址/

2. 体检：

       python3 <技能目录>/scripts/check_h5.py

   退出码必须是 `0`。

3. 出 1 份样张 → **真手机扫码** → 视频能播 → 再跑整批。

## 验收要点

- 页面能打开，且地址带 `?u=` 参数时视频自动加载
- 白名单只放行 `*.shiguangshufang.com` 与 `hsyx-*.oss-cn-beijing.aliyuncs.com`，
  乱填地址会被页面自己拦掉（不会变成开放跳板）
- 手机微信里打开时，**不应**出现"非微信官方网页"中间页（出现说明域名被标风险，换一个）
