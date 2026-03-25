# CONSTITUTION

## 1. One Sentence

note.com/morodomi の記事を管理し、note-skills パイプラインで執筆からレビュー・公開まで一貫して品質を担保する。

## 2. Goal / Non-Goals

### Goals
- 記事の品質を一定水準以上に保つ（AI臭排除、推敲8観点レビュー）
- 執筆ワークフローの再現性を確保する（パイプライン化）
- PV・スキ数のトラッキングで記事パフォーマンスを可視化する

### Non-Goals
- ソフトウェア開発（コードベースのテスト・カバレッジ）
- SEO最適化の自動化
- 他プラットフォームへの同時配信

## 3. Human vs AI 責務

| 領域 | Human | AI |
|------|-------|----|
| テーマ選定 | 最終決定 | ネタ候補提案（note-seed） |
| 執筆 | 体験・意見の提供 | ドラフト作成支援（note-draft） |
| 推敲 | AI臭の最終チェック | 8観点並列レビュー（note-review） |
| 公開 | 公開判断 | 投稿自動化（note-publish） |
| PV分析 | 方針決定 | データ記録（note-track） |

## 4. Source of Truth (5-Layer)

| Layer | File | Role |
|-------|------|------|
| 0 | CONSTITUTION.md | Why（存在意義・原則） |
| 1 | AGENTS.md | What（技術スタック・ワークフロー） |
| 2 | CLAUDE.md | How（AI振る舞い・委譲ルール） |
| 3 | docs/ | Detail（調査・ネタ・サイクル） |
| 4 | 記事ファイル | Truth（記事本文が最終的な真実） |

## 5. 変更ポリシー

- CONSTITUTION.md の変更は Human の承認が必要
- 変更時は理由を Git コミットメッセージに記録する
- 他のレイヤーとの矛盾が生じた場合、上位レイヤーが優先
