# note-skills 改善検討

## 背景

現行のnote-draft / note-reviewスキルを、novel-skillsやtdd-skillsの構造と比較し、改善方針を検討する。

## 現状分析: novel-skillsとの比較

### 構造比較

| 観点 | novel | note-draft | note-review |
|------|-------|-----------|------------|
| ディレクトリ構成 | creators/ + evaluators/ + references/ | references/ のみ | references/ のみ |
| フェーズ管理 | 7段階の明示的ワークフロー | 5段階（SKILL.mdに内包） | 4段階（SKILL.mdに内包） |
| サブスキル分離 | 創作5ファイル + 評価4ファイル | なし（1ファイル完結） | なし（review-criteria.mdに全集約） |
| コマンド体系 | `/novel concept` `/novel write` 等 | なし | なし |

### novelが勝っている点

1. **モジュール分離**: evaluatorが個別ファイルで独立。改善・差し替えが容易
2. **創作フェーズ別ガイド**: scene.md、character.md等がそれぞれ独立
3. **評価の独立性**: prose/consistency/pacing/ai-smellが個別ファイル

### note側が勝っている点

1. **philosophy.md（執筆思想）**: 「なぜそう書くか」の根底を持つ。novelにはない
2. **AI臭の対策が記事特化**: 事なかれ主義、抽象語等のリストが実用的
3. **並列レビュー設計**: 8サブエージェント並列が明文化されている

---

## AgentSkills横展開の視点

### 共通パターン

tdd-skills / redteam-skills / writing-skills / novel-skills で共通:
- フェーズ制ワークフロー
- 専門エージェント並列レビュー
- 品質ゲート

### TDD構造からの転用

```
SEED → RESEARCH → DRAFT → REVIEW → POLISH → PUBLISH
```

| フェーズ | やること | TDD対応 |
|---------|---------|---------|
| SEED | 日常の作業ログからネタ抽出 | INIT |
| RESEARCH | 裏付け調査、類似記事との差別化 | PLAN |
| DRAFT | 初稿生成 | RED+GREEN |
| REVIEW | 複数観点で並列レビュー | REVIEW |
| POLISH | トーン統一、タイトル最適化 | REFACTOR |
| PUBLISH | note.com用フォーマット出力 | COMMIT |

### レビューエージェント構成（quality-gate転用）

| エージェント | 観点 |
|-------------|------|
| fact-checker | 技術的正確性、バージョン情報の検証 |
| voice-reviewer | 「そのへんのエンジニア」トーンの維持 |
| seo-reviewer | タイトル、ハッシュタグ、冒頭3行の引き |
| uniqueness-reviewer | 既存記事との差別化、独自の視点があるか |

### SEED自動化の可能性

毎日のClaude Code作業ログ（Cycle doc等）を読み、「この体験は記事になる」と判断するエージェント。ネタ切れを防ぐ。

---

## 核心の課題

30記事で7フォロワー。投稿頻度は高い。問題は量ではなく刺さる記事を作れるか。

最も価値が高いのは:
- 「これは他の人も書けるクロード紹介記事になっていないか」
- 「morodomi固有の体験（AgentSkills開発、TDD強制、agent-memory設計）が入っているか」

をチェックする**uniqueness-reviewer**。

---

## 改善方針

### 優先度1: レビュー精度の向上（evaluator個別ファイル化）

現行の review-criteria.md（200行）を分割:

```
note-review/
  evaluators/
    accuracy.md       # 情報の正確性
    writing.md        # 文章の正確性
    structure.md      # 構成
    readability.md    # 読みやすさ
    engagement.md     # 面白さ
    title.md          # タイトル設計
    consistency.md    # 整合性
    ai-smell.md       # AI臭
    paywall.md        # 有料エリア（条件付き）
    uniqueness.md     # 独自性（新設）
```

効果: 各サブエージェントに渡す情報が明確になり、レビュー精度が上がる。

### 優先度2: uniqueness-reviewer 新設

チェック項目:
- morodomi固有の体験が入っているか
- 「Claude Code 便利です」的な汎用記事になっていないか
- 過去記事との差別化ができているか
- 読者が「この人の記事でしか読めない」と感じるか

参照: 過去記事一覧（CLAUDE.md）、philosophy.md

### 優先度3: 記事パターン別creatorファイル

```
note-draft/
  creators/
    experience.md    # 体験記パターン
    opinion.md       # 主張型パターン
    comparison.md    # 比較型パターン
    howto.md         # How-toパターン
```

パターンごとに冒頭の入り方、構成テンプレ、チェックリストを持たせる。

### 優先度4: ワークフローの明確化

SEED → RESEARCH → DRAFT → REVIEW → POLISH → PUBLISH の6フェーズ化。
コマンド体系: `/note-draft seed`, `/note-draft opinion` 等。

### 優先度5: SEEDの自動化

Cycle doc / 作業ログからのネタ抽出エージェント。
ただし、これは他の改善が安定してから。

### 後回し: novel-skills

- note.comは今すぐ実益がある（転職活動、ブランディング）
- 小説は要件が曖昧で、ワークフロー設計が難しい
- note-skillsで執筆ワークフローのパターンを確立してから応用する方が堅い

---

## 次のアクション

- [ ] evaluator個別ファイル化（review-criteria.md → evaluators/）
- [ ] uniqueness-reviewer の設計・実装
- [ ] SKILL.md のワークフロー更新（6フェーズ化）
- [ ] 記事パターン別creatorファイル作成
