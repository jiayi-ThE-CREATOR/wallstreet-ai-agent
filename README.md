# wallstreet-ai-agent

macOS 上定时抓取各大投行/独立研究机构【官方公开栏目】中与 AI 投资相关的文章，经 Claude 主题深读（中英混合）后通过 [Server酱](https://sct.ftqq.com/) 推送到微信。由 `launchd` 调度。

> 这是 [macos-llm-agents](https://github.com/jiayi-ThE-CREATOR/macos-llm-agents) 的独立拆分仓库，只含这一个 agent（附带独立运行所需的最小工具集），保留原仓库中该 agent 目录的完整提交历史。仓库内不含任何密钥——私人配置以 `.env.example` 模板入库，真身由 `.gitignore` 排除。

## 数据流

```
Google News site定向 RSS（官方域名，发现）
  → batchexecute 解码出原文真实 URL
  → Jina Reader（r.jina.ai）抓正文全文 → 剥离免责声明/导航
  → Claude（claude-sonnet-4-6）主题深读：论据 → 机制 → 分歧 → 启示
  → Server酱 → 微信
```

覆盖 12 家投行/研究机构（GS/MS/JPM/BofA/Citi/Wells Fargo/UBS/Barclays/Deutsche Bank/Jefferies/Wedbush/Bernstein），只抓官方域名内文章，不含媒体二手报道。详细的抓取链设计、易碎点、行为规范见 [agents/wallstreet_ai/CLAUDE.md](agents/wallstreet_ai/CLAUDE.md)。

## 定时任务

- 每周一、周五 08:00 JST 各推送一次
- 手动测试：`agents/wallstreet_ai/run.sh monday --dry-run`（只抓取整理打印，不推送、不写去重历史）

## 快速开始

```bash
# 1. 克隆
git clone https://github.com/jiayi-ThE-CREATOR/wallstreet-ai-agent.git && cd wallstreet-ai-agent

# 2. 配置密钥
cp .env.example .env
$EDITOR .env

# 3. 安装 Python 依赖
pip3 install -r agents/wallstreet_ai/requirements.txt

# 4. 安装定时任务到 launchd（自动把 plist 占位符替换为本机真实路径）
bash scripts/install_launchagents.sh

# 5. 手动跑一次验证（带 slot 名）
agents/wallstreet_ai/run.sh monday
```

需要的 API key 及获取地址见 [.env.example](.env.example)。Python 默认走系统 framework 解释器，可用环境变量覆盖：`PYTHON=/path/to/python3 agents/wallstreet_ai/run.sh monday`。

## 仓库结构

```
agents/wallstreet_ai/    # 抓取/深读/推送逻辑 + run.sh + plist
tools/load_env.sh        # 作用域密钥注入：只把 .env 里这个 agent 需要的 key 注入进程
scripts/                 # install_launchagents.sh 一键部署 launchd
.env.example              # 密钥模板
```

## 平台要求

- macOS（依赖 `launchd`）
- Python 3.14（或用 `PYTHON` 环境变量指向你的解释器）
- `ANTHROPIC_API_KEY` 需要有余额（Claude API 调用课金）

## License

[MIT](LICENSE)
