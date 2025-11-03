# Apify Actor 版本核心文件说明

本文档详细说明 Apify Actor 版本（v4.0）的核心文件及其作用。

## 📦 Apify Actor 必需文件

### 1. `.actor/actor.json` - Actor 配置文件 ⭐⭐⭐⭐⭐

**作用**: Apify Actor 的元数据配置文件，定义 Actor 的基本信息和运行参数。

**位置**: `.actor/actor.json`

**内容说明**:
```json
{
  "actorSpecification": 1,           // 必需，固定为 1
  "name": "eastmoney-stock-forum-crawler",  // Actor 唯一标识
  "title": "EastMoney Stock Forum Crawler", // 显示名称
  "description": "A web scraper that crawls...",  // 功能描述
  "version": "1.0",                   // 版本号
  "buildTag": "latest",               // 构建标签
  "readme": "./README.md",            // README 文件路径
  "input": "./INPUT_SCHEMA.json",     // 输入参数定义文件
  "dockerfile": "./Dockerfile",       // Docker 构建文件
  "minMemoryMbytes": 512,            // 最小内存要求
  "maxMemoryMbytes": 2048            // 最大内存限制
}
```

**重要性**:
- ⭐⭐⭐⭐⭐ 必需文件
- Apify 平台通过此文件识别和配置 Actor
- 删除此文件，项目将无法作为 Actor 运行

---

### 2. `.actor/INPUT_SCHEMA.json` - 输入参数定义 ⭐⭐⭐⭐⭐

**作用**: 使用 JSON Schema 定义 Actor 的输入参数，Apify 会根据此文件自动生成输入表单。

**位置**: `.actor/INPUT_SCHEMA.json`

**内容说明**:
```json
{
  "title": "EastMoney Stock Forum Crawler Input",
  "type": "object",
  "schemaVersion": 1,
  "properties": {
    "stockCode": {
      "title": "Stock Code",        // 表单显示的标题
      "type": "string",              // 数据类型
      "description": "股票代码...",   // 帮助说明
      "default": "002001",           // 默认值
      "editor": "textfield"          // 输入控件类型
    },
    "maxPosts": {
      "type": "integer",
      "minimum": 1,                  // 最小值验证
      "maximum": 100,                // 最大值验证
      "default": 10
    },
    "crawlComments": {
      "type": "boolean",
      "editor": "checkbox"           // 复选框
    }
  },
  "required": ["stockCode", "stockName"]  // 必填字段
}
```

**重要性**:
- ⭐⭐⭐⭐⭐ 必需文件
- 定义用户输入参数和验证规则
- 自动生成美观的输入界面
- 提供输入验证和类型检查

---

### 3. `Dockerfile` - Docker 容器配置 ⭐⭐⭐⭐⭐

**作用**: 定义如何构建 Actor 的 Docker 镜像，包括环境、依赖和启动命令。

**位置**: `Dockerfile`

**内容说明**:
```dockerfile
# 基础镜像：Apify 官方 Python 3.11 镜像
FROM apify/actor-python:3.11

# 复制依赖文件
COPY requirements.txt ./

# 安装 Python 依赖
RUN pip install --no-cache-dir -r requirements.txt

# 安装 Playwright 和浏览器
RUN pip install playwright apify && \
    playwright install chromium && \
    playwright install-deps chromium

# 复制所有源代码
COPY . ./

# 启动命令：运行 main.py
CMD python3 -u main.py
```

**重要性**:
- ⭐⭐⭐⭐⭐ 必需文件
- 定义运行环境和依赖
- 确保在 Apify 平台和本地 Docker 环境一致
- 处理浏览器和系统依赖安装

---

### 4. `main.py` - Actor 主程序 ⭐⭐⭐⭐⭐

**作用**: Apify Actor 的入口文件，集成 Apify SDK 和爬虫逻辑。

**位置**: `main.py`

**核心功能**:
```python
from apify import Actor
from playwright.async_api import async_playwright

async def main() -> None:
    async with Actor:
        # 1. 获取输入参数
        actor_input = await Actor.get_input() or {}
        stock_code = actor_input.get('stockCode', '002001')

        # 2. 初始化 Playwright 浏览器
        async with async_playwright() as playwright:
            browser = await playwright.chromium.launch(...)

            # 3. 执行爬虫逻辑
            # ... 爬取数据 ...

            # 4. 推送数据到 Apify Dataset
            await Actor.push_data(posts_data)

        # 5. 输出日志和统计
        Actor.log.info("爬取完成！")

if __name__ == '__main__':
    asyncio.run(main())
```

**关键特性**:
- 使用 `Actor.get_input()` 获取用户输入
- 使用 `Actor.push_data()` 保存数据到 Dataset
- 使用 `Actor.log` 记录日志
- 异步编程提高性能

**重要性**:
- ⭐⭐⭐⭐⭐ 必需文件
- Actor 的核心业务逻辑
- 连接 Apify 平台和爬虫功能

---

### 5. `requirements.txt` - Python 依赖 ⭐⭐⭐⭐⭐

**作用**: 定义项目所需的 Python 包及版本。

**位置**: `requirements.txt`

**内容**:
```txt
apify>=2.0.0
playwright>=1.40.0
```

**说明**:
- `apify`: Apify Python SDK，提供 Actor 生命周期管理
- `playwright`: 浏览器自动化框架

**重要性**:
- ⭐⭐⭐⭐⭐ 必需文件
- Dockerfile 构建时安装依赖
- 确保运行环境正确

---

## 📄 辅助配置文件

### 6. `.dockerignore` - Docker 构建忽略 ⭐⭐⭐⭐

**作用**: 指定在构建 Docker 镜像时忽略的文件和目录。

**位置**: `.dockerignore`

**内容示例**:
```
# Git 文件
.git
.gitignore

# Python 缓存
__pycache__/
*.pyc

# 日志和输出
*.log
*.json
!INPUT_SCHEMA.json
!.actor/actor.json

# 开发工具
.vscode/
.idea/
```

**作用**:
- 减小 Docker 镜像大小
- 加快构建速度
- 避免敏感信息泄露

**重要性**:
- ⭐⭐⭐⭐ 推荐使用
- 优化构建性能
- 提高安全性

---

### 7. `.gitignore` - Git 版本控制忽略 ⭐⭐⭐⭐

**作用**: 指定 Git 版本控制应忽略的文件。

**重要性**:
- ⭐⭐⭐⭐ 推荐使用
- 避免提交临时文件、日志、输出数据
- 保持仓库整洁

---

## 📚 文档文件

### 8. `README.md` - 项目文档 ⭐⭐⭐⭐

**作用**: 项目说明文档，介绍功能、使用方法、部署步骤。

**Apify 相关内容**:
- 快速开始指南
- 输入参数说明
- 输出数据格式
- 部署到 Apify 的步骤

**重要性**:
- ⭐⭐⭐⭐ 强烈推荐
- 帮助用户快速上手
- Apify 平台会显示在 Actor 页面

---

### 9. `APIFY_DEPLOYMENT.md` - Apify 部署指南 ⭐⭐⭐

**作用**: 详细的 Apify 平台部署教程。

**内容包括**:
- 部署步骤（控制台 + CLI）
- 输入参数配置示例
- 调度任务设置
- 性能优化建议
- 故障排查

**重要性**:
- ⭐⭐⭐ 推荐
- 降低部署难度
- 提供完整的操作指南

---

## 🔧 旧版本文件（非 Apify 必需）

以下文件是项目历史版本的代码，**不是 Apify Actor 运行所必需的**：

### Selenium 版本（v2.0）
- `main_old.py` - Selenium 版本主程序
- `crawler.py` - Selenium 爬虫类
- `parser.py` - HTML 解析器
- `mongodb.py` - MongoDB 数据库接口
- `stealth.min.js` - 反检测脚本

### 独立 Playwright 版本（v3.0）
- `playwright_eastmoney_crawler.py` - 独立运行的 Playwright 爬虫

### 测试和工具文件
- `check_database.py` - 数据库检查工具
- `test_*.py` - 测试文件
- `*.json` - 爬取结果示例

**是否需要保留**:
- ✅ 可以保留作为参考和备用
- ❌ 删除不影响 Apify Actor 运行
- 💡 建议保留在单独的分支或目录

---

## 🎯 Apify Actor 最小文件集

如果要创建一个干净的 Apify Actor，**最少需要**以下 5 个文件：

```
EastMoneyCrawler/
├── .actor/
│   ├── actor.json              ⭐ 必需
│   └── INPUT_SCHEMA.json       ⭐ 必需
├── Dockerfile                  ⭐ 必需
├── main.py                     ⭐ 必需
├── requirements.txt            ⭐ 必需
└── README.md                   📄 推荐
```

---

## 📊 文件重要性总结

| 文件 | 重要性 | 必需 | 作用 |
|------|--------|------|------|
| `.actor/actor.json` | ⭐⭐⭐⭐⭐ | ✅ 是 | Actor 元数据配置 |
| `.actor/INPUT_SCHEMA.json` | ⭐⭐⭐⭐⭐ | ✅ 是 | 输入参数定义 |
| `Dockerfile` | ⭐⭐⭐⭐⭐ | ✅ 是 | Docker 镜像构建 |
| `main.py` | ⭐⭐⭐⭐⭐ | ✅ 是 | Actor 主程序 |
| `requirements.txt` | ⭐⭐⭐⭐⭐ | ✅ 是 | Python 依赖 |
| `.dockerignore` | ⭐⭐⭐⭐ | 📋 推荐 | 优化构建 |
| `README.md` | ⭐⭐⭐⭐ | 📋 推荐 | 项目文档 |
| `APIFY_DEPLOYMENT.md` | ⭐⭐⭐ | 📋 可选 | 部署指南 |
| 其他文件 | ⭐⭐ | ❌ 否 | 旧版本/测试 |

---

## 🔄 文件间的关系

```
用户在 Apify 控制台输入参数
          ↓
INPUT_SCHEMA.json (定义参数格式和验证)
          ↓
参数传递给 main.py
          ↓
Actor.get_input() 读取参数
          ↓
执行爬虫逻辑
          ↓
Actor.push_data() 保存数据
          ↓
数据存储到 Apify Dataset
          ↓
用户可导出为 JSON/CSV/Excel
```

**构建流程**:
```
GitHub 代码仓库
          ↓
Apify 读取 .actor/actor.json
          ↓
使用 Dockerfile 构建镜像
          ↓
安装 requirements.txt 中的依赖
          ↓
复制 main.py 和其他代码
          ↓
构建完成，准备运行
```

---

## 🚀 如何测试 Apify Actor

### 方法 1: 使用 Apify CLI（本地测试）
```bash
# 安装 Apify CLI
npm install -g apify-cli

# 在项目目录运行
apify run

# 查看输出数据
cat apify_storage/datasets/default/000000001.json
```

### 方法 2: 部署到 Apify 平台
1. 推送代码到 GitHub
2. 在 Apify 控制台创建 Actor
3. 连接 GitHub 仓库
4. 触发构建
5. 配置输入参数并运行

---

## 💡 常见问题

### Q1: 我可以删除旧版本的文件吗？
**A**: 可以。以下文件不影响 Apify Actor 运行：
- `main_old.py`
- `crawler.py`
- `parser.py`
- `mongodb.py`
- `playwright_eastmoney_crawler.py`
- 所有测试文件

### Q2: 为什么要用 Dockerfile？
**A**:
- 确保环境一致性
- 自动安装浏览器和依赖
- 支持在任何平台运行

### Q3: INPUT_SCHEMA.json 可以省略吗？
**A**: 技术上可以，但强烈不推荐：
- 没有它，用户需要手动编写 JSON
- 无法进行参数验证
- 用户体验差

### Q4: 如何在本地调试 main.py？
**A**: 可以不使用 Apify 环境：
```python
# 修改 main.py，添加本地调试代码
if __name__ == '__main__':
    # 模拟输入
    import os
    os.environ['ACTOR_INPUT'] = '{"stockCode": "002001", "stockName": "新和成"}'
    asyncio.run(main())
```

---

## 📖 相关资源

- [Apify Actor 文档](https://docs.apify.com/platform/actors)
- [Apify Python SDK](https://docs.apify.com/sdk/python)
- [JSON Schema 文档](https://json-schema.org/)
- [Dockerfile 最佳实践](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
