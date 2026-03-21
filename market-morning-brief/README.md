# 三地股市智能晨报/晚报系统

> 基于 Claude 大模型第一性原则分析。开盘前30分钟自动推送 A股/港股/美股 深度分析到飞书，收盘后真正复盘验证建议准确性，全程数据可溯源。

## 功能概览

| 推送时间 (CST) | 内容 |
|---------------|------|
| 09:00 周一至五 | A股+港股 开盘前分析（包含美股隔夜数据） |
| 15:30 周一至五 | A股+港股 收盘复盘 + 对比早盘建议准确性 + 明日策略 |
| 21:00 周一至五 | 美股 开盘前分析 + 当日亚洲盘回顾 |

## 核心特点

- **Claude 大模型分析** — 基于第一性原则框架（E/r/L/ρ）进行真实语义理解，理解否定句、跨事件联动、复合影响
- **真实收盘复盘** — 对比早盘预测与实际走势，分析判断对错的原因，形成学习闭环
- **数据可溯源** — 每条新闻/研报标注来源和链接，结论有据可查
- **自动降级兜底** — 未配置 API Key 时自动使用规则引擎（关键词匹配，分析质量较低）

## 快速开始（一键部署）

### 前置条件

1. **Anthropic API Key** → [console.anthropic.com](https://console.anthropic.com/) 申请（强烈推荐，未配置则降级为规则引擎）
2. **飞书机器人 Webhook** → 飞书群 > 设置 > 群机器人 > 添加机器人 > 自定义机器人（免费）
3. **Docker**（推荐）或 **Python 3.12+**

### Docker 部署（推荐）

```bash
# 1. 克隆仓库
git clone https://github.com/straggler-liu/financial-services-plugins.git
cd financial-services-plugins/market-morning-brief

# 2. 一键部署
bash deploy.sh

# 脚本会引导你配置 .env（包括 ANTHROPIC_API_KEY），然后自动构建并启动
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
# 编辑 .env 填写 ANTHROPIC_API_KEY 和 FEISHU_WEBHOOK_URLS

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
# 强烈推荐：Claude 大模型分析（未配置则降级为规则引擎）
ANTHROPIC_API_KEY=sk-ant-...

# 必填：飞书 Webhook
FEISHU_WEBHOOK_URLS=https://open.feishu.cn/...

# 可选：关注的板块和自选股
FOCUS_SECTORS=科技,新能源,半导体,消费,医药
WATCHLIST=600519,00700,AAPL,NVDA

# 可选：调整推送时间（CST）
ASIA_PREMARKET_HOUR=9    # 亚洲开盘前
ASIA_POSTMARKET_HOUR=15  # 亚洲收盘后
US_PREMARKET_HOUR=21     # 美股开盘前（冬令时改为22）
```

### 美国夏令时调整

| 时期 | 美股开盘 CST | 建议设置 US_PREMARKET_HOUR |
|------|-------------|--------------------------|
| 夏令时（3月第2周 ~ 11月第1周） | 21:30 CST | 21 |
| 冬令时（其余时间） | 22:30 CST | 22 |

## 分析引擎：Claude 大模型

系统使用 Claude 对所有市场数据进行第一性原则深度分析，而不是简单的关键词匹配：

```
市场价格 = f(盈利预期E, 无风险利率r, 流动性L, 风险情绪ρ)

对每个新闻事件，Claude 会推理：
  1. 真实语义理解 → 正确处理否定句、同义词、上下文
  2. 影响变量判断 → 影响哪个变量（E/r/L/ρ），方向和幅度
  3. 传导链条推理 → 完整的逻辑路径（非模板拼接）
  4. 跨事件联动 → 综合多个事件的复合影响
  5. 个股映射 → 推理受影响个股并给出理由
```

**与规则引擎的关键差距：**

| 维度 | 规则引擎（降级模式） | Claude 大模型（主模式） |
|------|-----------------|-------------------|
| 收盘复盘 | 重复早盘策略，无真正对比 | 真正对比预期 vs 实际，分析对错原因 |
| 操作建议 | 信号计数后套模板句子 | 有逻辑推理的具体建议 |
| 新闻理解 | "不加息"误匹配为利空信号 | 正确理解语义和否定句 |
| 个股分析 | 无法输出受影响个股 | 独立推理并给出理由 |

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
│   └── SKILL.md                   # 第一性原则分析框架说明
├── src/
│   ├── main.py                    # 主入口（调度器 + CLI）
│   ├── config.py                  # 配置管理（含 ANTHROPIC_API_KEY）
│   ├── fetchers/
│   │   ├── news_fetcher.py        # 新闻/政策抓取（10个来源）
│   │   ├── market_data.py         # 行情数据（A股/港股/美股）
│   │   ├── research_fetcher.py    # 研报摘要（东方财富）
│   │   └── economic_calendar.py   # 经济日历（Investing.com等）
│   ├── analyzers/
│   │   ├── claude_analyzer.py     # Claude 大模型分析（主引擎）
│   │   └── rule_analyzer.py       # 规则引擎（未配置 API Key 时的降级兜底）
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

**Q: 没有 Anthropic API Key 可以运行吗？**
A: 可以，系统会自动降级为规则引擎（基于关键词匹配），但分析质量明显较低：收盘复盘无法真正对比早盘建议，操作建议为模板化输出，个股分析缺失。强烈建议配置 API Key 使用完整功能。

**Q: 数据抓取失败怎么办？**
A: 系统设计了多层降级：每个数据源失败会自动跳过，不影响整体分析。查看日志 `cache/market_brief.log` 了解详情。

**Q: 怎么设置多个飞书群？**
A: 在 `.env` 中用逗号分隔多个 Webhook URL：
```
FEISHU_WEBHOOK_URLS=https://open.feishu.cn/.../hook/xxx,https://open.feishu.cn/.../hook/yyy
```

**Q: 美国夏令时/冬令时怎么处理？**
A: 修改 `.env` 中的 `US_PREMARKET_HOUR`，夏令时设 21，冬令时设 22，然后 `docker-compose restart`。

**Q: 可以自定义关注的股票吗？**
A: 可以，在 `.env` 中设置 `WATCHLIST=600519,00700,AAPL,NVDA`（A股代码/港股代码/美股Ticker）。

**Q: Claude 每次分析大概消耗多少 token？**
A: 每次分析约 3,000~6,000 input tokens + 1,000~2,000 output tokens，使用 claude-sonnet-4-6 每次成本约 $0.02~0.05，每日3次约 $0.06~0.15。

## 免责声明

本系统生成的分析报告基于公开可获取的数据，由 Claude AI 辅助分析生成。所有分析仅供参考，不构成任何投资建议。投资决策需自行承担风险，建议在做出投资决定前咨询专业金融顾问。数据来源均已标注，但不对数据准确性作出保证。
