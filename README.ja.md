# wallstreet-ai-agent

[简体中文](README.md) | [English](README.en.md) | **日本語**

macOS 上で各投資銀行・独立系リサーチ機関の【公式コラム】から AI 投資関連の記事を定期取得し、Claude によるテーマ別深読み（中英混在）を経て [Server酱（ServerChan）](https://sct.ftqq.com/) 経由で WeChat に配信します。`launchd` によってスケジューリングされます。

> これは [macos-llm-agents](https://github.com/jiayi-ThE-CREATOR/macos-llm-agents) から分割された独立リポジトリで、このエージェント 1 つだけを含みます（単独で動かすための最小ツールセット付き）。元のリポジトリでこのエージェントのディレクトリが持っていた完全なコミット履歴を保持しています。リポジトリにシークレットは一切含まれません。プライベートな設定は `.env.example` テンプレートとしてのみ入っており、実ファイルは `.gitignore` で除外しています。

## データフロー

```
Google News の site 指定 RSS（公式ドメイン、発見）
  → batchexecute でデコードして原文の実 URL を取得
  → Jina Reader（r.jina.ai）で本文全文を取得 → 免責事項/ナビを除去
  → Claude（claude-sonnet-4-6）によるテーマ別深読み：論拠 → メカニズム → 見解の相違 → 示唆
  → Server酱 → WeChat
```

12 の投資銀行・リサーチ機関（GS/MS/JPM/BofA/Citi/Wells Fargo/UBS/Barclays/Deutsche Bank/Jefferies/Wedbush/Bernstein）をカバーし、公式ドメイン内の記事のみを取得します（メディアの二次報道は含みません）。取得チェーンの設計・壊れやすいポイント・挙動仕様は [agents/wallstreet_ai/CLAUDE.md](agents/wallstreet_ai/CLAUDE.md) を参照。

## スケジュール

- 毎週月曜・金曜 08:00 JST にそれぞれ 1 回配信
- 手動テスト：`agents/wallstreet_ai/run.sh monday --dry-run`（取得・整理して表示のみ。配信せず、重複排除履歴も書き込まない）

## クイックスタート

```bash
# 1. クローン
git clone https://github.com/jiayi-ThE-CREATOR/wallstreet-ai-agent.git && cd wallstreet-ai-agent

# 2. シークレット設定
cp .env.example .env
$EDITOR .env

# 3. Python 依存関係のインストール
pip3 install -r agents/wallstreet_ai/requirements.txt

# 4. launchd へ定期ジョブをインストール（plist のプレースホルダは実機の実パスに置換される）
bash scripts/install_launchagents.sh

# 5. 動作確認に 1 回実行（slot 名を渡す）
agents/wallstreet_ai/run.sh monday
```

必要な API key と取得先は [.env.example](.env.example) を参照してください。Python は既定でシステムの framework インタプリタを使います。環境変数で上書きできます：`PYTHON=/path/to/python3 agents/wallstreet_ai/run.sh monday`。

## リポジトリ構成

```
agents/wallstreet_ai/    # 取得/深読み/配信ロジック + run.sh + plist
tools/load_env.sh        # スコープ付きシークレット注入：.env からこのエージェントに必要な key だけを注入
scripts/                 # install_launchagents.sh、ワンショット launchd 配備
.env.example              # シークレットのテンプレート
```

## 動作要件

- macOS（`launchd` に依存）
- Python 3.14（または `PYTHON` 環境変数で自分のインタプリタを指定）
- `ANTHROPIC_API_KEY` に残高が必要（Claude API 呼び出しは課金対象）

## ライセンス

[MIT](LICENSE)
