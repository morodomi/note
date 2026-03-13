# 記事1: exspec実践ガイド -- テスト設計の問題を見つける静的解析ツール

前提記事（記事0）: 「AIが書くテストは動く。でも「仕様」として読めるか？」（公開済み 2026-03-12）
https://note.com/morodomi/n/naf2a2dcf7c31

## ターゲット読者

- 記事0を読んで「面白そうだけど具体的に何ができるの？」と思った人
- AIでテストを書いているが、品質に漠然と不安がある人
- CIに何か入れたいけど、カバレッジ以外に何を入れればいいか分からない人
- チームのテスト品質を底上げしたいリード/テックリード

## 主張

exspecは「テストを書いたあと」に走らせるlint。
カバレッジが「どこをテストしたか」を測るなら、exspecは「テストがちゃんと仕様になっているか」を測る。
難しい設定不要、cargo installして走らせるだけで、テスト設計の問題点が見える。

## 構成

### 1. 導入: カバレッジ100%でも安心できない理由（300字）

- カバレッジが高い = テストが良い、ではない
- 実例: assertが1つもないテスト。通るけど何も検証していない
- 実例: mockだらけのテスト。実装を少し変えただけで壊れる
- こういう「構造的な問題」はカバレッジでは見えない
- exspecはここを見る

### 2. 誰がいつ使うか（400字）

**こんな場面で使う:**
- AIにテストを書かせたあと、「これ本当に大丈夫？」と確認したい
- PRレビュー前にセルフチェックしたい
- 既存テストの棚卸し。どこから手をつけるか優先順位をつけたい
- CIに組み込んで、テスト設計品質の劣化を自動検知したい

**こんな人に向いている:**
- Python / TypeScript / PHP / Rust でテストを書いている人
- 1人〜小規模チーム（レビューの目が足りない）
- AIにテスト生成を任せている人（品質の担保手段がない）

### 3. インストールと最初の実行（400字）

```bash
cargo install exspec
exspec .
```

- Rustのツールチェーンがあれば1コマンド
- 言語指定: `exspec --lang python .`
- **実際の出力例を見せる:**

```
exspec v0.1.2 -- 8 test files, 42 test functions
BLOCK tests/test_user.py:12 T001 assertion-free: test has no assertions
WARN  tests/test_user.py:45 T002 mock-overuse: 6 mocks (4 classes), threshold: 5 mocks / 3 classes
WARN  tests/test_api.py:78 T003 giant-test: 62 lines, threshold: 50
INFO  tests/test_api.py:120 T109 undescriptive-test-name: suffix 'it' is too generic
Score: BLOCK 1 | WARN 2 | INFO 1 | PASS 38
```

- BLOCKが出たらそこが最優先。WARNはコンテキスト次第。INFOは参考程度
- 「まずは走らせて、BLOCKが何件出るか見る」が最初のステップ

### 4. 何を検出するか -- 代表的なルール5つを具体例つきで（1,500字）

各ルールをbefore（NG）→ after（OK）のコード例つきで紹介。
各例に「この1件を直すと何が変わるか」を1文で添える。

**T001 assertion-free（BLOCK）-- アサーションのないテスト**
- before: 関数を呼ぶだけで何も検証しない
- after: 戻り値や状態をassertで検証
- なぜ問題か: テストが通っても、何も保証していない
- 直すと: 「このテストが通る = この振る舞いが保証されている」と言えるようになる

**T002 mock-overuse（WARN）-- mock依存が多すぎる**
- before: 1テストに6個のmock
- after: 必要最小限（外部API等のみ）
- なぜ問題か: 実装のリファクタでテストが壊れる
- 直すと: リファクタしてもテストが壊れない。テストが「仕様」として安定する

**T003 giant-test（WARN）-- 巨大すぎるテスト**
- before: 80行のテスト関数
- after: 責務ごとに分割
- なぜ問題か: 失敗したとき「何が壊れたか」分からない
- 直すと: テスト名だけで「どの仕様が壊れたか」が即座に分かる

**T101 how-not-what（WARN）-- 実装の詳細をテストしている**
- before: privateフィールドに直接アクセスして検証
- after: 公開インターフェース経由で検証
- なぜ問題か: 内部構造を変えただけでテストが壊れる
- 直すと: 内部リファクタが自由にできる。テストが振る舞いだけを保証する

**T109 undescriptive-test-name（INFO）-- テスト名が曖昧**
- before: `test_1`, `test_it_works`
- after: `test_empty_cart_returns_zero_total`
- なぜ問題か: テスト名が仕様書の目次。曖昧だと仕様が読めない
- 直すと: テスト一覧がそのまま仕様書になる

残り12ルールの全仕様: [docs/SPEC.md](https://github.com/morodomi/exspec/blob/main/docs/SPEC.md)
各ルールの設計思想: [docs/philosophy.md](https://github.com/morodomi/exspec/blob/main/docs/philosophy.md)

### 5. 「まずBLOCKだけ」で始めた成功例（300字）

- 自分のプロジェクト（942テスト）で初めて走らせた
- BLOCK 7件。全部「assertのないテスト」
- 中身を見たら4件はセットアップ確認（例外が出ないことを確認するだけ）
- assertを1行ずつ足した。30分の作業
- これだけで「このテストスイートは全テストが何かを検証している」と言える状態になった
- 最初から全ルール有効にする必要はない。BLOCKゼロが最初のゴール

### 6. 段階的に導入する（500字）

**Step 1: Tier 1だけで始める**
- `.exspec.toml` でTier 2を無効化
- BLOCKを全部潰す（アサーションのないテストをなくす）
- これだけでテストの最低品質が保証される

**Step 2: Tier 2を1ルールずつ有効化**
- まずT101（how-not-what）を試す。mock依存の問題が可視化される
- 次にT109（undescriptive-test-name）。テスト名の棚卸し
- 一気に全部有効にしない。ルールごとに対応する

**Step 3: プロジェクト固有の調整**
- `custom_patterns`: プロジェクト固有のassertヘルパーを登録
- `[rules.severity]`: ルールごとに重要度を変更/無効化
- `# exspec-ignore: T002`: インラインで特定行をサプレス
- 設定例を具体的に見せる

**Step 4: CIに入れる**
- GitHub Actions / GitLab CI の設定例
- `exspec .` はBLOCK時にexit 1。CIが落ちる
- `--strict` でWARNも落とせる
- SARIF出力でGitHub Code Scanningに統合可能

### 7. 制約を正直に（300字）

- Rustマクロ内のテストは検出できない（tree-sitterの制約）
- プロジェクト固有のassertヘルパーは `custom_patterns` で登録が必要
- T109の日本語テスト名対応は不完全
- 「仕様として読めるか」の最終判断は人間がする。exspecは構造的な兆候を見つけるだけ
- 13プロジェクト/45,000テストで検証済み。でもまだv0.1.2

### 8. まとめ: まず自分のリポジトリで1回走らせてみる（200字）

- `cargo install exspec && exspec .` だけで始められる
- BLOCKが0件なら、最低限のテスト品質は確保されている
- BLOCKが出たら、そこがテスト改善の最優先ポイント
- 記事0で書いた「テストが仕様として読めるか」を、手を動かして確認できるツール
- まず1回走らせる。数字が出れば、次に何をすべきか見える
- GitHub: https://github.com/morodomi/exspec

## 素材

- README.md（インストール、ルール一覧、設定例、CI例）
- docs/SPEC.md（全ルールの入力→期待出力）
- docs/configuration.md（.exspec.toml設定リファレンス）
- docs/known-constraints.md（制約とワークアラウンド）
- docs/ci.md（CI統合例）
- tests/fixtures/（各言語の違反/準拠コード例）

## 想定文字数

4,000-5,000字

## ハッシュタグ

テスト設計, 静的解析, exspec, CI, ClaudeCode, Python, TypeScript, PHP, Rust

## Review Notes

- [x] 実践ガイドとしては、読者が「自分でも試せそう」と思える最小の実例がまだ弱い → セクション3に出力例追加、セクション5に成功例追加
- [x] `exspec .` 実行時の出力例を1つ入れる → セクション3に追加
- [x] before / after のコード例に加えて、「この1件を直すと何が改善されるか」を短く添える → セクション4の各ルールに「直すと:」を追加
- [x] 「まずBLOCKだけ見る」で改善できた短い成功例を1つ入れると、導入ハードルが下がる → セクション5を新設
- [x] 残り12ルールをREADMEに逃がす方針は妥当だが、読者導線として「次にどこを見るか」を明記した方がよい → セクション4末尾にSPEC.mdとphilosophy.mdへのリンク追加
- [x] まとめでは GitHub リンクに加えて、「まず自分のリポジトリで1回走らせる」という CTA を1文入れる → セクション8のタイトルとCTAに反映
