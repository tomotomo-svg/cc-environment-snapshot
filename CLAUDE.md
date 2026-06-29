# CLAUDE.md — cc-environment-snapshot

## Linksee Memory エンティティ設定

entity_name: `cc-environment-snapshot`

## プロジェクト概要

Claude Code 開発環境の棚卸しドキュメント生成・管理リポジトリ（2026-06-29〜）。
3 つの同時目的: ①DIG部門メンバーへの共有 ②自身の整理 ③環境スナップショット時点記録。

**成果物（main ブランチ）:**
- `current_development_environment.md` — Obsidian 取込み用 Markdown
- `current_development_environment.html` — DIG 部門配布用 HTML（スタンドアロン・オフライン閲覧可）

## 現在の状況（2026-06-29 完成・🟢 Sonnet）

- ✅ Markdown 版完成（10セクション・550行強）
- ✅ HTML 版完成（v4・約 1,800 行）
- ✅ SVG 詳細ノード図（案B 採用・案A は `svg-a-simple-archive` タグで保存）
- ✅ Section 4 MCP 23本に活用例・難易度・メリット詳細追加
- ✅ 本文・カード・テーブルのフォント全体的に拡大（body 17px / 見出し 24px / カード 16px 等）
- ✅ overview セクションは画面幅いっぱい（1600px max）表示
- ✅ マスキング確認（社内プロジェクト名・個人名・ドメイン除去済み・Skill名のみ残存）
- ✅ Obsidian `02_AREAS/Claude Code開発環境リファレンス_2026-06-29.md` に配置
- ✅ ナレッジ共有マップ更新済み

## GitHub Pages URL（iPad/iPhone 対応済み）

**公開URL**: https://tomotomo-svg.github.io/cc-environment-snapshot/

- DIG部門メンバーへはこの URL をそのまま共有してください（Safari / Chrome どちらでも閲覧可）
- iPad/iPhone 対応: viewport-fit=cover / safe-area / 768px ブレークポイント 追加済み（2026-06-30）
- リポジトリ: https://github.com/tomotomo-svg/cc-environment-snapshot（Public）

## 次のステップ

1. **URL を DIG 部門メンバーへ共有** → 🟢 Sonnet（上記 URL をそのまま送るだけ）
2. **環境変化時の差分更新** → 🟢 Sonnet（Skill/MCP 追加時に HTML を更新して `git push`）
3. **次回更新時の基準**: 「主要数字（MCP/Skill/Hook/Rule/CLI 本数）が変わったタイミング」または月1回程度

## ブランチ・タグ

- `main` — 確定版
- tag `svg-a-simple-archive` — 不採用の案A（シンプル同心円図）の履歴保存

## 情報源

- `~/PTC_WORK/Obsidian-Vault/02_AREAS/Mac開発環境インベントリ.md`
- `~/PTC_WORK/Obsidian-Vault/02_AREAS/Claude_ナレッジ共有マップ.md`

## 関連

- Obsidian 配置先: `02_AREAS/Claude Code開発環境リファレンス_2026-06-29.md`
- プラン: `~/.claude/plans/3-plugin-api-mcp-cli-curried-beaver.md`
