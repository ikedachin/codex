# Codex CLI — 日本語ガイド

<p align="center">
  <img src="https://github.com/openai/codex/blob/main/.github/codex-cli-splash.png" alt="Codex CLI" width="80%" />
</p>

**Codex CLI** は、OpenAI が提供するローカル動作型のコーディングエージェントです。  
ターミナルから自然言語でコードの作成・修正・説明を依頼でき、AI がファイルの読み書きやコマンドの実行まで自律的に行います。

---

## 目次

1. [Codex CLI とは](#codex-cli-とは)
2. [インストール方法](#インストール方法)
3. [認証（ログイン）](#認証ログイン)
4. [基本的な使い方](#基本的な使い方)
5. [非インタラクティブモード（自動化）](#非インタラクティブモード自動化)
6. [サンドボックス（安全な実行環境）](#サンドボックス安全な実行環境)
7. [設定ファイル](#設定ファイル)
8. [MCP（Model Context Protocol）連携](#mcpmodel-context-protocol連携)
9. [ソースからビルドする方法](#ソースからビルドする方法)
10. [リポジトリの構成](#リポジトリの構成)
11. [よくある質問](#よくある質問)

---

## Codex CLI とは

Codex CLI は **あなたのコンピュータ上で直接動作する** AI コーディングエージェントです。  
クラウド上のサービスにコードをアップロードするのではなく、ローカルのターミナルで AI と会話しながらコードを書いてもらうことができます。

### 他の Codex 製品との違い

| 製品 | 説明 |
|------|------|
| **Codex CLI（本リポジトリ）** | ターミナルで動くローカルエージェント |
| **Codex IDE 拡張** | VS Code / Cursor / Windsurf などのエディタ用。[インストールはこちら](https://developers.openai.com/codex/ide) |
| **Codex App** | `codex app` コマンドまたは [ChatGPT Codex App](https://chatgpt.com/codex?app-landing-page=true) |
| **Codex Web（クラウド版）** | [chatgpt.com/codex](https://chatgpt.com/codex) で使えるクラウド型エージェント |

---

## インストール方法

### Mac / Linux（推奨）

```shell
curl -fsSL https://chatgpt.com/codex/install.sh | sh
```

### Windows

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://chatgpt.com/codex/install.ps1 | iex"
```

> ⚠️ Windows は **WSL2 経由** での利用が推奨です（ネイティブ Windows でも動作します）。

### パッケージマネージャー経由

```shell
# npm を使う場合
npm install -g @openai/codex

# Homebrew を使う場合（Mac）
brew install --cask codex
```

### バイナリを直接ダウンロードする場合

[GitHub Releases](https://github.com/openai/codex/releases/latest) から OS・アーキテクチャに合ったファイルをダウンロードしてください。

| 環境 | ファイル名 |
|------|-----------|
| macOS (Apple Silicon) | `codex-aarch64-apple-darwin.tar.gz` |
| macOS (Intel) | `codex-x86_64-apple-darwin.tar.gz` |
| Linux (x86_64) | `codex-x86_64-unknown-linux-musl.tar.gz` |
| Linux (arm64) | `codex-aarch64-unknown-linux-musl.tar.gz` |

解凍後、実行ファイルを `codex` にリネームしてパスの通った場所に置いてください。

### システム要件

| 項目 | 要件 |
|------|------|
| OS | macOS 12 以上 / Ubuntu 20.04 以上 / Debian 10 以上 / Windows 11 (WSL2) |
| RAM | 最低 4GB（推奨 8GB） |
| Git（任意） | 2.23 以上（PR 補助機能を使う場合） |

---

## 認証（ログイン）

インストール後、`codex` コマンドを実行すると認証画面が表示されます。

### ChatGPT アカウントでログイン（推奨）

```shell
codex
```

起動後、**「Sign in with ChatGPT」** を選択します。  
ChatGPT Plus・Pro・Business・Edu・Enterprise プランに含まれる枠で利用できます。

> 詳細: [ChatGPT プランで Codex を使う](https://help.openai.com/en/articles/11369540-codex-in-chatgpt)

### API キーでログイン

OpenAI の API キーを持っている場合は、それを使って認証することもできます。

```shell
export OPENAI_API_KEY="sk-..."
codex
```

> 詳細: [API キー認証のセットアップ](https://developers.openai.com/codex/auth#sign-in-with-an-api-key)

---

## 基本的な使い方

### インタラクティブモード（通常の使い方）

```shell
codex
```

ターミナル上にフルスクリーンの TUI（テキストユーザーインターフェース）が起動します。  
チャット形式で AI に指示を出すと、ファイルの作成・編集・コマンド実行などを行ってくれます。

**使用例：**

```
> このコードベースを説明してください
> バグを修正してください
> テストを追加してください
> README を日本語に翻訳してください
```

### スラッシュコマンド

TUI 内でスラッシュから始まるコマンドを使えます（例：`/help`）。  
詳細は [スラッシュコマンド一覧](https://developers.openai.com/codex/cli/slash-commands) を参照してください。

### ログの確認

デバッグ用に詳細ログを記録したい場合：

```shell
codex -c log_dir=./.codex-log
tail -F ./.codex-log/codex-tui.log
```

---

## 非インタラクティブモード（自動化）

CI/CD やスクリプトからワンショットで実行したい場合は `codex exec` を使います。

```shell
# プロンプトを直接指定
codex exec "このコードにバグがあれば修正してください"

# 標準入力からプロンプトを渡す
echo "このエラーを解析してください" | codex exec

# 両方組み合わせる（stdin は <stdin> ブロックとして追加される）
cat error.log | codex exec "このエラーを要約してください"

# セッションファイルを残さずに実行（一時実行）
codex exec --ephemeral "ユニットテストを追加してください"
```

---

## サンドボックス（安全な実行環境）

Codex CLI は AI がコマンドを実行する際に、**サンドボックス**と呼ばれる安全な実行環境を使用します。  
これにより、意図しないファイルの削除やネットワークアクセスを防ぎます。

### サンドボックスポリシーの選択

```shell
# デフォルト：読み取り専用（書き込み・ネットワークアクセスは承認が必要）
codex --sandbox read-only

# ワークスペース内への書き込みを許可（ネットワークはブロック）
codex --sandbox workspace-write

# ⚠️ 危険：サンドボックスを無効化（コンテナ等の隔離環境でのみ使用）
codex --sandbox danger-full-access
```

### サンドボックスの動作を確認する

特定のコマンドがサンドボックスでどう動くか試したい場合：

```shell
# 現在の OS のサンドボックス実装で実行
codex sandbox ls -la

# macOS のみ：拒否されたアクセスをログに記録
codex sandbox --log-denials ls -la
```

OS ごとのサンドボックス実装：

- **macOS**: Seatbelt (`sandbox-exec`)
- **Linux**: Linux サンドボックス (Landlock / seccomp)
- **Windows**: 制限付きトークン

---

## 設定ファイル

Codex CLI は `~/.codex/config.toml` に設定ファイルを置きます。

### 設定例

```toml
# 使用するモデルを指定
model = "codex-mini-latest"

# サンドボックスポリシー
sandbox_policy = "workspace-write"

# ログ出力先
log_dir = "~/.codex/logs"

# 通知スクリプト（ターン完了時に実行）
[notify]
script = "terminal-notifier -message '{{message}}'"
```

### 設定ファイルのドキュメント

- [基本設定](https://developers.openai.com/codex/config-basic)
- [高度な設定](https://developers.openai.com/codex/config-advanced)
- [設定リファレンス（全項目）](https://developers.openai.com/codex/config-reference)
- [設定サンプル](https://developers.openai.com/codex/config-sample)

---

## MCP（Model Context Protocol）連携

Codex CLI は **MCP クライアント**として、外部の MCP サーバーに接続できます。  
また、Codex 自体を **MCP サーバー**として起動し、他のエージェントから呼び出すこともできます。

### MCP サーバーとして起動

```shell
codex mcp-server
```

### MCP インスペクターで動作確認

```shell
npx @modelcontextprotocol/inspector codex mcp-server
```

### MCP の管理コマンド

```shell
codex mcp add     # MCP サーバーを追加
codex mcp list    # 登録済みサーバーを一覧表示
codex mcp get     # サーバーの詳細を表示
codex mcp remove  # サーバーを削除
```

---

## ソースからビルドする方法

開発者向け：Rust のソースコードからビルドする手順です。

```bash
# リポジトリをクローン
git clone https://github.com/openai/codex.git
cd codex/codex-rs

# Rust ツールチェーンをインストール（未インストールの場合）
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source "$HOME/.cargo/env"
rustup component add rustfmt clippy

# ビルドに必要なツールをインストール
cargo install --locked just
cargo install --locked cargo-nextest

# ビルド
cargo build

# 動作確認
cargo run --bin codex -- "このコードベースを説明してください"
```

### よく使う開発コマンド

```bash
# コードの整形
just fmt

# Clippy（linter）の実行
just fix -p <変更したクレート名>

# テストの実行（特定クレートのみ）
just test -p codex-tui

# テスト全体の実行
just test
```

---

## リポジトリの構成

```
codex/
├── codex-rs/          # Rust 実装（メインの CLI）
│   ├── core/          # コアビジネスロジック
│   ├── tui/           # フルスクリーン TUI（Ratatui 使用）
│   ├── exec/          # 非インタラクティブ CLI
│   ├── cli/           # サブコマンドを束ねる CLI エントリポイント
│   ├── app-server/    # アプリサーバー（デスクトップアプリ向け）
│   └── ...            # その他多数のクレート
├── codex-cli/         # npm パッケージのラッパースクリプト
├── docs/              # ドキュメント（各機能の詳細）
├── sdk/               # SDK
└── README.md          # 英語版 README
```

### 主要クレートの役割

| クレート | 役割 |
|---------|------|
| `codex-core` | エージェントのコアロジック・API 通信 |
| `codex-tui` | ターミナル UI（Ratatui ベース） |
| `codex-exec` | `codex exec` コマンドの実装 |
| `codex-cli` | `codex` コマンドのエントリポイント |
| `codex-mcp` | MCP クライアント・サーバー機能 |
| `codex-config` | 設定ファイルの読み書き |
| `codex-sandboxing` | OS ごとのサンドボックス実装 |

---

## よくある質問

### Q. ChatGPT のアカウントがなくても使えますか？

A. OpenAI の API キーがあれば使えます。ただし API 利用料金が別途発生します。

### Q. コードをクラウドに送信されますか？

A. 自然言語の指示とコードの一部は OpenAI のサーバーに送信されます（AI の推論に必要なため）。ローカル実行はあくまで **操作（ファイル編集・コマンド実行）** がローカルで行われるという意味です。

### Q. どのモデルを使っていますか？

A. デフォルトでは `codex-mini-latest` モデルを使用します。設定ファイルの `model` で変更できます。

### Q. Windows でも使えますか？

A. はい。WSL2 経由での利用が推奨ですが、Windows ネイティブでも動作します。

### Q. 勝手にファイルを消されたり、変な操作をされたりしませんか？

A. サンドボックス機能により、デフォルトでは書き込みやネットワークアクセスに承認が必要です。不安な場合は `read-only` ポリシーを使うと、変更前に必ず確認が入ります。

---

## ドキュメント・リンク集

- 📖 [公式ドキュメント（英語）](https://developers.openai.com/codex)
- 🤝 [コントリビューション方法](./docs/contributing.md)
- 🔐 [セキュリティポリシー](./SECURITY.md)
- 📦 [リリースノート](https://github.com/openai/codex/releases)
- 💰 [オープンソースファンド](./docs/open-source-fund.md)

---

このリポジトリは [Apache-2.0 ライセンス](LICENSE) のもとで公開されています。
