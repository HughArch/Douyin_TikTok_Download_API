# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个基于 FastAPI 和 HTTPX 构建的高性能异步爬虫 API，支持抖音(Douyin)、TikTok 和 Bilibili 平台的数据爬取与无水印视频下载。

## 核心架构

### 三层架构设计

1. **`/crawlers` - 爬虫层**
   - `BaseCrawler`: 基础爬虫客户端，处理 HTTP 请求、重试、代理、并发控制
   - `DouyinWebCrawler`: 抖音网页版爬虫，使用 X-Bogus 和 A_Bogus 算法
   - `TikTokWebCrawler`: TikTok 网页版爬虫
   - `TikTokAPPCrawler`: TikTok APP 接口爬虫
   - `BilibiliWebCrawler`: Bilibili 网页版爬虫
   - `HybridCrawler`: 混合解析器，自动识别平台并调用对应爬虫

2. **`/app/api` - API 服务层**
   - FastAPI 应用，提供 RESTful API 接口
   - 各平台的 `endpoints/*.py` 文件定义路由和请求处理
   - `models/APIResponseModel.py` 定义响应模型

3. **`/app/web` - Web 界面层**
   - 使用 PyWebIO 构建的 Web 界面
   - 批量解析、下载等功能的视图组件

### 配置系统

项目使用 YAML 配置文件，配置分布在多个位置：
- `/config.yaml`: 主配置（API 端口、下载设置等）
- `/crawlers/douyin/web/config.yaml`: 抖音 Cookie 和代理设置
- `/crawlers/tiktok/web/config.yaml`: TikTok Cookie 和代理设置
- `/crawlers/tiktok/app/config.yaml`: TikTok APP 配置

**重要**: 由于抖音/TikTok 的风控，部署后必须更新对应 config.yaml 中的 Cookie。

## 常用开发命令

### 本地开发

```bash
# 安装依赖
pip install -r requirements.txt

# 启动开发服务器（支持热重载）
python start.py
# 或者
uvicorn app.main:app --host 0.0.0.0 --port 80 --reload
```

### Docker 部署

```bash
# 构建镜像
docker build -t douyin_tiktok_api .

# 运行容器
docker run -d --name douyin_tiktok_api -p 80:80 douyin_tiktok_api

# 查看日志
docker logs douyin_tiktok_api

# 停止并删除容器
docker stop douyin_tiktok_api && docker rm douyin_tiktok_api
```

### Linux 一键部署

```bash
wget -O install.sh https://raw.githubusercontent.com/Evil0ctal/Douyin_TikTok_Download_API/main/bash/install.sh && sudo bash install.sh
```

服务管理：
```bash
# 启动/停止服务
sudo systemctl start/stop Douyin_TikTok_Download_API.service

# 开机自启动
sudo systemctl enable Douyin_TikTok_Download_API.service

# 查看服务状态
sudo systemctl status Douyin_TikTok_Download_API.service

# 查看日志
sudo journalctl -u Douyin_TikTok_Download_API.service -f
```

### 项目更新

```bash
cd /www/wwwroot/Douyin_TikTok_Download_API/bash && sudo bash update.sh
```

## API 路由结构

- `/api/hybrid/*`: 混合解析接口（自动识别平台）
- `/api/douyin/web/*`: 抖音网页版 API
- `/api/tiktok/web/*`: TikTok 网页版 API
- `/api/tiktok/app/*`: TikTok APP API
- `/api/bilibili/web/*`: Bilibili 网页版 API
- `/api/download`: 视频下载接口
- `/api/ios/*`: iOS 快捷指令接口
- `/docs`: FastAPI 自动生成的 API 文档

## 开发注意事项

### Cookie 管理

由于平台风控，Cookie 需要定期更新：
1. 在浏览器中访问抖音/TikTok 网站
2. 登录账号
3. 复制完整的 Cookie
4. 粘贴到对应的 `crawlers/*/web/config.yaml` 中
5. 重启服务

### 异步编程规范

- 所有爬虫方法都是异步的 (`async def`)
- 使用 `asyncio.run()` 或在异步上下文中调用
- 并发控制通过 `BaseCrawler` 的 `max_tasks` 和 `max_connections` 参数配置

### X-Bogus 和 A_Bogus 算法

抖音接口需要 X-Bogus 和 A_Bogus 参数，相关实现位于：
- `/crawlers/douyin/web/xbogus.py`
- `/crawlers/douyin/web/abogus.py`

这些算法会自动生成，无需手动处理。

### 错误处理

爬虫层定义了统一的异常类 (`/crawlers/utils/api_exceptions.py`)：
- `APIConnectionError`: 连接错误
- `APITimeoutError`: 超时
- `APIResponseError`: 响应解析错误
- `APIRateLimitError`: 请求频率限制
- `APIUnauthorizedError`: 认证失败（通常是 Cookie 失效）

### 日志系统

使用 `/crawlers/utils/logger.py` 中的 logger：
```python
from crawlers.utils.logger import logger
logger.info("信息日志")
logger.error("错误日志")
```

## 添加新的 API 端点

1. 在 `/app/api/endpoints/` 创建新的端点文件
2. 定义 APIRouter 和路由函数
3. 在 `/app/api/router.py` 中注册新的路由
4. 如需新的爬虫功能，在 `/crawlers/` 对应平台目录下实现

## 测试 API

```bash
# 本地测试
curl "http://localhost/api/hybrid/video_data?url=https://v.douyin.com/xxxxx&minimal=false"

# 远程测试
curl "https://api.douyin.wtf/api/hybrid/video_data?url=VIDEO_URL&minimal=false"
```

## 相关资源

- GitHub: https://github.com/Evil0ctal/Douyin_TikTok_Download_API
- 在线文档: https://douyin.wtf/docs
- TikHub API (商业 API): https://api.tikhub.io/docs
- 视频教程: https://www.bilibili.com/video/BV1vE421j7NR/
