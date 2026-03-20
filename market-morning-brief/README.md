# 三地股市智能晨报/晚报系统

> 开盘前30分钟推送 A股/港股/美股 第一性原则分析，收盘后复盘验证，全程数据可溯源。

## 功能概览

| 推送时间 (CST) | 内容 |
|---------------|------|
| 09:00 周一至五 | A股+港股 开盘前分析（包含美股隔夜数据） |
| 15:30 周一至五 | A股+港股 收盘复盘 + 明日建议 |
| 21:00 周一至五 | 美股 开盘前分析 + 当日亚洲盘回顾 |

## 快速开始（一键部署）

### 前置条件

1. **Claude API Key** → [申请地址](https://console.anthropic.com/settings/keys)（按用量付费）
2. **飞书机器人 Webhook** → 飞书群 > 设置 > 群机器人 > 添加机器人 > 自定义机器人（免费）
3. **Docker**（推荐）或 **Python 3.12+**

### Docker 部署（推荐）

```bash
# 1. 克隆仓库
git clone https://github.com/anthropics/financial-services-plugins.git
cd financial-services-plugins/market-morning-brief

# 2. 一键部署
bash deploy.sh

# 脚本会引导你配置 .env，然后自动构建并启动
```

### 手动 Docker 部署

```bash
# 1. 复制并编辑配置
cp .env.example .env
nano .env     # 填写 ANTHROPIC_API_KEY 和 FEISHU_WEBHOOK_URLS

# 2. 启动
docker-compose up -d

# 3. 测试飞书连接
docker-compose exec market-brief python src/main.py --test

# 4. 立即触发（测试）
docker-compose exec market-brief python src/main.py --now premarket_asia
```

### 本地 Python 部署

```bash
pip install -r requirements.txt
cp .env.example .env
# 编辑 .env 填写配置

# 测试
python src/main.py --test

# 启动定时调度
python src/main.py

# 或后台运行
nohup python src/main.py > cache/nohup.log 2>&1 &
```

## 配置说明

编辑 `.env` 文件（完整说明见 `.env.example`）：

```bash
# 必填
ANTHROPIC_API_KEY=sk-ant-api03-xxx   # Claude API Key
FEISHU_WEBHOOK_URLS=https://open.feishu.cn/...  # 飞书 Webhook

# 可选：关注的板块和自选股
FOCUS_SECTORS=科技,新能源,半导体,消费,医药
WATCHLIST=600519,00700,AAPL,NVDA

# 可选：调整推送时间（CST）
ASIA_PREMARKET_HOUR=9    # 亚洲开盘前
ASIA_POSTMARKET_HOUR=15  # 亚洲收盘后
US_PREMARKET_HOUR=21     # 美股开盘前（夏令时22，冬令时改为22）
```

### 美国夏令时调整

| 时期 | 美股开盘 CST | 建议设置 US_PREMARKET_HOUR |
|------|-------------|--------------------------|
| 夏令时（3月第2周 ~ 11月第1周） | 21:30 CST | 21 |
| 冬令时（其余时间） | 22:30 CST | 22 |

## 数据来源（全部免费/可溯源）

### 行情数据
| 来源 | 覆盖范围 | 接口 |
|------|---------|------|
| [AKShare](https://akshare.akfamily.xyz/) | A股/港股/美股 | Python 库，免费 |
| [Yahoo Finance](https://finance.yahoo.com/) | 全球股票/指数 | yfinance 库，免费 |
| [BaoStock](http://baostock.com/) | A股历史数据 | Python 库，免费注册 |
| [东方财富](https://www.eastmoney.com/) | A股实时行情 | 免费 JSON API |

### 新闻/政策
| 来源 | 内容 | 接入方式 |
|------|------|---------|
| [财联社](https://www.cls.cn/) | 实时财经电报 | API |
| [东方财富快讯](https://www.eastmoney.com/) | 7x24 快讯 | API |
| [新浪财经](https://finance.sina.com.cn/) | 宏观/A股新闻 | RSS |
| [新华社](http://www.xinhuanet.com/) | 政策解读 | RSS |
| [美联储](https://www.federalreserve.gov/) | 货币政策公告 | 官方 RSS |
| [中国人民银行](http://www.pbc.gov.cn/) | 货币政策公告 | 官网解析 |
| [中国证监会](http://www.csrc.gov.cn/) | 监管公告 | 官网解析 |
| [港交所 HKEX](https://www.hkex.com.hk/) | 市场公告 | 官网解析 |

### 研究报告
| 来源 | 内容 | 接入方式 |
|------|------|---------|
| [东方财富研报中心](https://data.eastmoney.com/report/) | 机构研报摘要 | 免费 API |

### 经济日历
| 来源 | 内容 | 接入方式 |
|------|------|---------|
| [Investing.com](https://cn.investing.com/economic-calendar/) | 全球经济数据日历 | API |
| [AKShare 宏观数据](https://akshare.akfamily.xyz/) | CPI/PPI/PMI 等 | Python 库 |
| [CNN Fear & Greed](https://edition.cnn.com/markets/fear-and-greed) | 市场情绪指数 | 免费 API |

## 分析框架：第一性原则

所有分析基于以下框架，拒绝主观预测：

```
市场价格 = f(盈利预期E, 无风险利率r, 流动性L, 风险情绪ρ)

对每个事件：
  1. 它改变了哪个变量？
  2. 改变方向（+/-）和幅度（高/中/低）？
  3. 传导链条：事件 → 变量 → 行业 → 个股
  4. 时间维度：短期(<1周) vs 中期(1-3月)
```

## 飞书推送示例

**开盘前报告卡片包含：**
- 📊 市场摘要（一句话）
- 🔔 关键事件（事件 + 影响变量 + 传导链 + 来源链接）
- 🗂️ 板块展望（涨跌方向 + 核心驱动）
- 🎯 重点标的表格（代码/名称/建议/目标价/逻辑/风险）
- 💡 整体操作建议
- ⚠️ 风险提示
- 📝 数据来源注脚

## 常用命令

```bash
# Docker 用户
docker-compose logs -f                                          # 查看实时日志
docker-compose exec market-brief python src/main.py --test     # 测试飞书
docker-compose exec market-brief python src/main.py --now premarket_asia  # 立即推送亚洲盘前
docker-compose exec market-brief python src/main.py --now postmarket_asia # 立即推送复盘
docker-compose exec market-brief python src/main.py --now premarket_us    # 立即推送美股盘前
docker-compose restart                                          # 重启服务
docker-compose down && docker-compose up -d                    # 重新部署

# 本地 Python 用户
python src/main.py --validate          # 验证配置
python src/main.py --test              # 测试飞书连接
python src/main.py --now premarket_asia
python src/main.py                     # 启动定时调度
```

## 目录结构

```
market-morning-brief/
├── .claude-plugin/plugin.json      # 插件元数据
├── commands/                       # Claude 插件命令
│   ├── premarket.md               # /market-morning-brief:premarket
│   └── postmarket.md              # /market-morning-brief:postmarket
├── skills/market-analysis/         # 技能知识库
│   └── SKILL.md                   # 第一性原则分析框架
├── src/
│   ├── main.py                    # 主入口（调度器 + CLI）
│   ├── config.py                  # 配置管理
│   ├── fetchers/
│   │   ├── news_fetcher.py        # 新闻/政策抓取（10个来源）
│   │   ├── market_data.py         # 行情数据（A股/港股/美股）
│   │   ├── research_fetcher.py    # 研报摘要（东方财富）
│   │   └── economic_calendar.py   # 经济日历（Investing.com等）
│   ├── analyzers/
│   │   └── claude_analyzer.py     # Claude API 分析引擎
│   └── notifiers/
│       └── feishu.py              # 飞书消息卡片推送
├── docker-compose.yml              # Docker 编排
├── Dockerfile                      # Docker 镜像
├── requirements.txt                # Python 依赖
├── .env.example                   # 配置模板
├── deploy.sh                      # 一键部署脚本
└── README.md                      # 本文档
```

## 常见问题

**Q: 不想用 Docker，可以直接用 Python 吗？**
A: 可以。`pip install -r requirements.txt && python src/main.py` 即可。

**Q: 数据抓取失败怎么办？**
A: 系统设计了多层降级：每个数据源失败会自动跳过，用其他来源补充，最终结果不影响 AI 分析质量。查看日志 `cache/market_brief.log` 了解详情。

**Q: 怎么设置多个飞书群？**
A: 在 `.env` 中用逗号分隔多个 Webhook URL：
```
FEISHU_WEBHOOK_URLS=https://open.feishu.cn/.../hook/xxx,https://open.feishu.cn/.../hook/yyy
```

**Q: 美国夏令时/冬令时怎么处理？**
A: 修改 `.env` 中的 `US_PREMARKET_HOUR`，夏令时设 21，冬令时设 22，然后 `docker-compose restart`。

**Q: 可以自定义关注的股票吗？**
A: 可以，在 `.env` 中设置 `WATCHLIST=600519,00700,AAPL,NVDA`（A股代码/港股代码/美股Ticker）。

**Q: 分析结果准确吗？**
A: 系统基于公开数据 + 第一性原则框架，分析质量与数据质量正相关。所有数据来源均注明，可独立核验。本工具仅供参考，不构成投资建议。

## 免责声明

本系统生成的分析报告基于公开可获取的数据，由 AI 模型（Claude）进行分析。所有分析仅供参考，不构成任何投资建议。投资决策需自行承担风险，建议在做出投资决定前咨询专业金融顾问。数据来源均已标注，但不对数据准确性作出保证。
