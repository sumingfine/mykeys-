# MyKeys

[English](#english) | [中文](#中文)

---

<a id="中文"></a>

## 中文

一个跑在 Cloudflare Workers 上的 Telegram 密码管理机器人。

发条消息就能存账号密码，发个关键词就能模糊搜索找回来。所有敏感信息 AES-256-GCM 加密后存入 Cloudflare D1 数据库，不花一分钱，不需要服务器。

### 功能

- 发一条消息保存账号密码，格式：`用途 网站 账号 密码 [备注]`
- 发关键词模糊搜索，点按钮查看完整信息
- 菜单里点 /list 查看所有条目
- 点按钮删除不需要的记录
- 只有你自己能用，别人发消息会被拒绝

### 安全性

- 账号、密码、备注字段 AES-256-GCM 加密存储，数据库里只有密文
- Bot Token、加密密钥等敏感配置通过 Cloudflare Secrets 加密保存，不在代码里
- 管理接口需要密钥才能访问
- 通过 Telegram User ID 做身份验证，只有你能操作
- 其他所有路径返回 404

### 前置条件

- 一个 Cloudflare 账号（免费就行）
- Node.js 18+
- 一个 Telegram Bot（找 @BotFather 创建）
- 你的 Telegram User ID（找 @userinfobot 获取）

### 部署步骤

整个过程大概 5 分钟。

#### 1. 克隆项目

```bash
git clone https://github.com/cocojojo5213/mykeys.git
cd mykeys
npm install
```

#### 2. 登录 Cloudflare

```bash
npx wrangler login
```

浏览器会弹出来让你授权，点同意就行。

#### 3. 创建数据库

```bash
npx wrangler d1 create password-bot-db
```

命令会输出一段信息，里面有个 `database_id`，把它复制下来。

打开 `wrangler.toml`，把 `your-database-id-here` 替换成你刚才拿到的 ID。

同时把 `ALLOWED_USER_ID` 改成你自己的 Telegram User ID。

#### 4. 设置密钥

需要设置三个密钥：

- **TELEGRAM_BOT_TOKEN**：找 @BotFather 创建 bot 时给你的 token
- **ENCRYPT_KEY**：自己想一个 32 位的字符串，用来加密密码，设好之后别丢了也别改
- **ADMIN_SECRET**：管理密钥，用来访问初始化接口，随便设一个你记得住的

逐个设置：

```bash
npx wrangler secret put TELEGRAM_BOT_TOKEN
npx wrangler secret put ENCRYPT_KEY
npx wrangler secret put ADMIN_SECRET
```

每条命令会提示你输入值，粘贴进去回车就行。

也可以创建一个临时文件批量设置：

```json
// .secrets.json（用完记得删）
{
  "TELEGRAM_BOT_TOKEN": "你的bot-token",
  "ENCRYPT_KEY": "你的32位加密密钥",
  "ADMIN_SECRET": "你的管理密钥"
}
```

```bash
npx wrangler secret bulk .secrets.json
rm .secrets.json
```

#### 5. 部署

```bash
npx wrangler deploy
```

部署成功后会输出一个 URL，类似 `https://mykeys.xxx.workers.dev`。

#### 6. 初始化数据库

浏览器访问（把 URL 和密钥换成你自己的）：

```
https://mykeys.xxx.workers.dev/init?key=你的ADMIN_SECRET
```

看到"数据库初始化完成"就行。

#### 7. 设置 Webhook

浏览器访问：

```
https://mykeys.xxx.workers.dev/setWebhook?key=你的ADMIN_SECRET
```

看到 `"result":true` 就行。

#### 8. 开始使用

打开 Telegram，找到你的 bot，发条消息试试：

```
claude claude.ai myemail@test.com mypassword123
```

然后发 `claude` 搜索一下，应该能找到刚才存的。

### 使用方法

**保存账号**：直接发消息，至少四段，用空格分开

```
用途 网站 账号 密码
用途 网站 账号 密码 备注信息
```

比如：

```
claude claude.ai test@mail.com abc123
github github.com zhangsan mypass 开了2FA
```

**搜索**：直接发关键词，支持模糊匹配

```
cla
github
```

只有一条结果会直接显示，多条结果会列出按钮让你选。

**查看全部**：点菜单里的 /list

**删除**：查看详情时点删除按钮，或者在列表里点"进入删除模式"

### 注意事项

- ENCRYPT_KEY 设好之后不要改，改了之前存的数据就解不开了
- 如果要换 ENCRYPT_KEY，需要先把旧数据全删了
- 建议给 Cloudflare 账号开两步验证
- 可以在 Telegram 里设置聊天自动删除消息，避免密码留在聊天记录里

---

<a id="english"></a>

## English

A Telegram bot for managing passwords, running on Cloudflare Workers.

Send a message to save credentials, send a keyword to search them back. All sensitive data is encrypted with AES-256-GCM and stored in Cloudflare D1. Zero cost, no server needed.

### Features

- Save credentials by sending: `name site account password [notes]`
- Fuzzy search by keyword, tap buttons to view details
- Tap /list to see all saved entries
- Delete entries with inline buttons
- Only you can use it, others get rejected

### Security

- Account, password, and notes fields are AES-256-GCM encrypted at rest
- Bot token, encryption key, and admin secret stored via Cloudflare Secrets
- Admin endpoints require a secret key
- Telegram User ID verification, only your ID is allowed
- All other paths return 404

### Prerequisites

- A Cloudflare account (free tier works)
- Node.js 18+
- A Telegram Bot (create one via @BotFather)
- Your Telegram User ID (get it from @userinfobot)

### Deployment

Takes about 5 minutes.

#### 1. Clone the repo

```bash
git clone https://github.com/cocojojo5213/mykeys.git
cd mykeys
npm install
```

#### 2. Log in to Cloudflare

```bash
npx wrangler login
```

A browser window will open for authorization.

#### 3. Create the database

```bash
npx wrangler d1 create password-bot-db
```

Copy the `database_id` from the output.

Open `wrangler.toml`, replace `your-database-id-here` with your database ID.

Also replace `ALLOWED_USER_ID` with your Telegram User ID.

#### 4. Set secrets

You need three secrets:

- **TELEGRAM_BOT_TOKEN**: the token from @BotFather
- **ENCRYPT_KEY**: a 32-character string for encryption. Don't lose it, don't change it.
- **ADMIN_SECRET**: a key for accessing admin endpoints, pick something you can remember

Set them one by one:

```bash
npx wrangler secret put TELEGRAM_BOT_TOKEN
npx wrangler secret put ENCRYPT_KEY
npx wrangler secret put ADMIN_SECRET
```

Or use a temporary file for bulk setup:

```json
// .secrets.json (delete after use)
{
  "TELEGRAM_BOT_TOKEN": "your-bot-token",
  "ENCRYPT_KEY": "your-32-char-encryption-key",
  "ADMIN_SECRET": "your-admin-secret"
}
```

```bash
npx wrangler secret bulk .secrets.json
rm .secrets.json
```

#### 5. Deploy

```bash
npx wrangler deploy
```

You'll get a URL like `https://mykeys.xxx.workers.dev`.

#### 6. Initialize the database

Visit in your browser:

```
https://mykeys.xxx.workers.dev/init?key=YOUR_ADMIN_SECRET
```

#### 7. Set the webhook

Visit in your browser:

```
https://mykeys.xxx.workers.dev/setWebhook?key=YOUR_ADMIN_SECRET
```

You should see `"result":true`.

#### 8. Start using it

Open Telegram, find your bot, and send:

```
claude claude.ai myemail@test.com mypassword123
```

Then send `claude` to search for it.

### Usage

**Save**: send a message with at least 4 parts separated by spaces

```
name site account password
name site account password notes
```

Examples:

```
claude claude.ai test@mail.com abc123
github github.com zhangsan mypass 2FA enabled
```

**Search**: send a keyword, fuzzy matching supported

```
cla
github
```

One result shows directly, multiple results show as buttons.

**List all**: tap /list in the menu

**Delete**: tap the delete button in detail view, or use "delete mode" from the list

### Important

- Do not change ENCRYPT_KEY after setup, or previously saved data becomes unreadable
- If you must change it, delete all existing data first
- Enable 2FA on your Cloudflare account
- Consider setting auto-delete messages in your Telegram chat

## Tech Stack

- Cloudflare Workers
- Cloudflare D1 (SQLite)
- Cloudflare Secrets
- Web Crypto API (AES-256-GCM)
- TypeScript
- Wrangler CLI

## License

MIT
