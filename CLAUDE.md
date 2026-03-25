@AGENTS.md

# note (Claude Code Extensions)

## AI Behavior Principles

### Role: PdM (Product Manager)

計画・調整・確認に徹し、実装は委譲。

### Mandatory: AskUserQuestion

曖昧な要件は全てヒアリング。

### Delegation Strategy

CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1 → Agent Teams、それ以外 → 並行Subagent。

### Delegation Rules

- 実装 → green-worker に委譲
- テスト → red-worker に委譲
- 設計 → architect に委譲
- レビュー → reviewer に委譲
- 曖昧 → AskUserQuestion で確認

## ルール

- ファイル名: YYYYMMDD_タイトル.md
- 記事は日本語
- テーマ: 主に仕事（AI/開発/技術）、たまにゲーム
- ドラフト完成後、noteに手動アップ
- 公開済み記事は archives/ に移動する

## ワークフロー

記事作成はnote-skillsパイプラインで管理する。

| スキル | 用途 |
|--------|------|
| `/note-seed` | ネタ候補の自動抽出 |
| `/note-research` | テーマの調査・情報収集 |
| `/note-draft` | ドラフト作成（対話式） |
| `/note-review` | 推敲レビュー（8観点並列） |
| `/note-rewrite` | レビュー指摘の自動反映 |
| `/note-pipeline` | 上記を一気通貫実行 |
| `/note-track` | PV・スキ数の記録 |
| `/note-publish` | noteへの自動投稿 |

## 記事一覧

| 日付 | タイトル | URL | PV | スキ | 記録日 |
|------|---------|-----|----|------|--------|
| 2026-03-25 | AIに反論させたら、理論武装が始まった。その先でやったこと | https://note.com/morodomi/n/ndeaebd73afc6 | - | - | - |
| 2026-03-24 | AIに何も指示せずコードを書かせたら半年後に破綻した | draft | - | - | - |
| 2026-03-23 | 4つのAIに同じ設計判断を聞いたら、全員違うことを言った | https://note.com/morodomi/n/n487424b2359b | 16 | 3 | 2026-03-25 |
| 2026-03-16 | AIは理由を探している。README.mdでは足りなかった | https://note.com/morodomi/n/n68627932f546 | 17 | 0 | 2026-03-25 |
| 2026-03-16 | AIは私のコードではなく、働き方を読んでいた。insightsで見えた弱点 | https://note.com/morodomi/n/n4d67a6a8e8b9 | 17 | 1 | 2026-03-25 |
| 2026-03-13 | もし今、私が入社試験を設計するなら | draft | - | - | - |
| 2026-03-13 | exspecでAIが書くテストの欠陥を見つける | https://note.com/morodomi/n/n83223ea8b882 | 8 | 0 | 2026-03-25 |
| 2026-03-13 | チャットAIを使っていたら、開発ワークフローの方が作り変わった | https://note.com/morodomi/n/nd1855519eabe | 34 | 4 | 2026-03-25 |
| 2026-03-12 | AIが書くテストは動く。でも「仕様」として読めるか？ | https://note.com/morodomi/n/naf2a2dcf7c31 | 27 | 0 | 2026-03-25 |
| 2026-03-11 | コードレビューが死んだので、製造業の品質保証に乗り換えた | https://note.com/morodomi/n/ne098fe888c8e | 37 | 1 | 2026-03-25 |
| 2026-03-10 | 壊して覚えるClaudeCode入門 | https://note.com/morodomi/n/n1b9e77606bc5 | 45 | 2 | 2026-03-25 |
| 2026-03-09 | テストの理想形を言語化したらAIへの指示が変わった | https://note.com/morodomi/n/nf9e820194410 | 35 | 0 | 2026-03-25 |
| 2026-03-08 | LLMの自己改善にLLMはいらない。 | https://note.com/morodomi/n/n5de541c88e41 | 29 | 0 | 2026-03-25 |
| 2026-03-08 | Claude Codeのルール管理が破綻した。AIに自分で学ばせた話 | https://note.com/morodomi/n/nd572c0e80d77 | 28 | 1 | 2026-03-25 |
| 2026-03-07 | AIにセキュリティレビューさせるなら攻撃側をやらせる | https://note.com/morodomi/n/n99f7722c2e99 | 34 | 0 | 2026-03-25 |
| 2026-03-06 | コンテキストウィンドウは制約じゃない。フェーズの区切りだ | https://note.com/morodomi/n/n55b4b658e80e | 36 | 1 | 2026-03-25 |
| 2026-03-06 | AIが書いた大量のテストコードを整理する。2つに分類したら構造が見えた | draft | - | - | - |
| 2026-03-06 | typo修正に4人のレビュアーは要らない。リスクベースレビューの設計 | https://note.com/morodomi/n/n00074c4f1d1e | 24 | 0 | 2026-03-25 |
| 2026-03-05 | 暴走するAIを飼い慣らすまで9ヶ月。dev-crewを公開する | https://note.com/morodomi/n/ne40fb866f9c6 | 65 | 0 | 2026-03-25 |
| 2026-03-03 | 死ぬのはSaaSじゃなくて自作SaaSのほうだった | https://note.com/morodomi/n/na3931db69c0f | 25 | 1 | 2026-03-25 |
| 2026-03-02 | 10個の自作スキルを捨ててClaude Codeのデフォルトに戻した | https://note.com/morodomi/n/nd960ef99d0f3 | 98 | 4 | 2026-03-25 |
| 2026-02-26 | コード0行でCLI完成。Claude Codeとの1時間の開発記録 | https://note.com/morodomi/n/n546767bb1135 | 28 | 0 | 2026-03-25 |
| 2026-02-22 | N+1問題はORMの話だと思っていた。166クエリを5に減らすまでの記録 | https://note.com/morodomi/n/nc75415d248c0 | 22 | 0 | 2026-03-25 |
| 2026-02-21 | 大手がAIで新卒を絞る今、地方中小企業に採用チャンスが来ている | https://note.com/morodomi/n/n0d75ba61ae9b | 46 | 4 | 2026-03-25 |
| 2026-02-21 | ゲーム時間をポイント制にしたら、9歳が腕立て10回できるようになった | https://note.com/morodomi/n/nccedc0a60ba8 | 41 | 5 | 2026-03-25 |
| 2026-02-16 | AIと同じ動き方をする新卒が生き残る | https://note.com/morodomi/n/n33f5599a2fec | 37 | 2 | 2026-03-25 |
| 2026-02-16 | 確定申告のチェックをAIにやらせてみた | https://note.com/morodomi/n/n9c8d373665d8 | 162 | 1 | 2026-03-25 |
| 2026-02-13 | 「そうですね」しか言わないAIに、反論を義務づけた | https://note.com/morodomi/n/n4f0533f2ce78 | 57 | 2 | 2026-03-25 |
| 2026-02-12 | エディタがvimからClaude Codeに変わった | https://note.com/morodomi/n/n0d9212702a04 | 189 | 4 | 2026-03-25 |
| 2026-02-12 | 計画の検証は人間がするべきか？ | https://note.com/morodomi/n/n5586e4bc2f3b | 79 | 3 | 2026-03-25 |
| 2026-02-11 | ミスの指摘は、人より機械からがいい | https://note.com/morodomi/n/n0f4bd484a181 | 59 | 1 | 2026-03-25 |
| 2026-02-10 | AIで全員90点の時代、差がつくのは語彙力かな？ | https://note.com/morodomi/n/nb848ca69b1fc | 58 | 3 | 2026-03-25 |
| 2026-02-10 | Claude Codeが暴走しなくなった。tdd-skillsで1チケットを処理する流れ | https://note.com/morodomi/n/ne28b6c06db69 | 160 | 1 | 2026-03-25 |
| 2026-02-10 | レビューはSubagent、実装はAgent Teams | https://note.com/morodomi/n/n7069743cc7e0 | 70 | 1 | 2026-03-25 |
| 2026-02-10 | Claude Codeのスキル、作らないのは損してる | https://note.com/morodomi/n/nd98871405ce8 | 116 | 3 | 2026-03-25 |
| 2026-02-09 | Claude Codeで確定申告の漏れを見つけた話 | https://note.com/morodomi/n/nbea5cde5d73f | 1213 | 16 | 2026-03-25 |
| 2026-02-09 | TDDをClaude Codeに任せる、Agent Teamsで変わったこと | https://note.com/morodomi/n/nbf0dcbd045f1 | 110 | 1 | 2026-03-25 |
| 2026-02-06 | GUIをやめたらAWS設定ミスが消えた | https://note.com/morodomi/n/n2cadd27e61d9 | 57 | 0 | 2026-03-25 |
| 2026-02-05 | Claude Codeに褒められたけど提案は全却下した | https://note.com/morodomi/n/n6a9c562de2e2 | 131 | 1 | 2026-03-25 |
| 2026-02-03 | 週1の技術顧問で、採用の悪循環を断ち切る | https://note.com/morodomi/n/n9ec231eb1ecd | 52 | 1 | 2026-03-25 |
| 2026-02-03 | AIがジュニアの仕事を奪い始めた今、新卒エンジニアが狙うべき「穴場」 | https://note.com/morodomi/n/n921afb8ba02c | 108 | 8 | 2026-03-25 |
| 2026-02-03 | 生成AI時代のフレームワーク選定基準が変わってきた話 | https://note.com/morodomi/n/nf385a40befd5 | 59 | 1 | 2026-03-25 |
| 2026-01-26 | AIに首輪はつけられない。だからレールを敷いて、セキュリティゲートも置いた | https://note.com/morodomi/n/n81f0dbee09b6 | 39 | 0 | 2026-03-25 |
| 2025-12-22 | Claude CodeにTDDを強制したら、AIが勝手にテストを書くようになった話 | https://note.com/morodomi/n/n5b089a48fe7b | 396 | 6 | 2026-03-25 |
| 2025-11-10 | AWS Bedrock gpt-oss-120bでTool利用時の3つのエラーと解決方法 | https://note.com/morodomi/n/n9e819e437d62 | 82 | 2 | 2026-03-25 |
| 2025-07-18 | AIの首輪はすぐ抜ける | https://note.com/morodomi/n/n20787aa82620 | 50 | 2 | 2026-03-25 |
| 2025-07-11 | Claudeはこっそり無駄コードを生成し、バレて嘘をついた | https://note.com/morodomi/n/n1ca3c4a25097 | 163 | 2 | 2026-03-25 |
| 2025-07-11 | AIが壊したプロジェクトを、AIで作り直し、また壊される無限ループの住人 | https://note.com/morodomi/n/naf727f5351cd | 64 | 1 | 2026-03-25 |
| 2023-11-13 | CloudFront配下でLaravelがhttpsにならない問題 | https://note.com/morodomi/n/n039a1427c995 | 960 | 1 | 2026-03-25 |

## Documentation

| ファイル | 内容 |
|---------|------|
| `docs/ネタ/` | ネタ候補ストック |
| `docs/cycles/` | 記事作成サイクル |
| `docs/20260204_note公式分析まとめ.md` | note公式の分析 |
| `docs/20260211_note-skills改善検討.md` | パイプライン改善検討 |

## Codex Integration

- `codex exec --full-auto`: 非対話実行
- `codex exec resume --last --full-auto`: セッション継続（cwdフィルタ）
- `codex review` は使わない

**Auto-orchestrate after plan approve**: The `## Post-Approve Action` section in plan files persists in compressed context after compact. This triggers /orchestrate automatically after compact + accept edits on transition.
