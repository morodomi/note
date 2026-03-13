# N+1問題を8本のインデックスで解決した — コード変更なしのDB最適化

## ステータス: 執筆中（最優先）

## 記事の種

### 核となる知見

データ分析パイプラインで1レースあたり166クエリが走っていた。N+1問題の原因は各extractorが馬ごとにDBクエリを個別実行していたこと。解決は2段階:

1. インデックス5本追加（コード変更なし）
2. 一括プリロードで166クエリ → 5クエリ（97%削減）

「N+1 = ORMの問題」という思い込みへのカウンター。

### ソース

- Keiba/docs/cycles/20260210_1200_discovered-query-optimization.md
- Keiba/docs/cycles/20260210_1400_feature-calculator-n1-optimization.md

### 数字

- Before: 11クエリ/馬 x 15馬 = 166クエリ/レース
- After (Step 1): インデックス5本追加、コード変更なし
- After (Step 2): 5クエリ/レース（97%削減）
- N+1内訳: Horse 3, Performance 2, Jockey 3, Bloodline 1, Track 2 = 11クエリ/馬
- 1000レースで166,000 → 5,000クエリ

### 具体的なインデックス

```sql
-- FK列
CREATE INDEX IF NOT EXISTS idx_race_entries_race_id ON race_entries(race_id);
CREATE INDEX IF NOT EXISTS idx_race_entries_horse_id ON race_entries(horse_id);
CREATE INDEX IF NOT EXISTS idx_race_entries_jockey_id ON race_entries(jockey_id);
-- 日付範囲検索
CREATE INDEX IF NOT EXISTS idx_races_race_date ON races(race_date);
-- FK
CREATE INDEX IF NOT EXISTS idx_payoffs_race_id ON payoffs(race_id);
```

## 切り口案

1. 「N+1問題を8本のインデックスで解決した」（体験記 + How-to）
2. 「166クエリが5クエリになった。コード変更なしのDB最適化」（数字インパクト重視）
3. 「N+1はORMの問題じゃない」（主張型）

## 関連する既存記事

- なし（DB最適化系は初）

## メモ

- パターン: 体験記 + How-to
- ピックアップ狙い: 条件付き（画像不足）
- SEO: 「N+1問題」検索ボリュームあり、エバーグリーン
- 筆者メモ: 「書きやすそう」→ 最優先で執筆
- Keibaを「あるデータ分析パイプライン」に抽象化してOK
- 注意: タイトルの「8本」は5（インデックス）+ プリロード最適化を含む表現。正確には「インデックス5本 + 一括プリロード」
