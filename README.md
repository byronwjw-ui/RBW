# RBW — Confidential Strategy Deck

**CONFIDENTIAL — DO NOT DISTRIBUTE.** 商业机密文件，未经书面许可不得复制、转发、上传或传播。

## 三档受众分级 / Three Audience Tiers

| Tier | 名字 | 页数 | 受众 | 密码 |
|---|---|---|---|---|
| 1 | **Public** | 6 页 | 任何人，包括陌生人 | 无 |
| 2 | **Investor** | 28 页 | 中国国内市场投资人 | 密码 1 |
| 3 | **Full Access · 完整版** | 43 页 | 创始团队、资深顾问、家人 | 密码 2 |

**两套密码独立。** Investor 密码不能解锁完整版，反之亦然。所有解锁状态存在 `sessionStorage`，关浏览器自动清。

## 下载与使用 / Download & Use

1. 仓库右上角 **Code → Download ZIP**
2. 解压到桌面
3. 双击 `RBW_Confidential_Strategy_Deck.html` 在浏览器中打开
4. **必须确保所有 7 个 JS/HTML 文件在同一文件夹**

## 文件清单 / Files

| 文件 | 说明 |
|---|---|
| `RBW_Confidential_Strategy_Deck.html` | 主文件。双击打开。含 CSS + 切换逻辑 + 密码门。 |
| `slides_head_en.js` | English slides 1–10（含 Public 专属页 + Founding Team） |
| `slides_head_zh.js` | 中文版 slides 1–10 |
| `slides.js` | English slides 11–25 |
| `slides2.js` | English slides 26–43（含附录） |
| `slides_zh.js` | 中文版 slides 11–25 |
| `slides2_zh.js` | 中文版 slides 26–43（含附录） |

---

## 🔑 部署前必做：设置你自己的密码

主 HTML 文件里有两行**占位符密码 hash**，你必须替换成自己的。

### 步骤

1. 在任意浏览器打开 deck（先用占位符密码也行）
2. 按 **F12** 打开开发者工具，切到 **Console** 标签
3. 输入下面这行，回车（密码替换成你想要的）：

```js
await sha256("我的投资人密码123" + "RBW-2025-confidential")
```

4. 控制台会返回一串 64 字符的 hash，例如：
   `"a3f5c8...."`

5. 同样方法生成 Full Access 密码的 hash：

```js
await sha256("我的完整版密码456" + "RBW-2025-confidential")
```

6. 打开 `RBW_Confidential_Strategy_Deck.html`，找到下面这两行（约在 `<script>` 标签开头）：

```js
const PASSWORD_HASH_INVESTOR = "8e9d2c5f..."; // PLACEHOLDER
const PASSWORD_HASH_FULL     = "1a2b3c4d..."; // PLACEHOLDER
```

7. **把引号里的占位符 hash 替换成你刚才生成的真 hash**，保存。

8. 刷新浏览器，用你的密码测试一下，确认能解锁。

### 安全说明

- 密码的明文**永远不会出现在文件里**，存的是 SHA-256(密码 + 盐值) 的 hash
- 盐值是 `"RBW-2025-confidential"`（写在 HTML 顶部，可以改但不必）
- 这是**轻度保护**，不是银行级安全。前端密码门主要防"路人手贱点一下就看到完整版"，挡不住技术性破解者
- 真正机密的信息**永远不要**放在前端 deck 里——这套机制保护的是"展示分级"，不是"加密"

---

## 顶部控件 / Top Controls

- **EN / 中文** — 语言切换（键盘 **L**）
- **Public / Investor / Full Access** — 受众切换（键盘 **I** 在已解锁档之间循环）
- **TOC** — 目录（键盘 **T**）
- **‹ ›** — 翻页
- **PDF** — 打印 / 导出 PDF（键盘 **P**）

按钮上的 🔒 表示该档锁定，✓ 表示该档已解锁但当前不在该档。

## 键盘 / Keyboard

| Action | Key |
|---|---|
| Next / 下一页 | `→` `Space` `PageDown` |
| Prev / 上一页 | `←` `PageUp` |
| First / 首页 | `Home` |
| Last / 末页 | `End` |
| TOC / 目录 | `T` |
| Close overlay | `Esc` |
| Cycle audience tier (only unlocked) | `I` |
| Toggle language | `L` |
| Print / 打印 | `P` |

## PDF 导出 / Export as PDF

桌面浏览器 → 按 **P** 或点 **PDF** → 系统对话框选 "Save as PDF" → 纸张 **1280 × 720** 或 A4 横向 → 背景图形 **开** → 保存。

> **重要：** 导出前先切到你想发的档。Public 模式导出 6 页 PDF；Investor 模式导出 28 页 PDF；Full Access 模式导出 43 页 PDF。

---

## 部署到腾讯云 / Deploy to Tencent Cloud

### 准备
- 一台 Linux 服务器（CVM / 轻量应用服务器都行）
- 一个域名（可选，但建议）
- 装好 Nginx（默认配置就行）

### 步骤

1. **替换密码 hash**（见上文 🔑 章节）
2. **上传文件**到服务器，例如 `/var/www/rbw/`：
   ```bash
   scp -r ~/Desktop/RBW/* user@your-server:/var/www/rbw/
   ```
3. **配置 Nginx**，例如 `/etc/nginx/conf.d/rbw.conf`：
   ```nginx
   server {
       listen 80;
       server_name your-domain.com;
       root /var/www/rbw;
       index RBW_Confidential_Strategy_Deck.html;

       location / {
           try_files $uri $uri/ =404;
           add_header X-Frame-Options "DENY";
           add_header X-Content-Type-Options "nosniff";
           add_header Referrer-Policy "no-referrer";
       }

       # 禁止抓取
       location = /robots.txt {
           add_header Content-Type text/plain;
           return 200 "User-agent: *\nDisallow: /\n";
       }
   }
   ```
4. `sudo nginx -t && sudo systemctl reload nginx`
5. 浏览器访问 `http://your-domain.com` 即可

### 加 HTTPS（强烈建议）

```bash
sudo certbot --nginx -d your-domain.com
```

### 想加第二层服务器密码（除前端密码外）

在 Nginx 配置加：
```nginx
location / {
    auth_basic "RBW Confidential";
    auth_basic_user_file /etc/nginx/.htpasswd;
    try_files $uri $uri/ =404;
}
```
然后：
```bash
sudo htpasswd -c /etc/nginx/.htpasswd youruser
```

---

## 安全声明 / Safety

本仓库不含真实外部资源名称、不含医疗治疗承诺、不含投资收益承诺、不含真实股权比例、不含真实个人身份。所有外部资源以 Resource A–E 代号呈现。详细信息披露须通过 NDA 与专业法律流程。

This repository contains no real external resource names, no medical claims, no investment return guarantees, no real equity percentages, and no real individual identities. All external resources are referenced as Resource A–E. Further disclosure happens only via NDA and professional legal process.

Version 1.0 · RBW Confidential Strategy Deck
