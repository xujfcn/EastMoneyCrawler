# Apify Actor 部署指南

本文档介绍如何将东方财富爬虫部署到 Apify 平台。

## 前置准备

1. **Apify 账户**
   - 注册地址: https://console.apify.com/sign-up
   - 免费账户包含一定的运行配额

2. **GitHub 仓库**
   - 项目已推送到 GitHub: https://github.com/xujfcn/EastMoney_Crawler

## 部署步骤

### 方式一: 通过 Apify 控制台部署

1. **登录 Apify 控制台**
   - 访问: https://console.apify.com/
   - 使用账户登录

2. **创建新 Actor**
   - 点击左侧菜单 "Actors"
   - 点击 "Create new" 按钮
   - 选择 "From scratch"

3. **配置 Actor**
   - Name: `eastmoney-stock-forum-crawler`
   - Description: `东方财富股吧爬虫 - 爬取指定股票的帖子和评论数据`
   - Source type: 选择 "GitHub"
   - Repository: 输入 `xujfcn/EastMoney_Crawler`
   - Branch: `main`

4. **构建设置**
   - Apify 会自动检测到 `.actor/actor.json` 配置文件
   - 自动读取 `Dockerfile` 进行构建
   - 输入参数会根据 `INPUT_SCHEMA.json` 自动生成

5. **触发构建**
   - 点击 "Build" 标签
   - 点击 "Build Actor" 按钮
   - 等待构建完成（首次构建需要 5-10 分钟）

6. **运行 Actor**
   - 构建成功后，点击 "Input" 标签
   - 配置输入参数（或使用默认值）
   - 点击 "Start" 按钮运行

### 方式二: 使用 Apify CLI 部署

1. **安装 Apify CLI**
   ```bash
   npm install -g apify-cli
   ```

2. **登录 Apify**
   ```bash
   apify login
   ```
   - 按提示在浏览器中授权

3. **推送到 Apify**
   ```bash
   cd c:\Users\jianf\Desktop\scraper\EastMoney_Crawler
   apify push
   ```
   - 首次推送会创建新 Actor
   - 后续推送会更新现有 Actor

4. **在控制台查看**
   - 访问 https://console.apify.com/actors
   - 找到新创建的 Actor 并运行

## 输入参数配置

在 Apify 控制台的输入界面，可以配置以下参数：

### 基础参数

```json
{
  "stockCode": "002001",
  "stockName": "新和成",
  "maxPosts": 10,
  "crawlComments": false,
  "headless": true
}
```

### 参数说明

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| stockCode | string | 是 | "002001" | 股票代码（6位数字） |
| stockName | string | 是 | "新和成" | 股票名称 |
| maxPosts | integer | 否 | 10 | 最大爬取帖子数量（1-100） |
| crawlComments | boolean | 否 | false | 是否爬取帖子评论 |
| headless | boolean | 否 | true | 是否使用无头浏览器 |

### 使用代理（可选）

```json
{
  "stockCode": "002001",
  "stockName": "新和成",
  "maxPosts": 20,
  "crawlComments": true,
  "headless": true,
  "proxyConfiguration": {
    "useApifyProxy": true,
    "apifyProxyGroups": ["RESIDENTIAL"]
  }
}
```

## 输出数据

### 数据存储位置

- Actor 运行完成后，数据存储在 Apify Dataset 中
- 在运行详情页点击 "Dataset" 标签查看数据

### 数据导出格式

Apify 支持导出为多种格式：
- JSON
- CSV
- Excel
- HTML Table
- RSS

### API 访问

可以通过 Apify API 访问数据：

```bash
# 获取最新运行的数据集
curl "https://api.apify.com/v2/acts/YOUR_ACTOR_ID/runs/last/dataset/items?token=YOUR_API_TOKEN"
```

## 调度任务

### 设置定期运行

1. 在 Actor 详情页点击 "Schedules" 标签
2. 点击 "Create new schedule"
3. 配置运行频率：
   - Hourly（每小时）
   - Daily（每天）
   - Weekly（每周）
   - Monthly（每月）
   - Custom cron expression（自定义）

### Cron 表达式示例

```bash
# 每天早上 9 点运行
0 9 * * *

# 每周一和周五 下午 6 点运行
0 18 * * 1,5

# 每小时运行一次
0 * * * *
```

## 性能优化

### 内存配置

在 Actor 设置中可以调整内存分配：
- 最小: 512 MB
- 推荐: 1024 MB
- 最大: 2048 MB（需要更高套餐）

### 超时设置

```json
{
  "timeout": 3600
}
```
单位：秒，默认 3600 秒（1小时）

### 并发运行

- Apify 支持同时运行多个 Actor 实例
- 可以在 Schedules 中设置最大并发数

## 监控和日志

### 查看运行日志

1. 在 Actor 详情页点击 "Runs" 标签
2. 点击具体的运行记录
3. 查看 "Log" 标签的详细日志

### Webhook 通知

可以配置 Webhook 在运行完成时接收通知：

1. 在 "Settings" 中添加 Webhook URL
2. 选择触发事件：
   - Actor run succeeded
   - Actor run failed
   - Actor run aborted

## 成本估算

### 免费套餐

- 每月 5 美元的免费额度
- 基础爬取任务足够使用

### 付费套餐

- 根据运行时间和内存使用计费
- 估算：每次爬取 10 个帖子约需 0.01-0.05 美元

## 故障排查

### 常见问题

1. **构建失败**
   - 检查 Dockerfile 语法
   - 查看构建日志中的错误信息
   - 确认 requirements.txt 中的依赖版本

2. **运行超时**
   - 增加超时时间设置
   - 减少 maxPosts 数量
   - 检查目标网站是否可访问

3. **数据为空**
   - 检查目标网站是否更改了页面结构
   - 查看日志中是否有解析错误
   - 确认输入的股票代码是否正确

4. **IP 被限制**
   - 启用 Apify 代理
   - 降低爬取频率
   - 增加请求间隔时间

## 本地测试

在部署到 Apify 之前，建议先在本地测试：

```bash
# 使用 Apify CLI 本地运行
apify run

# 查看本地存储的数据
cat apify_storage/datasets/default/000000001.json
```

## 更新 Actor

### 通过 Git 自动更新

1. 推送代码到 GitHub
2. 在 Apify 控制台点击 "Build Actor"
3. 选择 "Build from Git"

### 手动上传代码

1. 在控制台选择 "Upload from local"
2. 上传项目 ZIP 文件
3. 触发构建

## 资源链接

- [Apify 文档](https://docs.apify.com/)
- [Apify Python SDK](https://docs.apify.com/sdk/python)
- [Playwright 文档](https://playwright.dev/python/)
- [Actor 示例](https://apify.com/store?type=acts)

## 技术支持

如遇到问题：
1. 查看 [GitHub Issues](https://github.com/xujfcn/EastMoney_Crawler/issues)
2. 参考 [Apify Community](https://community.apify.com/)
3. 查看项目 [README.md](README.md)
