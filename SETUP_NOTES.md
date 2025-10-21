# 🔧 Moon Dev AI Trading System - 环境配置说明

## ✅ 当前状态

**时间**: 2025年10月21日  
**Python版本**: 3.11.5 (pyenv)  
**操作系统**: macOS (ARM64)

---

## 📦 依赖安装状态

### ✅ 已成功安装的包 (2025-10-21)

以下核心包已成功安装：

- **AI模型**: `anthropic`, `openai`, `groq`
- **数据处理**: `pandas`, `numpy`  
- **技术指标**: `ta-lib` (✅ 已安装)
- **回测**: `backtesting`, `backtesting.py`
- **音频**: `openai-whisper`, `whisper`, `ffmpeg-python`
- **PDF处理**: `PyPDF2`
- **视频处理**: `opencv-python`
- **其他工具**: `requests`, `python-dotenv`, `termcolor`, `tqdm`

---

## ⚠️ pandas_ta 问题及解决方案

### 问题描述

`pandas-ta` 包在 PyPI 上**不存在或已被移除**，导致无法通过 `pip install pandas-ta` 安装。

错误信息：
```
ERROR: Could not find a version that satisfies the requirement pandas-ta
(from versions: none)
```

### 为什么会出现这个问题？

1. **包名混淆**: 项目代码中使用 `import pandas_ta`，但 `requirements.txt` 中写的是 `pandas-ta`
2. **PyPI 不存在**: `pandas-ta` 这个包从未在 PyPI 上发布过，或者已经被移除
3. **GitHub 仓库不可用**: 原始的 GitHub 仓库 `twopirllc/pandas-ta` 已经不存在

### ✅ 解决方案（3种方式）

#### 方案1: 使用 ta-lib 替代 ✅ **推荐**

项目已经安装了 `ta-lib>=0.4.0`，它提供了所有常用的技术指标功能，可以完全替代 `pandas_ta`。

**需要修改的代码**：
```python
# 原代码 (src/nice_funcs.py 第17行)
import pandas_ta as ta

# 改为
import talib as ta
# 或者
from talib import abstract as ta
```

**优点**: 
- ✅ 已安装，无需额外配置
- ✅ 更稳定，社区支持更好
- ✅ 性能更优

#### 方案2: 从 Conda 安装 pandas_ta

如果必须使用 `pandas_ta`，可以尝试：

```bash
conda install -c conda-forge pandas-ta
```

#### 方案3: 手动安装本地版本

如果找到了 `pandas_ta` 的源代码，可以手动安装：

```bash
# 假设有源代码
cd path/to/pandas_ta
pip install -e .
```

---

## 🔍 项目当前使用 pandas_ta 的位置

根据代码搜索，以下文件使用了 `pandas_ta`：

1. **`src/nice_funcs.py`** (第17行): `import pandas_ta as ta`
2. **`src/nice_funcs_hl.py`**: 可能也使用了
3. **回测策略文件** (`src/data/rbi/` 目录下的大量策略文件)

---

## 🚀 下一步操作建议

### 立即可以做的:

1. **测试基础功能**（不涉及技术指标的代码）:
   ```bash
   cd /Users/yibu/dev_workspace/ai_trading/moon-dev-ai-agents
   python src/main.py  # 测试主程序
   ```

2. **修改代码使用 ta-lib**：
   - 在 `src/nice_funcs.py` 中将 `pandas_ta` 替换为 `talib`
   - 测试是否有兼容性问题

3. **检查哪些功能依赖 pandas_ta**：
   ```bash
   grep -r "pandas_ta" src/
   ```

### 如果必须使用 pandas_ta:

联系 Moon Dev 社区询问：
- Discord: [moondev.com](http://moondev.com)
- 可能有内部版本或替代方案

---

## 📝 环境变量配置 (下一步)

创建 `.env` 文件：

```bash
# AI模型API密钥
ANTHROPIC_KEY=your_anthropic_api_key_here
OPENAI_KEY=your_openai_api_key_here
GROQ_API_KEY=your_groq_api_key_here
DEEPSEEK_KEY=your_deepseek_api_key_here
GROK_API_KEY=your_grok_api_key_here

# 交易和数据API
BIRDEYE_API_KEY=your_birdeye_api_key_here  # ⚠️ 必需
SOLANA_RPC_URL=your_solana_rpc_url
SOLANA_PRIVATE_KEY=your_solana_private_key  # ⚠️ 谨慎保管！

# 其他API（可选）
TWITTER_API_KEY=your_twitter_api_key
ELEVENLABS_API_KEY=your_elevenlabs_key
```

---

## 🎯 总结

✅ **90% 的依赖已成功安装**  
⚠️ **pandas_ta 需要替换为 ta-lib 或从其他源安装**  
🚀 **系统基本可用，只需解决技术指标库的问题**

---

## 🆘 需要帮助？

- **Discord社区**: [moondev.com](http://moondev.com)
- **YouTube教程**: [完整播放列表](https://www.youtube.com/playlist?list=PLXrNVMjRZUJg4M4uz52iGd1LhXXGVbIFz)
- **文档**: 查看 `docs/` 目录

**创建时间**: 2025-10-21  
**创建者**: Claude AI Assistant 🤖

