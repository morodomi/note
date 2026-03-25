# AST-only でテストとコードの対応関係を静的に解析する

## ネタ元

exspec observe PoC (Phase 8b, 2026-03-16)

## 概要

テストコードとプロダクションコードの対応関係 (test-to-code mapping) を、
実行なし・LLM なし・静的 AST 解析のみで高精度に実現した PoC の記録。

## 数字

- NestJS: Precision 99.4%, Recall 93.4%, F1 96.3% (GT完全監査)
- typeorm: Precision 100% (50ペア spot-check)
- 2リポジトリで外部妥当性確認済み

## 記事の切り口候補

1. **技術記事**: barrel import 解決の仕組み、tree-sitter クエリの設計、symbol-aware filtering
2. **ストーリー**: F1 72.8% → 96.3% の改善過程。wildcard 全展開で Precision 15% に壊れて直した話
3. **思想記事**: 「テストは仕様の実行可能な表現」という exspec の哲学と observe の位置づけ
4. **AI開発記事**: Claude Code + dev-crew で TDD サイクルを回した開発プロセス

## キーワード

静的解析, test-to-code mapping, barrel import, tree-sitter, TypeScript, TDD, exspec

## 競合/差別化

- 既存ツール: 全て動的計測 (Istanbul, nyc, jest --coverage)
- exspec observe: 実行不要、CI 不要、AST のみ
- 「テストカバレッジ」ではなく「テスト対応関係」という新しい観点
