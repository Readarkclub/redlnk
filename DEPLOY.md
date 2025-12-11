# RedInk 部署指南

## 推荐方案：Railway 一键部署

### 前置准备

1. **获取 Gemini API Key**
   - 访问 [Google AI Studio](https://aistudio.google.com/apikey)
   - 点击 "Create API Key"
   - 复制生成的 API Key（格式：`AIza...`）

2. **注册 Railway 账号**
   - 访问 [railway.app](https://railway.app)
   - 使用 GitHub 账号登录

### 部署步骤

#### 方式一：从 GitHub 部署（推荐）

1. **Fork 或推送代码到 GitHub**
   ```bash
   git add .
   git commit -m "准备 Railway 部署"
   git push origin main
   ```

2. **在 Railway 创建项目**
   - 点击 "New Project"
   - 选择 "Deploy from GitHub repo"
   - 选择你的 RedInk 仓库

3. **等待部署完成**
   - Railway 会自动检测 Dockerfile 并构建
   - 构建完成后会分配一个 URL（如 `redink-xxx.up.railway.app`）

4. **在 Web 界面配置 Gemini API Key**
   - 访问部署后的 URL
   - 进入 **设置** 页面
   - 添加 Gemini 文本服务商和图片服务商
   - 填入你的 API Key

#### 方式二：使用 Railway CLI

```bash
# 安装 Railway CLI
npm install -g @railway/cli

# 登录
railway login

# 初始化项目
railway init

# 部署
railway up

# 部署完成后，访问 Web 界面配置 API Key
```

### ⚠️ 重要：API Key 配置方式

本项目的 API Key **通过 Web 界面配置**，不使用环境变量。这样设计的好处：
- 无需重新部署即可修改配置
- 支持多个服务商切换
- API Key 不会暴露在代码或环境变量中

**配置步骤：**
1. 访问部署后的 URL
2. 点击右上角 **设置** 图标
3. 在"文本生成服务商"中添加 Gemini：
   - 类型：`google_gemini`
   - API Key：你的 Gemini API Key
   - 模型：`gemini-2.0-flash`
4. 在"图片生成服务商"中添加 Gemini：
   - 类型：`google_genai`
   - API Key：你的 Gemini API Key
   - 模型：`gemini-2.0-flash-preview-image-generation`
   - 高并发：关闭（推荐）

### 数据持久化

Railway 提供持久化卷：

```bash
# 在 Railway 控制台添加 Volume
# 挂载路径：
/app/history  → 历史记录
/app/output   → 生成的图片
```

---

## 备选方案：Render 部署

### 部署步骤

1. 访问 [render.com](https://render.com)
2. 点击 "New" → "Web Service"
3. 连接 GitHub 仓库
4. 选择 Docker 作为运行环境
5. 点击 "Create Web Service"
6. **部署完成后，访问 Web 界面配置 API Key**

---

## 本地开发

```bash
# 1. 安装依赖
uv sync
cd frontend && pnpm install && cd ..

# 2. 启动后端
uv run python -m backend.app

# 3. 启动前端（另一个终端）
cd frontend && pnpm dev

# 4. 访问 http://localhost:5173，在设置页面配置 API Key
```

---

## 常见问题

### Q: 图片生成超时怎么办？

Railway 默认请求超时 30 秒，AI 图片生成可能需要更长时间。

解决方案：在 Railway 设置中增加超时时间，或在设置页面关闭"高并发模式"。

### Q: 如何查看日志？

```bash
# Railway CLI
railway logs

# 或在 Railway 控制台的 "Deployments" 页面查看
```

### Q: 如何更新部署？

```bash
git push origin main
# Railway 会自动重新部署
```

---

## Gemini API 配额说明

| 类型 | 免费额度 | 说明 |
|------|----------|------|
| **文本生成** | 1500 请求/天 | gemini-2.0-flash |
| **图片生成** | 50 张/天 | gemini-2.0-flash-preview-image-generation |

⚠️ **注意**：GCP $300 试用账号建议关闭高并发模式，避免触发速率限制。
