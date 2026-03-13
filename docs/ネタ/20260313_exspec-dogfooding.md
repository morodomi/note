# 記事2: 45,000テストと戦った1週間 -- OSSに自作lintをぶつけたら誤検知だらけだった

## ターゲット読者

- 自作ツール・lintを作っている/作りたいエンジニア
- 静的解析の精度改善に興味がある人
- 「dogfooding」の具体的なプロセスを知りたい人

## 主張

静的解析ツールの精度は、作った時点では分からない。実プロジェクトにぶつけて、誤検知を1つずつ潰す泥臭いプロセスが本体。
dogfoodingは機能ではなくプロセスだ。

## 構成

### 1. 導入: 自分のプロジェクトでは完璧だった（200字）

- exspecを自分の942テストに走らせた。BLOCKゼロ。いい感じ
- 「OSSにもぶつけてみるか」。ここから地獄が始まった

### 2. Round 1 -- LaravelとVitest（3/9）: 現実を知る（700字）

- Laravel 10,790テスト → 1,305 BLOCK。85%が誤検知
- vitest 3,120テスト → 432 BLOCK
- 自分のテストでは見えなかった「テストオラクル」の多様性
  - Mockery `shouldReceive()->once()` -- これもアサーションだ
  - Chai `expect(x).to.be.an('array')` -- プロパティチェーンも
  - `expect(x).not.toBe(y)` -- 修飾子チェーンも
- 共通点: 全部「テストが何かを検証している」のに、exspecが認識できていなかった
- **誤検知されていたコード例（Laravel）:**

```php
// exspecはこれを「アサーションなし」と判定した
public function test_user_can_login()
{
    $response = $this->post('/login', ['email' => 'test@example.com', 'password' => 'password']);
    $response->assertOk();           // ← これがアサーション。でもexspecは認識できなかった
    $response->assertRedirect('/');   // ← これも
}
```

### 3. 転換点: 「oracle-free」というリフレーミング（600字）

- 4つのAIに同じ質問をぶつけた（詳細は別記事で書く）
- GPTの回答: 「T001はassertion-freeではなくoracle-free。テストオラクルを検出すべき」
- この一言で設計が変わった
  - assert呼び出しを数える → テストオラクルの「形」を検出する

**assertion detection → oracle-shape detectionの転換:**

```
[BEFORE] assert / expect / should を文字列マッチ
  → 各フレームワーク固有の書き方を全部見逃す

[AFTER]  テストオラクルの「形」をパターンマッチ
  root (expect/assert/should)
    → modifier chain (.not / .resolves / .to.be)
      → terminal (メソッド呼び出し or プロパティ)
  → 形が同じなら、フレームワークが違っても検出できる
```

- MLは却下。理由: T001のパターンは有限で列挙可能。MLの学習データ収集コストに見合わない

### 4. Round 2 -- FP修正の連続（3/9-3/10）: パターンを殺す（800字）

- 1日に10+のTDDサイクルを回した
- 各パターンの修正:
  - **expect修飾子チェーン**: `.not.toBe()`, `.resolves.toThrow()` → vitest 432→350 (-82)
  - **Chai語彙拡張**: `returned`, `ok`, `true` → vitest 350→326 (-24)
  - **PHP Mockery**: `shouldReceive`, `shouldHaveReceived` → Laravel 1305→776 (-529)
  - **PHP obj->assert**: `$response->assertOk()` → Laravel 776→224 (-552)
- Chai property chainの深さ問題
  - depth-1: `expect(x).to.be.true` → 認識できた
  - depth-5: `expect(x).to.be.an.instanceof(Error)` → 認識できない
  - depth-7まで対応するクエリを書いた。tree-sitterの限界と戦う
- **誤検知されていたコード例（vitest）:**

```typescript
// depth-5のChainプロパティ。exspecはこれを「アサーションなし」と判定した
expect(result).to.be.an.instanceof(ValidationError);
```

- NestJS 2,675テスト投入 → 90 BLOCK → 34 → 17。最終FP率0%
  - 残り17件は全てTP（helper delegation 8、done() 3、bare expect 2...）

### 5. Round 3 -- pytest/django/symfony（3/11）: 最後の壁（700字）

- pytest 2,380テスト → 594 BLOCK。ほぼ100%誤検知
  - 原因: `^assert_` パターン。`reprec.assertoutcome()` を見逃していた
  - 修正: `^assert_` → `^assert`。アンダースコア1文字で594 FPが消えた
- **誤検知されていたコード例（pytest）:**

```python
# exspecは ^assert_ にマッチするメソッドだけをアサーションと認識していた
# assertoutcome() はアンダースコアがないので見逃した
reprec = testdir.inline_run()
reprec.assertoutcome(passed=1)  # ← これがアサーション。_ がないだけ
```

- symfony 17,148テスト → 759 BLOCK
  - `addToAssertionCount()` (91件): PHPUnitがアサーション数を手動加算するメソッド
  - `markTestSkipped()` のみのテスト (91件): スキップされるだけ → T001の対象外にすべき
- django 1,047テスト → 23 BLOCK、39%FP。残差はhelper delegation

### 6. 数字で振り返る（300字）

| プロジェクト | 言語 | テスト数 | FP率（初回）| FP率（最終）|
|-------------|------|---------|------------|------------|
| Laravel | PHP | 10,790 | 85% | ~2% (残差=helper) |
| NestJS | TS | 2,675 | 90% | 0% |
| vitest | TS | 3,120 | -- | 残差=local helper |
| pytest | Python | 2,380 | ~100% | ~0% |
| symfony | PHP | 17,148 | ~24% | -- |

1週間で35 TDDサイクル。パターン修正のたびにテストを書き、既存テストが壊れないことを確認した。

### 7. 結論: 精度は出荷後に上がる（400字）

- dogfoodingは「やったほうがいい」ではなく「やらないと使い物にならない」
- 自分のプロジェクトだけでは見えない世界がある
- 誤検知を潰すプロセス自体がツールの設計を改善する
  - 「assert呼び出し検出」→「オラクル検出」への転換は、dogfoodingなしでは起きなかった
- 精度が100%になることはない。だからescape hatch（custom_patterns）を最初から用意した
  - 自作lintを作るなら、最初からescape hatchを設計に入れておくべき。パターンの追加だけでは必ず限界が来る
- あなたが自作ツールを作っているなら、自分のコードだけで満足しないこと。他人のコードにぶつけてみる。見える世界が変わる

## 素材

- docs/dogfooding-results.md（全数字の出典）
- docs/cycles/（35 TDDサイクルの記録）
- MEMORY.md「T001 FP Architecture Decision」
- tests/fixtures/（誤検知されていたコード例の参考）

## 想定文字数

4,500-5,500字

## ハッシュタグ

静的解析, dogfooding, テスト設計, OSS開発, 誤検知, ClaudeCode, Rust

## Review Notes

- [x] 「なぜ精度が上がったか」の技術的因果をもう1段見せたい → セクション3にBEFORE/AFTERの模式図を追加
- [x] `assertion detection` から `oracle-shape detection` に変えたことを模式図か擬似コードで示す → セクション3に構造図を追加
- [x] 各1例ずつ「誤検知されていたコード断片」を入れる → セクション2(Laravel), 4(vitest), 5(pytest)に具体コード追加
- [x] Issue番号は公開記事では読み手に負荷がある。本文では最小限に絞る → Issue番号を全削除
- [x] 末尾に「自作lintを作るなら最初からescape hatchを用意しておくべき」の実務的学びを明示 → セクション7に追加
- [x] 「自分のツールを他人のコードにぶつけてみる」という読後アクションを1文入れる → セクション7末尾に追加
