# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 提供在此代码库中工作的指导。

## 项目概览

这是一个实验性的 AI 交易系统，协调 48+ 个专业 AI 代理来分析市场、执行策略并管理加密货币市场（主要是 Solana）的风险。该项目采用模块化代理架构，支持统一的 LLM 提供商抽象，包括 Claude、GPT-4、DeepSeek、Groq、Gemini 和本地 Ollama 模型。

## 关键开发命令

### 环境设置
```bash
# 使用现有的 conda 环境（不要创建新的虚拟环境）
conda activate tflow

# 安装/更新依赖
pip install -r requirements.txt

# 重要：每次添加新包时都要更新 requirements.txt
pip freeze > requirements.txt
```

### 运行系统
```bash
# 运行主编排器（控制多个代理）
python src/main.py

# 独立运行单个代理
python src/agents/trading_agent.py
python src/agents/risk_agent.py
python src/agents/rbi_agent.py
python src/agents/chat_agent.py
# ... src/agents/ 中的任何代理都可以独立运行
```

### 回测
```bash
# 使用 backtesting.py 库，配合 pandas_ta 或 talib 作为指标
# 示例 OHLCV 数据可在以下位置获取：
# /Users/md/Dropbox/dev/github/moon-dev-ai-agents-for-trading/src/data/rbi/BTC-USD-15m.csv
```

## 架构概览

### 核心结构
```
src/
├── agents/              # 48+ 个专业 AI 代理（每个 <800 行）
├── models/              # LLM 提供商抽象（ModelFactory 模式）
├── strategies/          # 用户定义的交易策略
├── scripts/             # 独立实用脚本
├── data/                # 代理输出、内存、分析结果
├── config.py            # 全局配置（持仓、风险限制、API 设置）
├── main.py              # 多代理循环的主编排器
├── nice_funcs.py        # 约 1,200 行的共享交易工具
├── nice_funcs_hl.py     # Hyperliquid 专用工具
└── ezbot.py             # 旧版交易控制器
```

### 代理生态系统

**交易代理**: `trading_agent`、`strategy_agent`、`risk_agent`、`copybot_agent`
**市场分析**: `sentiment_agent`、`whale_agent`、`funding_agent`、`liquidation_agent`、`chartanalysis_agent`
**内容创作**: `chat_agent`、`clips_agent`、`tweet_agent`、`video_agent`、`phone_agent`
**策略开发**: `rbi_agent`（基于研究的推理 - 从视频/PDF 编写回测代码）、`research_agent`
**专业代理**: `sniper_agent`、`solana_agent`、`tx_agent`、`million_agent`、`tiktok_agent`、`compliance_agent`

每个代理都可以独立运行或作为主编排循环的一部分运行。

### LLM 集成（模型工厂）

位于 `src/models/model_factory.py` 和 `src/models/README.md`

**统一接口**: 所有代理使用 `ModelFactory.create_model()` 进行一致的 LLM 访问
**支持的提供商**: Anthropic Claude（默认）、OpenAI、DeepSeek、Groq、Google Gemini、Ollama（本地）
**关键模式**:
```python
from src.models.model_factory import ModelFactory

model = ModelFactory.create_model('anthropic')  # 或 'openai'、'deepseek'、'groq' 等
response = model.generate_response(system_prompt, user_content, temperature, max_tokens)
```

### 配置管理

**主配置**: `src/config.py`
- 交易设置: `MONITORED_TOKENS`、`EXCLUDED_TOKENS`、持仓大小（`usd_size`、`max_usd_order_size`）
- 风险管理: `CASH_PERCENTAGE`、`MAX_POSITION_PERCENTAGE`、`MAX_LOSS_USD`、`MAX_GAIN_USD`、`MINIMUM_BALANCE_USD`
- 代理行为: `SLEEP_BETWEEN_RUNS_MINUTES`、`main.py` 中的 `ACTIVE_AGENTS` 字典
- AI 设置: `AI_MODEL`、`AI_MAX_TOKENS`、`AI_TEMPERATURE`

**环境变量**: `.env`（参见 `.env_example`）
- 交易 API: `BIRDEYE_API_KEY`、`MOONDEV_API_KEY`、`COINGECKO_API_KEY`
- AI 服务: `ANTHROPIC_KEY`、`OPENAI_KEY`、`DEEPSEEK_KEY`、`GROQ_API_KEY`、`GEMINI_KEY`
- 区块链: `SOLANA_PRIVATE_KEY`、`HYPER_LIQUID_ETH_PRIVATE_KEY`、`RPC_ENDPOINT`

### 共享工具

**`src/nice_funcs.py`**（约 1,200 行）：核心交易函数
- 数据: `token_overview()`、`token_price()`、`get_position()`、`get_ohlcv_data()`
- 交易: `market_buy()`、`market_sell()`、`chunk_kill()`、`open_position()`
- 分析: 技术指标、盈亏计算、拉盘检测

**`src/agents/api.py`**: `MoonDevAPI` 类，用于自定义 Moon Dev API 端点
- `get_liquidation_data()`、`get_funding_data()`、`get_oi_data()`、`get_copybot_follow_list()`

### 数据流模式

```
配置/输入 → 代理初始化 → API 数据获取 → 数据解析 →
LLM 分析（通过 ModelFactory）→ 决策输出 →
结果存储（CSV/JSON 在 src/data/）→ 可选交易执行
```

## 开发规则

### 文件管理
- **保持文件在 800 行以下** - 如果更长，拆分为新文件并更新 README
- **未经询问不要移动文件** - 你可以创建新文件但不能移动
- **永远不要创建新的虚拟环境** - 使用现有的 `conda activate tflow`
- **添加任何新包后更新 requirements.txt**

### 回测
- 使用 `backtesting.py` 库（不要使用他们的内置指标）
- 使用 `pandas_ta` 或 `talib` 作为技术指标
- 示例数据位于 `/Users/md/Dropbox/dev/github/moon-dev-ai-agents-for-trading/src/data/rbi/BTC-USD-15m.csv`

### 代码风格
- **不要使用假数据/合成数据** - 始终使用真实数据或让脚本失败
- **最小化错误处理** - 用户希望看到错误，而不是过度工程化的 try/except 块
- **不要暴露 API 密钥** - 永远不要在输出中显示 `.env` 中的密钥

### 代理开发模式

创建新代理时：
1. 从现有代理的基础模式继承
2. 使用 `ModelFactory` 进行 LLM 访问
3. 将输出存储在 `src/data/[agent_name]/`
4. 使代理可独立执行（独立脚本）
5. 如需要，将配置添加到 `config.py`
6. 遵循命名规范：`[purpose]_agent.py`

### 测试策略

将策略定义放在 `src/strategies/` 文件夹中：
```python
class YourStrategy(BaseStrategy):
    name = "strategy_name"
    description = "what it does"

    def generate_signals(self, token_address, market_data):
        return {
            "action": "BUY"|"SELL"|"NOTHING",
            "confidence": 0-100,
            "reasoning": "explanation"
        }
```

## 重要背景

### 风险优先理念
- 风险代理在主循环中首先运行，早于任何交易决策
- 可配置的断路器（`MAX_LOSS_USD`、`MINIMUM_BALANCE_USD`）
- 平仓决策的 AI 确认（通过 `USE_AI_CONFIRMATION` 配置）

### 数据源
1. **BirdEye API** - Solana 代币数据（价格、交易量、流动性、OHLCV）
2. **Moon Dev API** - 自定义信号（清算、资金费率、持仓量、跟单数据）
3. **CoinGecko API** - 15,000+ 代币元数据、市值、情绪
4. **Helius RPC** - Solana 区块链交互

### 自主执行
- 主循环默认每 15 分钟运行一次（`SLEEP_BETWEEN_RUNS_MINUTES`）
- 代理优雅处理错误并继续执行
- 键盘中断实现优雅关闭
- 所有代理使用彩色编码输出记录到控制台（termcolor）

### AI 驱动的策略生成（RBI 代理）
1. 用户提供：YouTube 视频 URL / PDF / 交易想法文本
2. DeepSeek-R1 分析并提取策略逻辑
3. 生成与 backtesting.py 兼容的代码
4. 执行回测并返回性能指标
5. 成本：每次回测执行约 $0.027（约 6 分钟）

## 常用模式

### 添加新代理
1. 创建 `src/agents/your_agent.py`
2. 实现独立执行逻辑
3. 如需编排，添加到 `main.py` 中的 `ACTIVE_AGENTS`
4. 使用 `ModelFactory` 进行 LLM 调用
5. 将结果存储在 `src/data/your_agent/`

### 切换 AI 模型
编辑 `config.py`：
```python
AI_MODEL = "claude-3-haiku-20240307"  # 快速、便宜
# AI_MODEL = "claude-3-sonnet-20240229"  # 平衡
# AI_MODEL = "claude-3-opus-20240229"  # 最强大
```

或通过 ModelFactory 为每个代理使用不同的模型：
```python
model = ModelFactory.create_model('deepseek')  # 推理任务
model = ModelFactory.create_model('groq')      # 快速推理
```

### 读取市场数据
```python
from src.nice_funcs import token_overview, get_ohlcv_data, token_price

# 获取全面的代币数据
overview = token_overview(token_address)

# 获取价格历史
ohlcv = get_ohlcv_data(token_address, timeframe='1H', days_back=3)

# 获取当前价格
price = token_price(token_address)
```

## 项目理念

这是一个**实验性、教育性项目**，通过算法交易展示 AI 代理模式：
- 不保证盈利（有重大损失风险）
- 开源且免费供学习使用
- YouTube 驱动的开发，每周更新
- Discord 社区支持
- 项目无关联代币（避免诈骗）

目标是使 AI 代理开发民主化，展示可应用于交易之外的实用多代理编排模式。
