# wallstreet-ai-agent

[简体中文](README.md) | **English** | [日本語](README.ja.md)

Periodically scrapes AI-investing articles from the **official public columns** of major investment banks and independent research shops on macOS, has Claude do a thematic deep-read (mixed Chinese/English), and pushes it to WeChat via [ServerChan](https://sct.ftqq.com/). Scheduled by `launchd`.

> This is an independent split-out of [macos-llm-agents](https://github.com/jiayi-ThE-CREATOR/macos-llm-agents), containing only this one agent (plus the minimal toolset it needs to run standalone), with the full commit history of that agent's directory in the original monorepo preserved. The repo contains no secrets — private config lives only as an `.env.example` template, with the real file excluded via `.gitignore`.

## Data flow

```
Google News site-targeted RSS (official domains, discovery)
  → batchexecute decode → real source URL
  → Jina Reader (r.jina.ai) fetches full body → strip disclaimers/nav
  → Claude (claude-sonnet-4-6) thematic deep-read: argument → mechanism → divergence → implication
  → ServerChan → WeChat
```

Covers 12 institutions (GS/MS/JPM/BofA/Citi/Wells Fargo/UBS/Barclays/Deutsche Bank/Jefferies/Wedbush/Bernstein), only pulling articles hosted on their official domains — no secondhand media coverage. Fetch-chain design, fragile points, and behavior spec: [agents/wallstreet_ai/CLAUDE.md](agents/wallstreet_ai/CLAUDE.md).

## Schedule

- Pushes once each Monday and Friday at 08:00 JST
- Manual test: `agents/wallstreet_ai/run.sh monday --dry-run` (fetches and prints only; no push, no dedup-history write)

## Quick start

```bash
# 1. Clone
git clone https://github.com/jiayi-ThE-CREATOR/wallstreet-ai-agent.git && cd wallstreet-ai-agent

# 2. Configure secrets
cp .env.example .env
$EDITOR .env

# 3. Install Python dependencies
pip3 install -r agents/wallstreet_ai/requirements.txt

# 4. Install the scheduled job into launchd (plist placeholders are rewritten to this machine's real path)
bash scripts/install_launchagents.sh

# 5. Run once to verify (pass the slot name)
agents/wallstreet_ai/run.sh monday
```

The required API keys and where to get them are listed in [.env.example](.env.example). Python defaults to the system framework interpreter; override it with an environment variable: `PYTHON=/path/to/python3 agents/wallstreet_ai/run.sh monday`.

## Repository layout

```
agents/wallstreet_ai/    # fetch/deep-read/push logic + run.sh + plist
tools/load_env.sh        # scoped secret injection: injects only the keys this agent needs from .env
scripts/                 # install_launchagents.sh, one-shot launchd deploy
.env.example              # secrets template
```

## Platform requirements

- macOS (depends on `launchd`)
- Python 3.14 (or point the `PYTHON` environment variable at your own interpreter)
- `ANTHROPIC_API_KEY` needs a funded balance (Claude API calls are billed)

## License

[MIT](LICENSE)
