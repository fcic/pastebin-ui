# Pastebin-ui

这是一个基于 Cloudflare Workers 和 KV 的轻量级内容中转站项目，适合需要简洁、高效解决内容存储需求的开发者使用。无需依赖 R2 存储桶，部署方便，适用于小型项目。

该项目是原项目的历史版本，由于我个人认为还有需要，所以单独提取出来了，供大家使用，本人不拥有该项目，开源协议集成原项目，如有侵犯请联系删除。

原项目仓库：[Pastebin Worker - 历史版本](https://github.com/xiadd/pastebin-worker)

## 部署文档

### 1. 手动部署(推荐)

#### 获取 Cloudflare API Token

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)。
2. 点击右上角的个人头像，选择 **My Profile**。
3. 进入 **API Tokens** 页面。
4. 点击 `Create Token`，选择 `Edit Cloudflare Workers` 模板。
5. 配置并创建后，复制生成的 API Token。

![获取 API Token](./docs/get_api.png)

#### 创建 KV 存储

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)。
2. 点击左侧列表中的存储和数据库，选择 `KV`。
3. 创建两个`KV`，名称分别为：`PB`，`PBIMGS`。
4. 保存好`ID`，后期会用到。
   
![创建KV](/docs/create_kv.png)

#### 在 GitHub Actions 中设置 Secret

1. 打开项目的 GitHub 仓库。
2. 进入 **Settings > Secrets and variables > Actions**。
3. 点击 `New repository secret`。
4. 名称设置为 `CF_API_TOKEN`，值为刚刚生成的 API Token。

![设置 GitHub Secret](./docs/set_secret.png)

#### 修改环境变量

1. 在 `./static/.env` 文件中设置前端页面的环境变量。
   ```env
   VITE_API_BASE_URL=<你的 Cloudflare Worker 部署地址>
   ```
2. 确保该地址已在 Cloudflare 的 DNS 中添加，这个地址会自动添加到解析中。

![设置环境变量](./docs/set_env.png)

#### 修改 worker 配置文件

1. 在 `./wrangler.toml` 文件中修改以下变量：

![修改worker配置](./docs//set_worker_env.png)

#### 部署到 Cloudflare

- 每次将代码 push 到 `main` 分支后，GitHub Actions 会自动触发部署任务。
- 若需要手动运行部署任务：
  1. 打开 GitHub Actions 页面。
  2. 找到对应的 Action，点击 `Run workflow`。

![运行部署 Action](./docs/run_action.png)

### 2. 本地开发部署(DEV)

#### 配置 Cloudflare Workers KV Namespace

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)。
2. 创建两个 KV Namespace：
   - 一个用于存储文件（命名为 `PBIMG`）。
   - 一个用于存储文字（命名为 `PB`）。
3. 记录它们的 `ID`，后续需要使用。

#### 修改 `wrangler.toml`

```toml
name = "pastebin-worker"
compatibility_date = "2023-11-28"
account_id = "<你的 account_id>"
main = "src/index.ts"
workers_dev = false

vars = { ENVIRONMENT = "production" }
route = { pattern = "<你的域名>", custom_domain = true }

kv_namespaces = [
  { binding = "PB", id = "<PB kv id>" },
  { binding = "PBIMGS", id = "<PBIMG kv id>" }
]

[site]
bucket = "./static/dist"
```

- `account_id` 可在 Cloudflare Dashboard 的个人资料中找到。
- 如果不使用自定义域名，注释掉 `route` 即可。

### 启动服务

#### 后端启动

```bash
npm i @cloudflare/wrangler -g
wrangler login
wrangler dev
```

#### 前端启动

```bash
cd static
yarn install
yarn dev
```

启动完成后：
- 后端地址为 `http://localhost:8787`。
- 前端地址为 `http://localhost:5173`。

#### 测试前端打包效果

在 `static` 目录下运行：

```bash
yarn build
```

访问 `http://localhost:8787` 查看打包后的前端页面效果。

## 开发文档

### 环境初始化

1. 安装 Wrangler CLI：

   ```bash
   npm i @cloudflare/wrangler -g
   ```

2. 登录 Cloudflare 账号：

   ```bash
   wrangler login
   ```

3. 验证登录：

   ```bash
   wrangler whoami
   ```

   显示用户名即表示登录成功。

### 安装依赖

#### 后端依赖

```bash
yarn install
```

#### 前端依赖

```bash
cd static
yarn install
```

### 配置文件

参考 [部署文档](#配置-cloudflare-workers-kv-namespace) 中的 `wrangler.toml` 和 `.env` 设置。

### 启动开发环境

按照 [启动服务](#启动服务) 中的说明运行后端和前端。

### 测试

- 在本地访问 `http://localhost:5173` 测试前端。
- 测试后端接口地址为 `http://localhost:8787`。

### 部署到 Cloudflare

详见 [部署文档](#部署文档)。