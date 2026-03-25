# note

> docs are in migration. [CONSTITUTION.md](CONSTITUTION.md) is authoritative when other docs disagree.

## Start Here

1. [CONSTITUTION.md](CONSTITUTION.md) で存在意義・原則を確認
2. [docs/STATUS.md](docs/STATUS.md) で現在の状態を確認
3. [docs/cycles/](docs/cycles/) で進行中の作業を確認

## Overview

note.com/morodomi の記事管理プロジェクト。note-skills パイプラインで執筆からレビュー・公開まで一貫管理する。

### Tech Stack

- Content: Markdown
- Publishing: note-publisher (Playwright)
- Pipeline: note-skills (Claude Code plugin)
- Tracking: pv-data/ (JSON)

## Quick Commands

```bash
# 記事投稿（下書き）
note-publish post --file <mdファイルパス> --draft --verbose

# 記事投稿（公開）
note-publish post --file <mdファイルパス> --publish --verbose

# サムネイル生成
note-publish thumbnail --title "タイトル" --output images/thumb.png

# セッション更新
note-publish session --refresh --verbose
```

## TDD Workflow

```
spec → sync-plan → plan-review → RED → GREEN → REFACTOR → REVIEW → COMMIT
```

- spec: plan mode でスコープ定義
- sync-plan: plan を Cycle doc へ昇格
- plan-review: 設計検証（Codex 利用可能時は competitive review）
- RED: テスト作成 → 失敗確認
- GREEN: 最小限の実装 → テスト通過
- REFACTOR: テスト維持しつつコード品質改善
- REVIEW: 多角的コードレビュー

### Post-Approve Action (plan approve後)

1. sync-plan: plan fileからCycle doc生成
2. plan-review: 設計レビュー（Codex利用可能時はcompetitive review）
3. orchestrate: ゲート含む全サイクル実行 (RED → GREEN → REFACTOR → REVIEW → COMMIT)

## 執筆ルール

- 関連記事リンクはHTMLコメントではなく `<a>` タグで記述する（note-publisherがHTMLコメントを除去してからペーストするため）
- リンク形式: `<a href="URL">タイトル</a>`
- URL直書きは使わない（noteエディタで表示されない）

## Quality Standards

| 観点 | 基準 |
|------|------|
| AI臭 | 人間が最終チェック（noteアップ前） |
| 推敲 | note-review 8観点で並列レビュー |
| タイトル | 具体的、数字or固有名詞入り |
| 冒頭 | 3行以内で「何の話か」わかる |
| フォーマット | note-article-format.md 準拠 |

## Project Structure

```
note/
├── CONSTITUTION.md        # Why（存在意義・原則）
├── AGENTS.md              # What（技術スタック・ワークフロー）
├── CLAUDE.md              # How（AI振る舞い・委譲ルール）
├── YYYYMMDD_タイトル.md   # 執筆中・未公開の記事
├── archives/              # 公開済み記事
├── docs/
│   ├── STATUS.md          # プロジェクト状態
│   ├── cycles/            # 作業サイクル
│   ├── ネタ/              # ネタ候補ストック
│   └── *.md               # 調査・分析資料
├── images/                # サムネイル・記事用画像
└── pv-data/               # PV・スキ数データ (JSON)
```
