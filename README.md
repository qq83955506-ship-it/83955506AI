# 83955506 AI 工具箱（第一版）

一个中文单页个人工具箱，按“本地工具不依赖 AI、AI 请求走受保护的 Worker”设计。

## 已包含功能

- AI 写作：润色、改写、扩写、精简、专业化、整理成条目
- 小红书创作：根据用户提供的真实信息生成标题、笔记和配图建议
- AI 总结：核心结论、行动清单、会议纪要等
- AI 对话：对话记录保存在当前浏览器
- 文本批处理：去重、去空行、排序、反转、编号、大小写转换、分隔符转换
- IP / 端口管理：导入 TXT、去重、搜索、备注、删除、TXT 导出
- 提示词仓库：新建、编辑、删除、搜索、复制，保存在本机
- 本机数据备份与恢复
- Worker 登录验证、8 小时 HMAC 登录令牌、基本请求频率限制、输入长度限制
- 页面无外部字体、脚本 CDN 或图片依赖

## 文件

- `index.html`：前端，可由 Cloudflare Pages 托管
- `worker.js`：后端，负责登录和调用 Workers AI

## 部署步骤（建议先使用 Cloudflare 提供的域名测试）

### 第一步：部署网页

1. 新建一个 GitHub 仓库，例如 `83955506-ai-toolbox`。不要覆盖你现有的个人主页仓库。
2. 上传本目录中的 `index.html`（README 也可以一起上传）。
3. 在 Cloudflare Dashboard 打开 **Workers & Pages → Create → Pages → Connect to Git**，连接新仓库并部署。
4. 记下部署好的网址，例如 `https://your-toolbox.pages.dev`。这是网页的 Origin，后面要配置给 Worker。

也可以使用 Cloudflare Pages 的 Direct Upload 上传前端文件。

### 第二步：创建独立 Worker

1. 在 Cloudflare Dashboard 打开 **Workers & Pages → Create → Worker**，创建一个全新的 Worker，例如 `83955506-ai-toolbox-api`。
2. 将 `worker.js` 的全部内容复制到 Worker 编辑器，保存并部署。
3. 在 Worker 的 **Settings → Bindings** 添加 **Workers AI** 绑定，变量名称填 `AI`。
4. 在 **Settings → Variables and Secrets** 添加以下配置：

| 名称 | 类型 | 值 |
|---|---|---|
| `ADMIN_USER` | Secret 或普通变量 | 自己设置的管理员账号 |
| `ADMIN_PASSWORD` | Secret | 自己设置的强密码 |
| `SESSION_SECRET` | Secret | 至少 32 个随机字符，不要与管理员密码相同 |
| `ALLOWED_ORIGIN` | 普通变量 | Pages 网址的 Origin，例如 `https://your-toolbox.pages.dev`，不要带路径或末尾 `/` |
| `AI_MODEL` | 可选普通变量 | 默认 `@cf/meta/llama-3.1-8b-instruct`；更换前先核对 Workers AI 模型目录和用量价格 |

如果之后也要通过自定义域名访问网页，`ALLOWED_ORIGIN` 可以写多个 Origin，用英文逗号分隔，例如：

`https://your-toolbox.pages.dev,https://ai.example.com`

注意：Origin 只写协议、域名和可选端口，不要带 `/settings` 等路径。

### 第三步：网页登录 Worker

1. 打开部署好的 Pages 网页。
2. 点击右上角 **连接 AI**，进入“连接与设置”。
3. 填写 Worker 地址（例如 `https://83955506-ai-toolbox-api.your-subdomain.workers.dev`）、管理员账号和密码。
4. 点击 **登录并测试连接**。登录成功后即可使用 AI 工具。

Worker 地址是 Worker 的实际部署域名，不是 Pages 网页地址，也不要在后面加 `/api`。

## 使用说明

- 文本批处理、IP / 端口管理、提示词管理都可不登录使用。
- AI 功能需要 Worker 登录成功，AI 调用由 Worker 发起，网页中不需要放模型密钥。
- 提示词、IP 列表、对话和 AI 生成历史存放在浏览器 `localStorage` 中。换浏览器、换设备或清理站点数据可能丢失；请定期使用“导出本机数据备份”。
- 前端不会加载外部字体或 JS CDN，以减少不必要的外部依赖。但中国大陆访问 Cloudflare Pages / Workers 仍可能受跨境链路影响，本项目不保证大陆网络稳定性。
- IP / 端口管理只负责整理用户导入的文本，不执行公网扫描、端口探测或代理节点自动发现。

## 安全与费用提示

- 不要在 GitHub 里提交管理员密码、`SESSION_SECRET` 或任何 API 密钥。
- 不要复用个人主页 Worker 的管理员密码/密钥；建议为此工具单独创建 Worker 与凭据。
- 登录令牌有效期默认为 8 小时。退出登录会清除当前浏览器的令牌，但服务端无状态令牌不能立即撤销已经泄露的令牌；令牌会在到期后失效。
- Worker 内置的内存频率限制是基础防护，不能替代 Cloudflare 更强的 Rate Limiting 配置。若公开分享此网站，需进一步配置速率限制、使用量预算及访问控制。
- Workers AI 的可用模型、价格和免费额度会变化；请以 Cloudflare Dashboard 中当前模型与用量信息为准。未设置付费方式时，超出免费额度或所选模型不支持当前账户可能调用失败。
- AI 输出可能出错；产品内容请核对事实，避免输入敏感或机密信息。

## 故障排查

- **登录提示 origin not allowed**：检查 `ALLOWED_ORIGIN` 是否与浏览器地址栏的网页 Origin 完全一致，是否用了 `http` / `https` 不同协议，是否遗漏自定义域名。
- **缺少 Workers AI 绑定**：确认 Worker 已添加 Workers AI binding，变量名严格是 `AI`。
- **AI 调用失败或 502**：先检查 Workers AI 控制台中模型是否可用、账户额度和用量，再尝试恢复默认模型 `@cf/meta/llama-3.1-8b-instruct`。
- **登录成功但 AI 未连接**：刷新页面后重新登录；确认 Worker URL 正确且没有在末尾加 `/api`。
- **更换网页域名后失效**：把新网页 Origin 加入 `ALLOWED_ORIGIN`，然后重新部署 Worker。
