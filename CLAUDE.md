# CLAUDE.md — cc-environment-snapshot

## Linksee Memory エンティティ設定

entity_name: `cc-environment-snapshot`

## プロジェクト概要

Claude Code 開発環境の棚卸しドキュメント生成・管理リポジトリ（2026-06-29〜）。
3 つの同時目的: ①DIG部門メンバーへの共有 ②自身の整理 ③環境スナップショット時点記録。

**成果物（main ブランチ）:**
- `current_development_environment.md` — Obsidian 取込み用 Markdown
- `current_development_environment.html` — DIG 部門配布用 HTML（スタンドアロン・オフライン閲覧可）

## 現在の状況（最終更新：2026-06-30）

**cc-environment-snapshot 本体は変更なし（2026-06-29 完成済み）。**
2026-06-30 セッションは Gemini Enterprise のセットアップ作業に使用。

### Gemini Enterprise セットアップ 完了（2026-06-30）
- ✅ GCPプロジェクト `ptcg-gemini-enterprise` 作成・請求紐付け・Discovery Engine API 有効化
- ✅ Workforce Identity（Google Identity）設定 + ユーザー `nakano.tomonari@pritech.co.jp` 追加
- ✅ ウェブアプリ `gemini-enterprise-1782797168160` 作成・起動確認
- ✅ WebアプリURL: `https://vertexaisearch.cloud.google.com/home/cid/9126dcdd-7ccb-43c5-95a9-0bf2cd708590?hl=ja`
- ✅ モバイルURL: `https://vertexaisearch.cloud.google.com/mobile?cid=9126dcdd-7ccb-43c5-95a9-0bf2cd708590&cid_location=global`
- ⏳ `temporary-project/gemini-ent/` の CLAUDE.md・Git → **次セッション冒頭で作成**

### cc-environment-snapshot 本体（完成済み）
- ✅ Markdown 版完成（10セクション・550行強）
- ✅ HTML 版完成（v4・約 1,800 行）
- ✅ SVG 詳細ノード図（案B 採用・案A は `svg-a-simple-archive` タグで保存）
- ✅ Obsidian `02_AREAS/Claude Code開発環境リファレンス_2026-06-29.md` に配置
- ✅ ナレッジ共有マップ更新済み

## ⚡ 自動反映ルール（Skill/MCP/CLI/Hook/Rule 追加・変更時）

以下のいずれかに変化があったら、確認なしに一気に実行する：

**対象トリガー**
- Skill 追加・削除・変更
- MCP 追加・削除
- CLI ツール 追加
- Hook 追加・変更（`~/.claude/hooks/` 配下）
- Rule ファイル 追加・変更（`~/.claude/rules/` 配下）

**実行ステップ（順番通りに）**
1. **数字を更新** — ヒーロー stat-chip・SVGノード・Section見出しの本数を実際の数に合わせる
2. **HTMLにカテゴリ追加/更新** — 該当セクション（Section 3=Hook/Rule、Section 4=MCP、Section 5=Skill、Section 6=CLI）を追記・修正。日付 `🆕 YYYY-MM-DD` を付ける
3. **MDファイル更新** — `current_development_environment.md` に同内容を反映、`updated` を今日の日付に変更
4. **git push** — `main` へ push → GitHub Pages が自動デプロイ（約1分）
5. **Obsidian更新** — MDファイルを `02_AREAS/Claude Code開発環境リファレンス_2026-06-29.md` へ上書きコピー

**公開URL**: https://tomotomo-svg.github.io/cc-environment-snapshot/

- リポジトリ: https://github.com/tomotomo-svg/cc-environment-snapshot（Public）
- iPad/iPhone 対応: viewport-fit=cover / safe-area / 768px ブレークポイント 追加済み（2026-06-30）

## 次のステップ

1. **`gemini-ent` プロジェクトの CLAUDE.md・Git 作成** → 🟢 Sonnet（セッション冒頭で `/Users/c02-2405-01/PTC_WORK/AI_Development/ClaudeCode/temporary-project/gemini-ent/` に作成）
2. **Gemini Enterprise 残り9ライセンスの割り当て** → 🟢 Sonnet（AI Applications コンソール「ユーザーの管理」から追加）
3. **cc-environment-snapshot URL を DIG 部門メンバーへ共有** → 🟢 Sonnet（`https://tomotomo-svg.github.io/cc-environment-snapshot/`）
4. **環境変化時の差分更新** → 🟢 Sonnet（Skill/MCP 追加時に HTML を更新して `git push`）

## ブランチ・タグ

- `main` — 確定版
- tag `svg-a-simple-archive` — 不採用の案A（シンプル同心円図）の履歴保存

## 情報源

- `~/PTC_WORK/Obsidian-Vault/02_AREAS/Mac開発環境インベントリ.md`
- `~/PTC_WORK/Obsidian-Vault/02_AREAS/Claude_ナレッジ共有マップ.md`

## 関連

- Obsidian 配置先: `02_AREAS/Claude Code開発環境リファレンス_2026-06-29.md`
- プラン: `~/.claude/plans/3-plugin-api-mcp-cli-curried-beaver.md`
