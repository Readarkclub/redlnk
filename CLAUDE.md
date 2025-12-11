# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

RedInk (红墨) 是一个小红书AI图文生成器，用户输入一句话即可生成完整的小红书图文内容。

## 技术栈

- **后端**: Python 3.11+ / Flask / uv (包管理)
- **前端**: Vue 3 + TypeScript / Vite / Pinia
- **AI**: Google Gemini (文本) + 多种图片生成服务商

## 常用命令

### 后端开发
```bash
# 安装依赖
uv sync

# 启动开发服务器 (监听 localhost:12398)
uv run python -m backend.app

# 运行测试
uv run pytest tests/
```

### 前端开发
```bash
cd frontend

# 安装依赖
pnpm install

# 启动开发服务器 (监听 localhost:5173，代理 API 到 12398)
pnpm dev

# 类型检查
pnpm type-check

# 构建生产版本
pnpm build
```

### Docker
```bash
# 快速启动
docker run -d -p 12398:12398 -v ./history:/app/history -v ./output:/app/output histonemax/redink:latest

# 使用 docker-compose
docker-compose up -d
```

## 架构设计

### 后端分层架构 (`backend/`)
```
backend/
├── app.py              # Flask 应用入口
├── config.py           # 配置管理（YAML 热加载）
├── routes/             # 蓝图路由层
│   ├── config_routes.py    # 配置管理 API
│   ├── history_routes.py   # 历史记录 API
│   ├── image_routes.py     # 图片生成/管理 API
│   └── outline_routes.py   # 大纲生成 API
├── services/           # 业务逻辑层
│   ├── history.py      # 历史记录管理
│   ├── image.py        # 图片处理
│   └── outline.py      # 大纲生成
├── generators/         # 图片生成器（工厂模式）
│   ├── factory.py      # 生成器工厂
│   ├── google_genai.py # Google GenAI 实现
│   └── openai_compatible.py
├── utils/              # 工具库
│   ├── text_client.py  # 文本生成客户端（统一接口）
│   └── genai_client.py # GenAI 客户端
└── prompts/            # AI 提示词模板
```

### 前端结构 (`frontend/src/`)
```
views/          # 页面组件 (Home, Outline, Generate, Result, History, Settings)
components/     # 可复用组件
stores/         # Pinia 状态管理 (generator.ts)
api/            # API 调用封装
router/         # Vue Router 配置
```

### 设计模式

1. **工厂模式** - 图片/文本生成器通过工厂创建，支持多服务商切换
2. **蓝图模式** - Flask 模块化路由，按功能拆分
3. **热加载配置** - YAML 配置文件修改后无需重启

### 配置系统

- `text_providers.yaml` - 文本生成服务商配置
- `image_providers.yaml` - 图片生成服务商配置
- 支持 Web 界面配置和 YAML 文件配置两种方式

### 数据持久化

- `history/` - 历史记录 (JSON 文件)
- `output/` - 生成的图片

## API 端口

| 服务 | 端口 | 说明 |
|------|------|------|
| Flask 后端 | 12398 | API + 静态文件 |
| Vite 前端 | 5173 | 开发服务器 |

## 关键流程

1. 用户输入主题 → 调用文本生成 API → 返回多页大纲
2. 用户确认大纲 → 调用图片生成 API → 并发/顺序生成图片
3. 图片保存到 `output/` → 历史记录保存到 `history/`
