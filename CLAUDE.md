# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个无限图片浏览器，为 Stable Diffusion WebUI 和其他 AI 图像生成软件提供高性能的图像管理、搜索和浏览功能。项目采用前后端分离架构，支持多种部署方式。

## 核心架构

### 后端 (Python + FastAPI)
- **主入口**: `app.py` - 独立服务器模式的启动入口
- **API 核心**: `scripts/iib/api.py` - 包含所有 REST API 端点
- **数据库层**: `scripts/iib/db/datamodel.py` - SQLite 数据模型和查询逻辑
- **图像解析器**: `scripts/iib/parsers/` - 支持多种 AI 软件的元数据解析

### 前端 (Vue 3 + TypeScript + Ant Design Vue)
- **位置**: `vue/` 目录
- **技术栈**: Vue 3 + Vite + TypeScript + Pinia + Ant Design Vue
- **构建产物**: `vue/dist/` - 生产环境静态文件

### 数据库设计
核心表结构：
- `image` - 图像基本信息
- `tag` - 标签数据
- `image_tag` - 图像标签关联表
- `folder` - 文件夹信息
- `extra_path` - 自定义扫描路径

## 开发命令

### 后端开发
```bash
# 安装依赖
pip install -r requirements.txt

# 独立运行服务器
python app.py --port 8000 --host 127.0.0.1

# 更新图像索引
python app.py --update_image_index

# 预生成缓存
python app.py --generate_image_cache --generate_video_cover
```

### 前端开发
```bash
cd vue

# 安装依赖
yarn install

# 开发模式 (代理到后端 8000 端口)
yarn dev

# 类型检查
yarn type-check

# 构建生产版本
yarn build
```

### 桌面应用 (Tauri)
```bash
cd vue

# 开发模式
yarn tauri

# 构建桌面应用
yarn tauri-build

# 调试构建
yarn tauri-build-debug
```

## 关键概念

### 标签匹配系统
项目实现了三种标签匹配逻辑，位于 `ImageTag.get_images_by_tags()`:

1. **完全匹配** (`and_tags`): 图像必须包含所有指定标签
2. **任意匹配** (`or_tags`): 图像包含任一指定标签即可  
3. **排除匹配** (`not_tags`): 图像不能包含任何指定标签

### 图像解析器 (Parsers)
- 每个 AI 软件有独立的解析器模块
- 解析器负责从图像文件中提取生成参数和元数据
- 支持的软件：SD WebUI、ComfyUI、Fooocus、NovelAI、StableSwarm UI 等

### 缓存系统
- **图像缓存**: WebP 格式缩略图，可配置大小
- **视频封面**: 从视频中提取的封面图
- **缓存目录**: 通过 `IIB_CACHE_DIR` 环境变量配置

### 安全控制
通过环境变量配置访问控制：
- `IIB_SECRET_KEY`: 认证密钥
- `IIB_ACCESS_CONTROL`: 文件系统访问控制
- `IIB_ACCESS_CONTROL_ALLOWED_PATHS`: 允许访问的路径

## API 架构

### 核心 API 端点
- `/infinite_image_browsing/db/match_images_by_tags` - 标签匹配搜索
- `/infinite_image_browsing/db/search_by_substr` - 文本搜索
- `/infinite_image_browsing/files` - 文件系统浏览
- `/infinite_image_browsing/image-thumbnail` - 缩略图服务

### 数据流
1. 图像扫描 → SQLite 数据库
2. 前端请求 → FastAPI 路由 → 数据库查询
3. 结果过滤（安全控制）→ JSON 响应

## 部署方式

1. **SD WebUI 扩展**: 作为插件运行在 WebUI 环境中
2. **独立 Python 服务**: 使用 `app.py` 启动独立服务器
3. **桌面应用**: Tauri 打包的跨平台桌面应用
4. **库模式**: 通过 iframe 嵌入其他应用

## 开发注意事项

### Python 环境要求
- Python 3.7+
- 主要依赖：FastAPI、Pillow、SQLite3、imageio

### 前端开发环境
- Node.js 16+
- 开发时前端自动代理到后端 `127.0.0.1:8000`
- 生产构建输出到 `vue/dist/`

### 数据库迁移
- 数据库文件：`iib.db` (可通过环境变量修改)
- 自动备份机制，最多保留 8 个备份文件
- 表结构变更通过代码自动处理