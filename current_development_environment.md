---
title: Claude Code 開発環境リファレンス 2026-06-29
date: 2026-06-29
updated: 2026-06-30
tags:
  - env
  - mcp
  - cli
  - skill
  - hook
  - ptc-knowledge
status: draft
cssclasses:
  - table-of-contents
---

# 🖥️ Claude Code 開発環境リファレンス 2026-06-29

> [!note] このドキュメントについて
> DIG部門メンバーへの共有・自身の環境整理・時点記録の3目的で作成。
> 社内プロジェクト名・顧客名・認証情報は**すべてマスク済み**。
> 元情報: [[Mac開発環境インベントリ]] / [[Claude_ナレッジ共有マップ]]

---

## 目次

- [[#Section 1 Claude Code とは何か]]
- [[#Section 2 数字で見る環境規模]]
- [[#Section 3 中心：Claude Code 本体]]
  - [[#設定ファイル群]]
  - [[#Hook（自動化トリガー）12本]]
- [[#Section 4 周辺：MCP 23本]]
  - [[#Google Workspace 系（3本）]]
  - [[#Google Cloud 系（5本）]]
  - [[#開発支援系（5本）]]
  - [[#ナレッジ・記憶系（3本）]]
  - [[#業務連携系（6本）]]
  - [[#Skill 実行支援（1本）]]
- [[#Section 5 拡張：Skill 165個]]
- [[#Section 6 土台：CLI ツール 26本+]]
- [[#Section 7 データ・連携の流れ]]
- [[#Section 8 認証・秘密情報の管理]]
- [[#Section 9 取り入れたい方への参考]]
- [[#Section 10 振り返り欄]]

---

## Section 1: Claude Code とは何か

> [!tip] 非エンジニアの方へ
> Claude Code は、Anthropic 社（ChatGPT の競合）が提供する **AI アシスタントをターミナル（黒い画面）で動かすツール** です。
> 普通の AI チャットと違い、「ファイルを読む・書く・コマンドを実行する・外部サービスを呼ぶ」が全部できます。

### なぜこんなに複雑な環境になっているのか

開発開始から約3ヶ月で、以下のように段階的に拡張してきました。

| 時期 | 追加したもの |
|---|---|
| 2026年3月〜 | Claude Code 本体・基本設定 |
| 2026年4月〜 | Google Workspace 連携（Drive/Gmail/Calendar） |
| 2026年5月〜 | GCP 連携・Skill 体系化・Hook 自動化 |
| 2026年6月〜 | 記憶系MCP（engram/Linksee）・KeyVault・セキュリティ強化 |

> [!warning] 複雑さの目的
> 「難しいことをしたい」のではなく、「**繰り返す作業をゼロにしたい**」が目的です。
> Skill・Hook・MCP は、毎回同じ指示をしなくて済むようにするための「定型化」です。

---

## Section 2: 数字で見る環境規模

| 要素 | 数 | 一言説明 |
|---|---|---|
| 🧩 MCP（外部接続） | **23本** | Google・GitHub・AI記憶・業務SaaS等への接続口 |
| 📋 Skill（自動化テンプレート） | **165個** | 「〇〇して」で起動する定型作業の型紙 |
| ⚡ Hook（自動トリガー） | **12本** | ツール使用のたびに裏で動く安全装置・自動記録 |
| 📜 Rule（ルールファイル） | **24本** | CC が必ず読む共通ルール集 |
| 🛠️ CLI ツール | **26本+** | ターミナルで使うコマンドラインツール |
| 💾 設定ファイルサイズ | **1.6 GB** | ~/.claude/ 配下の総容量 |

> [!note] 「1.6 GB」の正体
> 大部分は Skill ファイル群（165個×複数ファイル）と Claude Code のキャッシュです。
> 実際の設定ロジック（ルール・Hook 等）は数MB 程度です。

---

## Section 3: 中心：Claude Code 本体

Claude Code は `~/.claude/` 配下の設定ファイル群で動作をカスタマイズします。

### 設定ファイル群

<details>
<summary>📁 ~/.claude/ のフォルダ構成（クリックで展開）</summary>

```
~/.claude/
├── CLAUDE.md          ← 全プロジェクト共通の基本指示書（200行ガイドライン）
├── settings.json      ← CC本体の動作設定（モデル・権限・Hook登録）
├── rules/             ← 詳細ルール集（24ファイル）
│   ├── safety-checklist.md       ← 全作業前後の5項目チェック
│   ├── failure-learning.md       ← 失敗学習ログ（F-28まで蓄積）
│   ├── cc-automation-policy.md   ← 自動化最大化の判断基準
│   ├── keyvault-policy.md        ← 秘密情報の3層管理ルール
│   ├── session-lifecycle.md      ← セッション開始/終了/切替の手順
│   ├── obsidian-rules.md         ← Obsidian 保存・整形ルール
│   └── ... (他18ファイル)
├── skills/            ← Skill 格納場所（165個）
├── hooks/             ← Hook スクリプト（12本）
├── scripts/           ← ユーティリティスクリプト（90+本）
├── plans/             ← CC が作成した実装計画書
└── projects/          ← プロジェクト別のメモリキャッシュ
```

</details>

#### CLAUDE.md（全体指示書）

全プロジェクト共通の「CC への基本指示書」です。

- **役割**: 常に日本語で回答する / 非エンジニア向けの説明スタイル / 秘密情報の扱い方 / 失敗学習の参照方法 など
- **運用**: 200行ガイドラインで薄く保ち、詳細は `rules/` に切り出す
- **更新タイミング**: セッション終了時の引き継ぎ作業で自動更新

#### rules/（ルールファイル 24本）

<details>
<summary>📋 主要ルールファイル一覧（クリックで展開）</summary>

| ファイル | 役割 | 使用頻度 |
|---|---|---|
| `safety-checklist.md` | 削除禁止・変更前確認・秘密情報禁止 等5項目 | 🔥 常用 |
| `failure-learning.md` | 過去の失敗事例 F-28 件を蓄積・次回参照 | 🔥 常用 |
| `cc-automation-policy.md` | CC が自動化できる操作の判断基準 | 🔥 常用 |
| `keyvault-policy.md` | APIキー・パスワードの3層管理方針 | 🔥 常用 |
| `session-lifecycle.md` | セッション開始/終了/切替の標準手順 | 🔥 常用 |
| `obsidian-rules.md` | Obsidian へのファイル保存・整形ルール | ✅ 定期 |
| `dashboard-data-integrity.md` | 経営数字の正確性に関する最優先ルール | ✅ 定期 |
| `common-paths.md` | よく使うファイルパスの一覧 | ✅ 定期 |
| `google-chat-automation.md` | Google Chat への自動配信ガイド | ✅ 定期 |
| `second-brain-vault.md` | Obsidian Vault（第二の脳）連携ルール | ✅ 定期 |
| `handover-protocol.md` | 引き継ぎ漏れ防止プロトコル（3層冗長） | ✅ 定期 |
| `db-shared-infra.md` | 共有PostgreSQL 運用ルール | ✅ 定期 |
| `failure-learning-archive.md` | 失敗ログ F-01〜F-10（旧版アーカイブ） | 💤 保有 |
| `linksee-rules.md` | Linksee Memory セッション間記憶ルール | 🔥 常用 |
| `model-recommendation.md` | Opus/Sonnet 使い分け推奨ルール | ✅ 定期 |
| `sort-clippings.md` | Obsidian Clippings 自動分類ルール | ✅ 定期 |
| `youtube-knowledge-flow.md` | YouTube→Vault 取込みフロー | ✅ 定期 |
| `slide-presentation.md` | プレゼン資料自動作成ルール | 💤 保有 |
| `google-knowledge-rules.md` | Google Cloud 公式ナレッジ参照ルール | ✅ 定期 |
| `knowledge-map-rules.md` | ナレッジ共有マップ自動更新ルール | ✅ 定期 |
| `knowledge-base-sync.md` | 共通ナレッジベース運用ルール | ✅ 定期 |
| `skill-checker-rules.md` | GitHub Skill セキュリティスキャン | 💤 保有 |
| `dev-environment-tracking.md` | ツール追加時のインベントリ自動更新 | ✅ 定期 |
| `update-dev-tools.md` | 開発ツール一括更新ルール | 💤 保有 |

</details>

---

### Hook（自動化トリガー）12本

Hook とは「CC が特定の動作をするたびに自動実行されるスクリプト」です。
人間で言うと「クセ・習慣」に相当します。意識しなくても勝手に動きます。

#### Hook の種類（タイミング別）

| タイミング | 本数 | 説明 |
|---|---|---|
| SessionStart | 2本 | セッション開始時に自動実行 |
| PreToolUse | 2本 | ツール使用前に自動チェック |
| PostToolUse | 3本 | ツール使用後に自動記録 |
| SessionEnd | 4本 | セッション終了時に自動保存 |
| Notification | 1本 | CC が何か通知する時 |

<details>
<summary>⚡ 12本の Hook 詳細（クリックで展開）</summary>

| Hook 名 | タイミング | 役割 | 重要度 |
|---|---|---|---|
| `auto-sync-knowledge.sh` | SessionStart | 共通ナレッジベースを自動 git push | 🔥 常用 ⚠️ 代替不可 |
| `cleanup-image-cache.sh` | SessionStart | 14日以上前の画像キャッシュを自動削除 | ✅ 定期 🆓 単体完結 |
| `pre-bash-secret-guard.sh` | PreToolUse | 危険コマンド（秘密情報の標準出力）をブロック | 🔥 常用 ⚠️ 代替不可 🔗 依存あり |
| `pre-write-claude-md-path.sh` | PreToolUse | CLAUDE.md 書き込み前にパスを明示 | 🔥 常用 🔗 依存あり |
| `post-bash-secret-detect.sh` | PostToolUse | コマンド出力に秘密情報パターンを検知 | 🔥 常用 ⚠️ 代替不可 🔗 依存あり |
| `dev-environment-track.sh` | PostToolUse | ツールインストール時にインベントリ更新を促す | ✅ 定期 🆓 単体完結 |
| `knowledge-map-trigger.sh` | PostToolUse | Skill 追加等でナレッジマップ更新を促す | ✅ 定期 🆓 単体完結 |
| `session-end-summary.sh` | SessionEnd | セッション終了時の要約を自動生成 | 🔥 常用 🔗 依存あり |
| `session-end-linksee-trigger.sh` | SessionEnd | Linksee Memory への記録を促す | 🔥 常用 🔗 依存あり |
| `session-end-engram-save.sh` | SessionEnd | engram への記憶保存を促す | 🔥 常用 🔗 依存あり |
| `session-end-knowledge-map-check.sh` | SessionEnd | ナレッジマップ更新チェック | ✅ 定期 🔗 依存あり |
| `notification-complete.sh` | Notification | 作業完了通知をシステム通知に転送 | 💤 保有 🆓 単体完結 |

> [!tip] セキュリティ 3層防御
> `pre-bash-secret-guard.sh`（ブロック）→ `post-bash-secret-detect.sh`（検知）→ `safety-checklist.md`（ルール）の3層で、APIキー・パスワードの意図せぬ露出を防ぎます。
> 過去に F-24/F-26/F-27/F-28（DB パスワード漏れ 4件）の失敗から構築した防御です。

</details>

---

## Section 4: 周辺：MCP 23本

MCP（Model Context Protocol）は「CC が外部サービスに接続するための規格化されたプラグイン」です。
USB ポートに例えると、CC 本体が PC で、MCP が各種 USB デバイスに相当します。

### Google Workspace 系（3本）

| MCP 名 | 役割 | 使用頻度 |
|---|---|---|
| `claude_ai_Gmail` | Gmail の読み書き・ラベル管理・スレッド検索 | 🔥 常用 |
| `claude_ai_Google_Drive` | Drive のファイル作成・検索・共有設定 | 🔥 常用 |
| `claude_ai_Google_Calendar` | カレンダーの予定作成・一覧・空き時間提案 | ✅ 定期 |

<details>
<summary>📧 Gmail MCP の活用例</summary>

- 「特定の会社からのメールを検索して」→ スレッド一覧を取得して内容を要約
- 「このメールに返信の下書きを作って」→ Draft 作成まで CC が実行
- フィルタ・ラベルの一括設定（Gmail 移行作業で活用）

**難易度**: ★★☆（claude.ai の OAuth 認証が必要。設定済みなら即使用可能）
**メリット**: メール確認・返信の定型作業が CC との会話で完結

</details>

<details>
<summary>📁 Google Drive MCP の活用例</summary>

- 「先週作ったスプレッドシートを検索して」→ ファイルメタデータ取得
- 「このファイルを部署全員に共有して」→ 権限設定まで CC が実行
- ドキュメントの読み書き（Docs/Sheets の内容を CC が直接操作）

**難易度**: ★★☆（Gmail と同じ OAuth）
**メリット**: Drive 整理・共有設定の手動操作がゼロになる

</details>

### Google Cloud 系（5本）

| MCP 名 | 役割 | 使用頻度 |
|---|---|---|
| `gcloud-mcp` | gcloud コマンドを MCP 経由で実行 | 🔥 常用 |
| `firebase` | Firebase プロジェクト管理・デプロイ・ルール設定 | ✅ 定期 |
| `google-cloud-storage` | GCS バケット・オブジェクト操作 | ✅ 定期 |
| `google-cloud-observability` | Cloud Monitoring・Logging・Trace の閲覧 | ✅ 定期 |
| `google-devknowledge` | Google Cloud 公式ドキュメント検索 | ✅ 定期 |

<details>
<summary>☁️ gcloud-mcp の活用例</summary>

- GCP プロジェクト作成・IAM 権限付与・API 有効化
- Cloud Run へのデプロイ・サービス設定変更
- Cloud SQL インスタンスの操作

**難易度**: ★★★（GCP の基礎知識が必要。初期認証設定が必要）
**メリット**: GCP の操作を会話で実行。CLI コマンドを覚えなくてよい

</details>

### 開発支援系（5本）

| MCP 名 | 役割 | 使用頻度 |
|---|---|---|
| `filesystem` | ローカルファイルの読み書き・ディレクトリ操作（開放フォルダ限定） | 🔥 常用 |
| `git` | git コマンド（status/diff/commit/log 等）を MCP 経由で実行 | 🔥 常用 |
| `github` | GitHub リポジトリ・PR・Issue・コードサーチ | 🔥 常用 |
| `context7` | 有名ライブラリ・フレームワークの最新公式ドキュメント検索 | 🔥 常用 |
| `cloudrun` | Cloud Run サービスのデプロイ・ログ取得 | ✅ 定期 |

<details>
<summary>📄 filesystem MCP の活用例</summary>

- CC がローカルファイルを直接読み書きできる（開放したフォルダのみ）
- `~/PTC_WORK/AI_Development/` 以下のファイルに CC がアクセス可能
- Obsidian Vault の直接編集もこの MCP 経由

**難易度**: ★☆☆（設定済みであれば即使用可能）
**メリット**: 「ファイルの内容を貼り付けて」が不要になる。CC が直接ファイルを読める

</details>

<details>
<summary>📚 context7 MCP の活用例</summary>

- 「Next.js の最新の Image コンポーネントの使い方を調べて」
- 「Prisma 6 の connect 構文を確認して」
- 学習コストが高いライブラリの API を即座に正確に参照

**難易度**: ★☆☆（設定するだけ。追加費用なし）
**メリット**: CC の「古い知識」に頼らず最新ドキュメントで回答。コードのバグが減る

</details>

### ナレッジ・記憶系（3本）

| MCP 名 | 役割 | 使用頻度 |
|---|---|---|
| `engram` | セッションをまたいだ永続記憶（神経科学的な記憶モデル） | 🔥 常用 |
| `linksee` | プロジェクトごとの構造化記憶（goal/caveat/learning） | 🔥 常用 |
| `notebooklm-mcp` | Google NotebookLM の操作（ノートブック作成・質問） | ✅ 定期 |

<details>
<summary>🧠 engram MCP の活用例（最重要MCP）</summary>

CC はデフォルトでは会話が終わると記憶がリセットされます。engram はそれを解決します。

- **記憶の保存**: 「このプロジェクトの方針は〇〇」→ `remember` で保存
- **記憶の想起**: 次のセッション開始時に `recall` で前回の文脈を復元
- **記憶の定着**: 役立った記憶を `reinforce` で強化（使うほど思い出しやすくなる）
- **記憶の訂正**: 間違えた記憶を `correct` で上書き（消さずに「間違えた」と記録）

**難易度**: ★★☆（MCP の設定が必要。engram サーバーの起動が必要）
**メリット**: 「前回の話を毎回また説明する」が不要になる。CC が文脈を引き継ぐ

</details>

<details>
<summary>🔗 linksee MCP の活用例</summary>

プロジェクトごとに「現在の目標・注意事項・学んだこと」を構造化して保存します。

- `entity_name: cc-environment-snapshot` のように名前を固定して管理
- セッション開始時に `recall` → 前回のゴール・ハマりポイントを即把握
- `goal`: 今セッションで何を達成するか
- `caveat`: 注意すべきこと・やらかしたこと
- `learning`: 今回学んだ重要な判断・技術知識

**難易度**: ★★☆（engram と同様 MCP 設定が必要）
**メリット**: プロジェクト固有の文脈が消えない

</details>

### 業務連携系（6本）

| MCP 名 | 役割 | 使用頻度 |
|---|---|---|
| `keyvaultmcp` | APIキー・パスワードの安全な管理・API代行呼び出し | 🔥 常用 ⚠️ 代替不可 |
| `claude_ai_Supabase` | Supabase プロジェクト管理・SQL 実行・マイグレーション | ✅ 定期 |
| `claude_ai_Miro` | Miro ボードの作成・図形追加・コメント | 💤 保有 |
| `claude_ai_Box` | Box ファイルの操作・共有・メタデータ管理 | ✅ 定期 |
| `claude_ai_ptcg-master-mcp` | 自社開発の組織マスタ参照（人物・部門・顧客情報） | 🔥 常用 |
| `cloudrun` | Cloud Run サービスへのデプロイ・管理 | ✅ 定期 |

<details>
<summary>🔑 KeyVault MCP の活用例（セキュリティの要）</summary>

APIキーやパスワードを「CC に直接渡さず」に API を呼び出せる仕組みです。

**問題**: CC にAPIキーを渡すと会話履歴に平文で残る → 漏洩リスク
**解決**: KeyVault（ローカルアプリ）にキーを保管し、CC は `{{KV:token:ラベル名}}` というプレースホルダーだけを使う

実際の動作:
1. CC が「このAPIキーを使ってリクエストして」と指示
2. KeyVault が内部でプレースホルダーを実際のキーに置き換えてリクエスト送信
3. CC にはキーの実値が一切返ってこない

**登録済みサービス数**: 10+ サービス（勤怠SaaS・労務SaaS・ホスティングサービス等）
**難易度**: ★★★（ローカルアプリのインストール・MCP 登録が必要）
**メリット**: APIキーを「貼り付ける」操作が永遠に不要になる

</details>

### Skill 実行支援（1本）

| MCP 名 | 役割 | 使用頻度 |
|---|---|---|
| `clasp` | Google Apps Script の管理（プロジェクト作成・コード push/pull） | ✅ 定期 |

---

## Section 5: 拡張：Skill 165個

Skill は「よく使う作業の手順書」を CC に渡したものです。
`/skill-name` と打つだけで起動し、CC がその手順書に沿って動きます。

> [!tip] Skill の例え
> レシピ本に例えると、Skill = レシピ 1本です。
> 「肉じゃがを作って」と言うと、CC がレシピを読んで手順通りに実行します。

### カテゴリ別一覧

<details>
<summary>🏢 社内情報系（5個）― 会社情報の参照・問い合わせ</summary>

| Skill 名 | 説明 | 使用頻度 |
|---|---|---|
| `pritech-group` | グループ各社の会社概要・事業内容・住所・経営者情報の参照 | 🔥 常用 |
| `pritech-infra` | 自社インフラ（DNS・GWS・メール構成）の参照 | ✅ 定期 |
| `pritech-knowledge` | 共通ナレッジベースの管理・更新手順 | ✅ 定期 |
| `add-shared-db` | 共有 PostgreSQL へ新規 DB を追加する手順 | ✅ 定期 |
| `superset` | Superset IDE の操作ガイド（新規プロジェクト作成等） | ✅ 定期 |

</details>

<details>
<summary>🤖 Claude Code 運用系（10個）― CC の日常運用を自動化</summary>

| Skill 名 | 説明 | 使用頻度 |
|---|---|---|
| `new-project` | 新規プロジェクト立ち上げ（README・仕様書・作業記録を自動作成） | 🔥 常用 |
| `claude-code-handover` | セッション終了時の引き継ぎ（CLAUDE.md更新→git push→Obsidian記録） | 🔥 常用 |
| `quick-handover` | 軽量版引き継ぎ（CLAUDE.md更新+git pushのみ） | 🔥 常用 |
| `gas-clasp` | GAS 開発の標準手順（clasp 経由のコード管理） | ✅ 定期 |
| `claude-md-refactor` | CLAUDE.md が肥大化した時の rules/ 切り出し手順 | 💤 保有 |
| `linksee-memory` | Linksee Memory の操作手順（recall/remember 手順） | ✅ 定期 |
| `codex-rescue` | Codex（OpenAI の CLI AI）への引き継ぎ・レビュー依頼 | ✅ 定期 |
| `slide-deck-builder` | Markdown からプレゼン資料を生成 | 💤 保有 |
| `youtube-fetcher` | YouTube 動画の字幕取得→Vault 保存 | ✅ 定期 |
| `kb-query` | Obsidian Vault の検索・参照 | 🔥 常用 |

</details>

<details>
<summary>📊 Google Workspace 系（44個）― GWS 操作の自動化</summary>

gws CLI（Google Workspace CLI）を活用した自動化の Skill 群です。
Drive・Sheets・Gmail・Calendar・Chat・Forms・Slides・Docs・Admin 等の操作を CC から実行できます。

**代表的なもの:**

| Skill 名 | 説明 | 使用頻度 |
|---|---|---|
| `gws-sheets-create` | 新規スプレッドシート作成・共有設定まで CC が実行 | 🔥 常用 |
| `gws-drive-manage` | Drive ファイルの整理・共有・移動 | ✅ 定期 |
| `gws-gmail-filter` | Gmail フィルタ・ラベルの一括設定 | 💤 保有 |
| `gws-admin-directory` | GWS 組織のユーザー一覧・部署構成の参照 | ✅ 定期 |
| `gws-chat-send` | Google Chat への自動配信設定 | ✅ 定期 |

残り 39個も同様に「gws コマンドの定型手順」を Skill 化したものです。

**難易度**: ★★☆（gws CLI の認証設定が必要）
**メリット**: GWS の手動操作が CC との会話で完結

</details>

<details>
<summary>🧪 Recipe（定型作業）系（41個）― 繰り返し作業のパターン化</summary>

「〇〇するときはいつもこの手順」という繰り返し作業をパターン化したものです。

**代表的なもの:**

| Skill 名 | 説明 | 使用頻度 |
|---|---|---|
| `cloud-run-basics` | Cloud Run へのデプロイ標準手順 | 🔥 常用 |
| `cloud-sql-basics` | Cloud SQL 接続・マイグレーション手順 | 🔥 常用 |
| `google-cloud-recipe-auth` | GCP 認証（ADC・SA鍵・Workload Identity）の設定パターン | 🔥 常用 |
| `google-cloud-recipe-onboarding` | 新規 GCP プロジェクト立ち上げの標準手順 | ✅ 定期 |
| `bigquery-basics` | BigQuery クエリ・テーブル操作の標準手順 | 💤 保有 |
| `firebase-basics` | Firebase プロジェクト管理の標準手順 | 💤 保有 |
| `alloydb-basics` | AlloyDB 接続・管理の標準手順 | 💤 保有 |

**メリット**: 「初めてやる作業」でも Skill があれば手順を覚えていなくてよい

</details>

<details>
<summary>☁️ Google Cloud 系（32個）― GCP 専門知識のナレッジ化</summary>

Google Cloud の各サービスについて「公式ドキュメントの要点 + 実際の使い方」をまとめた Skill 群です。

| カテゴリ | 含まれる Skill 例 |
|---|---|
| コンピュート | Cloud Run / GKE / Cloud Functions |
| データベース | Cloud SQL / AlloyDB / Firestore |
| AI/ML | Gemini API / Vertex AI Agent Platform |
| セキュリティ | WAF / Secret Manager |
| 監視・最適化 | Cloud Monitoring / コスト最適化 / SLO設計 |
| ネットワーク | VPC / ファイアウォール / DNS |

</details>

<details>
<summary>📚 ナレッジ・記憶系（10個）― 知識の蓄積・整理</summary>

| Skill 名 | 説明 | 使用頻度 |
|---|---|---|
| `kb-query` | Obsidian Vault からの情報検索 | 🔥 常用 |
| `kb-ingest` | 外部コンテンツ（記事・動画等）を Vault に取り込む | 🔥 常用 |
| `kb-lint` | Vault ノートの品質チェック・整形 | ✅ 定期 |
| `kb-daily` | 日次の学習ノート作成 | 💤 保有 |
| `obsidian-markdown` | Obsidian Flavored Markdown への整形 | ✅ 定期 |
| `linksee-memory` | Linksee Memory の操作 | 🔥 常用 |
| `second-brain-vault` | Vault 全体の参照・構造把握 | ✅ 定期 |

</details>

<details>
<summary>🎨 デザイン・プレゼン系（3個）</summary>

| Skill 名 | 説明 | 使用頻度 |
|---|---|---|
| `slide-deck-builder` | Markdown + SLIDE パターンからプレゼン資料生成 | 💤 保有 |
| `obsidian_style_guide` | Obsidian ノートのあしらい統一ガイド | ✅ 定期 |
| `liquid-dom` | Liquid Glass UI（ガラス風UI）の組み込み手順 | 💤 保有 |

</details>

<details>
<summary>🔌 業務API連携系（4個）</summary>

| Skill 名 | 説明 | 使用頻度 |
|---|---|---|
| `jobcan-api` | 勤怠SaaS API の操作手順（打刻・集計） | ✅ 定期 |
| `smarthr-api` | 労務SaaS API の操作手順（従業員情報取得） | ✅ 定期 |
| `mf-expense-api` | 経費SaaS API の操作手順 | ✅ 定期 |
| `xserver-api` | ホスティングサービス API 操作手順 | 💤 保有 |

</details>

<details>
<summary>🤖 Google Agents CLI（ADK エージェント開発）（7本）🆕 2026-06-30</summary>

Google の ADK（Agent Development Kit）を使った AI エージェントのライフサイクル全体を管理するスキル群。
`npx skills add google/agents-cli` でインストール。Gemini Enterprise と組み合わせて使用する。

| Skill 名 | 説明 | 使用頻度 |
|---|---|---|
| `google-agents-cli-workflow` | ADK 開発ライフサイクル全体の進行管理（常時アクティブ・エントリーポイント） | 🔥 常用 |
| `google-agents-cli-adk-code` | ADK Python のコードパターン（エージェント定義・ツール・コールバック・状態管理） | 🔥 常用 |
| `google-agents-cli-scaffold` | プロジェクト雛形生成（`create` / `enhance` / `upgrade`） | ✅ 定期 |
| `google-agents-cli-eval` | エージェント評価（データセット生成・LLM-as-judge 採点・失敗分析・プロンプト自動最適化） | ✅ 定期 |
| `google-agents-cli-deploy` | Cloud Run / GKE / Agent Runtime へのデプロイ + CI/CD 構築 | ✅ 定期 |
| `google-agents-cli-publish` | Gemini Enterprise Agent Registry への登録 | 💤 保有 |
| `google-agents-cli-observability` | Cloud Trace / BigQuery Agent Analytics / 外部監視ツール連携 | 💤 保有 |

**前提**: Gemini Enterprise（2026-06-30 導入決定・13本）/ Python + uv / CLI 本体は `uvx google-agents-cli setup`

</details>

---

## Section 6: 土台：CLI ツール 26本+

CC が内部で呼び出す、または作業中に使うコマンドラインツールです。

### パッケージ管理系

| ツール | 説明 | 使用頻度 |
|---|---|---|
| `brew` (Homebrew) | macOS のパッケージ管理。「apt-get の Mac版」 | 🔥 常用 ⚠️ 代替不可 |
| `npm` | JavaScript のパッケージ管理 | 🔥 常用 |
| `pip3` | Python のパッケージ管理 | ✅ 定期 |
| `pipx` | Python CLI ツールの独立インストール | ✅ 定期 |

### AI・CC 系

| ツール | 説明 | 使用頻度 |
|---|---|---|
| `claude` (Claude Code) | CC 本体のコマンド | 🔥 常用 ⚠️ 代替不可 |
| `codex` | OpenAI の CLI AI（CC との2人体制でレビュー等） | ✅ 定期 |
| `agy` (Antigravity) | 別の AI IDE（Superset の基盤） | 🔥 常用 |
| `nlm` (NotebookLM CLI) | Google NotebookLM の CLI 操作ツール | ✅ 定期 |
| `agents-cli` | Google ADK エージェントのライフサイクル管理（scaffold / eval / deploy / publish） | 🆕 定期 |
| `ollama` | ローカル AI モデルの実行環境 | 💤 保有 |

### クラウド・インフラ系

| ツール | 説明 | 使用頻度 |
|---|---|---|
| `gcloud` | Google Cloud CLI（GCP のほぼ全操作） | 🔥 常用 ⚠️ 代替不可 |
| `gh` (GitHub CLI) | GitHub のリポジトリ・PR・Issue 操作 | 🔥 常用 |
| `firebase` | Firebase CLI（Hosting/Functions デプロイ等） | ✅ 定期 |
| `gws` (Google Workspace CLI) | Gmail/Drive/Sheets 等 95 API の CLI | 🔥 常用 |
| `wrangler` | Cloudflare Workers の CLI | 💤 保有 |
| `vercel` | Vercel デプロイの CLI | 💤 保有 |

### 開発・ユーティリティ系

| ツール | 説明 | 使用頻度 |
|---|---|---|
| `git` | バージョン管理の基本ツール | 🔥 常用 ⚠️ 代替不可 |
| `node` / `bun` | JavaScript/TypeScript の実行環境 | 🔥 常用 |
| `python3` | Python の実行環境 | ✅ 定期 |
| `jq` | JSON データの整形・抽出ツール | 🔥 常用 |
| `deno` | モダンな JavaScript 実行環境 | 💤 保有 |
| `skillspector` | Skill ファイルのセキュリティスキャンツール | 💤 保有 |
| `mkcert` | ローカル HTTPS 証明書の作成ツール | ✅ 定期 |
| `curl` | HTTP リクエストのコマンドラインツール | ✅ 定期 |

---

## Section 7: データ・連携の流れ

### 基本フロー

```
tomoさんの指示
      ↓
  Claude Code（CC）
      ↓
 ┌─────────────┐
 │  MCP 経由   │ → Google Drive・Gmail・GitHub・GCP 等
 │  CLI 経由   │ → gcloud・gh・gws・firebase 等
 │  Skill 参照 │ → 手順書に従って多段階で実行
 └─────────────┘
      ↓
  外部サービス / ローカルファイル / クラウドインフラ
```

### セッション間の記憶引き継ぎ（3層冗長）

セッションが終わっても次回に文脈を引き継ぐ仕組みです。

```
セッション終了時:
  1. CLAUDE.md 更新（現在の状況・次のステップを上書き）
  2. Linksee Memory に goal/caveat/learning を記録
  3. engram に「次のあなたへ」エピソードを記憶

次のセッション開始時（「続きをお願いします」で起動）:
  1. CLAUDE.md を読む → 現在の状況を把握
  2. Linksee recall → goal・caveat・learning を復元
  3. engram recall → エピソード記憶から前回の文脈を補完
```

> [!note] なぜ3層にするのか
> 1層（CLAUDE.md のみ）だと、更新漏れが起きた時に「前回何をしていたか」がわからなくなります。
> 3層あると、1つが欠けても他の2つで補完できます（2026-06-26 の引き継ぎ漏れ事故を受けて設計）。

### セキュリティ3層防御

```
危険なコマンド実行前:
  pre-bash-secret-guard.sh（ブロック）
      ↓ 通過したコマンドの実行
コマンド実行後:
  post-bash-secret-detect.sh（出力チェック・警告）
      ↓
作業を通じて:
  safety-checklist.md（ルールとして常時意識）
```

---

## Section 8: 認証・秘密情報の管理

APIキー・パスワード等の秘密情報は以下の3層で管理しています。

> [!warning] このドキュメントには秘密情報の値は一切記載していません
> 変数名・ラベル名のみ記載。実際の値は下記の各保管先にあります。

### 層1: KeyVault MCP（文字列の秘密情報）

Mac ローカルアプリ「KeyVault」に格納。CC が直接値を見ることなく API を呼び出せます。

**登録済みサービス（名称は一般名詞化）:**

| カテゴリ | 登録サービス数 | 主な内容 |
|---|---|---|
| 勤怠SaaS | 1 | クライアントID/シークレット × 5ペア |
| 労務SaaS | 4社分 | サブドメイン + アクセストークン |
| ホスティングサービス | 2環境 | APIキー + サーバー名 |
| 脳内DBサービス | 1 | DB 接続情報 5社分 |
| 本番DB直接接続 | 2 | DBパスワード（echo-server 経由でのみ利用） |
| 各種ダッシュボード | 3 | セッションシークレット / OAuthクライアント等 |
| 検索API | 1 | APIキー |
| Box / Microsoft 365 | 各1 | アクセストークン系 |

### 層2: Config/ フォルダ（ファイル形式の認証情報）

`~/PTC_WORK/AI_Development/Config/` 配下に格納。

| 内容 | 格納先パターン |
|---|---|
| GCP サービスアカウント鍵 JSON | `Config/gcp/{プロジェクト名}/sa-key.json` |
| OAuth クライアントシークレット JSON | `Config/GWS-CLI/client_secrets.json` |
| gws CLI 認証キャッシュ | `Config/GWS-CLI/` 配下のバイナリファイル |
| SSH 鍵 | `Config/xserver-*/ssh_key` |

### 層3: .env（非秘密の設定値）

各プロジェクトのリポジトリ直下の `.env` には**秘密度の低い設定値のみ**。

例: スプレッドシートID・URL・ポート番号・DB名・リージョン等

---

## Section 9: 取り入れたい方への参考

> [!tip] DIG部門メンバーへ
> 「全部入れないと意味がない」ではありません。1つだけ試すだけで十分です。

### 最初に入れるなら（3選）

#### 1. context7 MCP ★☆☆ 導入簡単

「Claude Code が古い知識で回答する」を解決します。

- **効果**: ライブラリの最新 API を正確に参照してコードを書いてくれる
- **必要なもの**: Claude Code のインストール + MCP 設定（5分）
- **費用**: 無料

#### 2. filesystem MCP ★☆☆ 導入簡単

「ファイルの内容をいちいちコピペして CC に渡す」を解決します。

- **効果**: CC が指定フォルダのファイルを直接読み書きできる
- **必要なもの**: MCP 設定（3分）+ 開放するフォルダの指定
- **費用**: 無料

#### 3. Hook 基本セット（セキュリティ3層防御） ★★☆ 少し設定が必要

APIキー等の意図せぬ漏洩を防ぐ安全装置です。

- **効果**: 危険なコマンドを自動でブロック・警告
- **必要なもの**: `~/.claude/hooks/` へのスクリプト設置
- **費用**: 無料

### 業務効率化なら（3選）

#### 1. gws CLI + Google Workspace Skill 群 ★★☆

Gmail・Drive・Sheets の手動操作をゼロにできます。

- **効果**: 「スプレッドシートを作って部署全員に共有して」が会話で完結
- **必要なもの**: gws CLI インストール + OAuth 認証

#### 2. engram + linksee MCP（セッション間記憶） ★★☆

毎回「前回の話を説明し直す」が不要になります。

- **効果**: CC が前回のプロジェクト文脈を引き継いで作業開始
- **必要なもの**: engram サーバー・linksee MCP の設定

#### 3. claude-code-handover Skill ★★☆

セッション終了時の引き継ぎ作業（CLAUDE.md更新・git push・Obsidian記録）を自動化します。

- **効果**: 「今日の作業ログ」を CC が自動でまとめてくれる

### セキュリティ重視なら（3選）

1. **KeyVault MCP**: APIキーを CC に渡さず API を呼び出す
2. **pre/post-bash Hook（3層防御）**: 秘密情報の誤った出力を自動でブロック
3. **safety-checklist.md の Rule 化**: 削除前確認・変更前説明を CC に習慣化

---

## Section 10: 振り返り欄（自身の整理用）

### 使用状況サマリ

| 状態 | 件数（目安） | 代表例 |
|---|---|---|
| 🔥 常用（毎セッション） | MCP 10本・Hook 8本・Skill 20個 | context7・filesystem・engram・linksee |
| ✅ 定期（週/月に使う） | MCP 8本・Hook 3本・Skill 80個 | firebase・GWS系・Recipe系 |
| 💤 保有のみ（最近未使用） | MCP 5本・Hook 1本・Skill 58個 | Miro・ollama・bigquery-basics 等 |

### 今後の検討リスト（書き込み欄）

- [ ] 最近使っていない MCP・Skill の整理・削除
- [ ] Hook の追加が必要な操作はないか
- [ ] 新たに Skill 化すべき繰り返し作業はないか
- [ ] KeyVault 未登録のサービスはないか

---

## 参考・関連リンク

- [[Mac開発環境インベントリ]]（バージョン・インストール手順の正本）
- [[Claude_ナレッジ共有マップ]]（Skill/Rule/Hook/MCP の運用設計図・entity_name 一覧）

---

*作成: 2026-06-29 | 情報源: Mac開発環境インベントリ + Claude ナレッジ共有マップ | 次回更新: 環境に大きな変更があったタイミング*
