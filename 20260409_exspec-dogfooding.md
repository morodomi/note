# 45,000テストに自作lintをぶつけたら誤検知だらけだった

自作のテストlintツール「exspec」を、自分のプロジェクトに走らせた。942テスト、BLOCKゼロ。完璧だった。

「OSSにもぶつけてみるか」。ここから地獄が始まった。

## 自分のコードでは見えない世界

exspecはRust製の静的テスト品質linter。テストコードを解析して「アサーションのないテスト」「テスト名が曖昧」といった問題を検出する。LLMは使わない。tree-sitterでASTを解析するだけ。速くて安い。

自分のプロジェクトで動かしている限り、精度に問題はなかった。テストの書き方が一定だからだ。自分の癖の範囲内なら、パターンは足りていた。

OSSに投げた瞬間、世界が変わった。

Laravel 10,790テスト。exspecが出したBLOCKは1,305件。確認したら85%が誤検知だった。vitest 3,120テスト、432件BLOCK。NestJS 2,675テスト、90件BLOCK、うち90%が誤検知。

自分の942テストでは完璧だったツールが、他人のコードでは使い物にならなかった。

---

## Round 1: 誤検知の正体

最初に投入したのはvitestのリポジトリ。3,120テストに対して432件のBLOCKが出た。

中身を見ると、exspecが「アサーションなし」と判定したテストのほとんどに、ちゃんとアサーションがあった。exspecが認識できなかっただけだ。

```typescript
// exspecはこれを「アサーションなし」と判定した
expect(result).to.be.an.instanceof(ValidationError);
```

Chaiのプロパティチェーン。to、be、anが連なって、最後にinstanceofで検証している。人間が見れば一目でアサーションだとわかる。でもexspecは、expectの直後にtoBeやtoEqualのようなメソッド呼び出しが来るパターンしか知らなかった。

他にも見逃していたパターンがある。

- expect(x).not.toBe(y)。notという修飾子が挟まるだけで認識できない
- expect(x).resolves.toThrow()。非同期の修飾子チェーン
- Chaiのreturned、ok、trueといったプロパティアサーション

共通点がある。全部「テストが何かを検証している」のに、exspecが認識できていなかった。

---

## 転換点: assertion-freeではなくoracle-free

パターンを1つずつ追加する対症療法では追いつかないと感じて、設計そのものを見直した。

きっかけはGPTの一言だった。「T001はassertion-freeではなくoracle-freeを検出すべきだ」。

テストオラクルという概念がある。テストにおける「期待値の判定」を指す。assertやexpectの呼び出しはオラクルの一形態にすぎない。Chaiのプロパティチェーンも、Mockeryのモック検証も、全部テストオラクルだ。

この一言で検出の設計が変わった。

それまではassertやexpectという文字列を探していた。これを「テストオラクルの形」を検出する方式に切り替えた。

root（expect/assert/should）がある。modifier chain（.not/.resolves/.to.be）が続く。terminal（メソッド呼び出しかプロパティ）で終わる。この3層構造が同じなら、フレームワークが違っても検出できる。

MLは検討して却下した。T001が検出すべきパターンは有限で列挙可能。学習データを集めるコストに見合わない。

---

## Round 2: パターンを1つずつ殺す

oracle-shape detectionに切り替えてから、1日に10以上のTDDサイクルを回した。パターンを見つけて、テストを書いて、実装して、次のパターンへ。

- expect修飾子チェーン対応: vitestのBLOCK 432 → 350
- Chai語彙拡張（returned、ok、true等）: 350 → 326
- PHP Mockery対応（shouldReceive等）: LaravelのBLOCK 1,305 → 776
- PHP オブジェクトアサーション（response->assertOk()等）: 776 → 224

一番苦労したのはChaiのプロパティチェーンの深さ問題だった。

depth-1のexpect(x).to.be.trueは認識できた。だがdepth-5のexpect(x).to.be.an.instanceof(Error)は認識できない。tree-sitterのクエリでプロパティアクセスのネストを表現する必要があり、depth-7まで対応するクエリを手書きした。

NestJSには2,675テストを投入して、90件BLOCK、34件、17件と減らした。最終的にFP率0%。残った17件は全てTrue Positive。helper delegationが8件、done()コールバックのみが3件。全部本当に問題のあるテストだった。

---

## Round 3: アンダースコア1文字の代償

pytest 2,380テストに投入。594件BLOCK。確認したらほぼ100%が誤検知だった。

原因を調べて、目を疑った。

```python
# exspecは ^assert_ にマッチするメソッドだけをアサーションと認識していた
# assertoutcome() はアンダースコアがないので見逃した
reprec = testdir.inline_run()
reprec.assertoutcome(passed=1)  # これがアサーション。_ がないだけ
```

exspecはPythonのアサーションを「assert_」で始まるメソッドとして認識していた。assert_equal、assert_raises、assert_called_with。全部アンダースコアが入っている。

だがpytestの内部ではassertoutcomeのように、アンダースコアなしのアサーションメソッドが使われていた。

修正は1文字。正規表現の「^assert_」を「^assert」に変えた。アンダースコアを消しただけで、大半の誤検知が消えた。

symfony 17,148テストにも投入した。759件BLOCK。addToAssertionCount()というPHPUnitがアサーション数を手動加算するメソッドで91件、markTestSkipped()のみのテストで91件。パターンを追加するたびに数字が削れていく。

---

## 数字で振り返る

1週間で14プロジェクト、4言語、約45,000テストにexspecをぶつけた。35回のTDDサイクルを回した。

主要プロジェクトの結果:

- Laravel（PHP、10,790テスト）: 初回FP率85% → 最終約2%。残差はhelper delegationのみ
- NestJS（TypeScript、2,675テスト）: 初回FP率90% → 最終0%
- vitest（TypeScript、3,120テスト）: 残差はlocal helperのみ
- pytest（Python、2,380テスト）: 初回FP率ほぼ100% → 最終ほぼ0%
- symfony（PHP、17,148テスト）: 初回FP率約24% → 改善中

---

## 精度は出荷後に上がる

dogfoodingは「やったほうがいい」ではなく「やらないと使い物にならない」だった。

自分のプロジェクトだけでテストしていたら、exspecは「自分のコードだけ正しく判定するツール」のままだった。他人のコードにぶつけて初めて、検出の設計そのものが変わった。「assert呼び出しを数える」から「テストオラクルの形を検出する」への転換は、dogfoodingなしでは起きなかった。

精度が100%になることはない。フレームワーク固有のアサーションパターンは際限なく増える。だからexspecには最初からescape hatchを設計に入れた。.exspec.tomlにcustom_patternsを書けば、プロジェクト固有のアサーションパターンを追加できる。パターンの列挙だけでは必ず限界が来る。

自作ツールを作っているなら、自分のコードだけで満足しないこと。他人のコードにぶつけてみる。symfonyの17,148テストはまだ改善中だ。パターンの列挙に終わりはない。

## ハッシュタグ候補
静的解析, dogfooding, テスト設計, OSS開発, exspec, ClaudeCode
