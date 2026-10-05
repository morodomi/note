# 2026-08-25 ネタ候補

note-seed（全ソース探索）で抽出。筆者判定: #1 / #2 / #3 / #5 は「アリ」。#4（月1授業第3回）は実施後に書く前提で保留。

## 候補 #1: Playwright 自動投稿が「セッション切れの日」に突然全滅した。React 化で id が動的になる罠

- **ステータス: 執筆決定（2026-09-03 note-seed で筆者選択。詳細は 20260903_seed-candidates.md）**
- ソース: git（automation/note-publisher 2026-08-23 `fix: note.com の React 化でログインセレクタが一致せず再ログイン不能になる`）
  - note.com ログインページが React 化され、input の id が useId 由来の自動生成値（_r_ck_ / _r_co_）に変化。レンダリングごとに変わるため #email / #password は恒久的に不一致
  - セッション有効中は再ログインが走らないので気付けず、2026-08-22 のセッション失効時に初めて表面化。page.fill が 30s タイムアウト、3 回リトライも全滅、4 記事が未公開に
- ラベル: ストック型（SEO: 「Playwright useId セレクタ」「React ログイン 自動化 壊れた」）
- パターン: 体験記 + How-to（id 依存セレクタ → role/label/name ベースへの置換、セッション失効を待たずに再ログイン経路を定期検証する監視）
- 読者メリット: Playwright/Puppeteer で他社サイトを自動操作している人向け。「動いているうちは壊れていることに気付けない」経路の検知方法
- ピックアップ狙い: -
- 判断3条件: 実体験[○] 読者メリット[○] 差別化[要確認 — useId セレクタ崩壊は既出記事があるか調査]
- **記事の核**: セレクタの話より「セッションキャッシュが障害を隠す」構造の話。壊れたのは 8 月中旬以前だが発覚は 8/22。テストが通っていても本番経路が死んでいる時間帯があった

## 候補 #2: Bref 3 / AL2023 移行で踏んだ 2 つの地雷 — Lambda パッケージサイズ超過と compiled view の read-only 500

- **ステータス: 公開済み（2026-08-25 https://note.com/morodomi/n/n46c4f0994299）**
- きっかけ（筆者談）: AWS からの通知（provided.al2 ランタイム廃止予告）を受け、既存の Bref/PHP 環境の Lambda を入れ替えた
- ソース: git（2026-07-30 `fix: Lambda デプロイパッケージのサイズ超過を解消` / `fix: bridge v3 で compiled view 同梱が read-only 書き込み 500 を起こす問題を解消`）+ cycle doc `bref3_al2023_runtime_migration`（risk: high、7 テスト）+ 8/20 の後日談（serverless.yml / buildspec で resource・e2e を除外）
- ラベル: ストック型（SEO: 「Bref 3 移行」「provided.al2023 Laravel」「Lambda package size」）
- パターン: 体験記 + How-to
- 読者メリット: Laravel on Lambda（Bref）利用者向け。Bref 2→3 の実移行手順と、公式ドキュメントに書いていない 2 つの落とし穴
- ピックアップ狙い: -
- 判断3条件: 実体験[○] 読者メリット[○] 差別化[○に近い — 日本語の Bref 3 移行記事は少ない見込み。要調査]
- 注意: 守秘方針は調査結果末尾を参照（帰属も書かない・数値もぼかす）
- 参考: 同系統の「CloudFront 配下で Laravel が https にならない」は 965PV（長期流入型の実績あり）

### 調査結果（2026-08-25 note-research）

**一次資料（自社 cycle doc・fix コミット 2026-07-30〜31）**:
- きっかけ: AWS からの provided.al2 廃止通知。AWS 公式日程は deprecation 2026-07-31 / 新規関数作成ブロック 2027-02-01 / 既存関数更新ブロック 2027-03-03（[AWS Lambda runtimes](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html) の Deprecated runtimes 表で 2026-08-25 再確認。AL2 自体の EOL は 2026-06-30）。更新ブロック以降はロールバック不能
- 変更は composer.json 2 行 + serverless.yml 1 行（bref/bref 2.3.11→3.0.7、laravel-bridge 2.4.4→3.0.8、provided.al2→provided.al2023）。PHP 8.3 / Laravel 11 据え置き。ここまでは「公式の言う通り smooth」
- 地雷 1（デプロイ失敗）: `--with-all-dependencies` で 70 パッケージが transitive upgrade → vendor 肥大 → code+layers 269MB が 250MiB（262,144,000 bytes）上限を超過。CloudFormation ロールバック。対処: buildspec の `sls deploy` 直前に `composer install --no-dev --optimize-autoloader`（dev 依存 約76MB 除去）+ serverless.yml `package.patterns` に docs/ mysql/ php/ infrastructure/ .github/ reports/ 等 8 除外
- 地雷 2（dev 全ページ 500）: laravel-bridge v2 は `view.compiled` を無条件で /tmp 配下に上書き（v2.4.4 の BrefServiceProvider を実確認: `Config::set('view.compiled', StorageDirectories::Path . '/framework/views')` に条件なし）。v3 は `if (! is_string($currentCompiledPath) || ! is_dir($currentCompiledPath))` の時だけ上書きに変更（[master の BrefServiceProvider](https://github.com/brefphp/laravel-bridge/blob/master/src/BrefServiceProvider.php) で実確認）。buildspec で `view:cache` を走らせ `resources/cache/` をパッケージに同梱していたため、v3 はそれを「尊重」して read-only の /var/task に書きに行き 500。しかもこの compiled view は BladeCompiler basePath 空によるハッシュ不一致で Lambda 上では一度も参照されていない死物だった。対処: `!resources/cache/**` 除外 + `view:cache` 削除
- **訂正（2026-08-25 note-review integrity）**: is_dir 条件は v3 起源ではなく laravel-bridge **2.7.0（2025-09-12、PR #188「Allow pre-compiling views before deployment」）** で導入（2.7.0 raw ソースで実確認）。2.7.0 の release notes に 1 行あり。Bref v3 upgrade guide と 3.0.0 の notes には記載なし。2.4.4→3.0.8 へ飛んだため v2/v3 差に見えただけ。view:cache 追加は 2025-02-03（git log）で「2 年以上」は誤り→「1 年半」
- 副次: composer audit 39→9 勧告に改善（副産物で livewire critical CVE 発見 → 別 issue）。Bref 3 は Monolog CloudWatch formatter がデフォルト有効（唯一の公式 breaking change）で、メトリクスフィルタはテキスト `ERROR` マッチのため実測で無変更で OK だった
- Codex plan review が BLOCK×4 を出した（AWS 更新ブロック日の誤り 2026-09-30→2027-03-03、部分ロールバック不可 等）。「AI の計画を AI がレビューして日付誤りを潰した」は筆者の過去記事路線と接続可能

**類似記事と差別化ポイント**:
- [Upgrading Bref 2 to Bref 3 with zero downtime（atymic.dev, 英語）](https://atymic.dev/blog/bref-2-to-bref-3-zero-downtime/): 同じく 250MB 超過を踏んでいる。著者は「統合レイヤーが 46MB→92MB に倍増した」と主張し、Zip を諦めてコンテナイメージ + 並行スタック切替へ。**本記事は Zip のまま `--no-dev` + 除外パターンで解決した軽量ルート**で対比できる。compiled view の話は一切なし
- 日本語: Bref 3 移行の体験記は Qiita/Zenn/note で **見つからず**（[マーベリックス](https://ma-vericks.com/blog/serverless-laravel-app/) 等は Bref 導入記事で v3 非対応）。日本語一番乗りの可能性が高い
- 公式 [Bref 3.0 リリース記事](https://bref.sh/news/03-bref-3.0) は「Almost nothing broke / the upgrade should be smooth」と書いており、記事の導入で対比に使える
- **数値の食い違い（要注意）**: 公式は「PHP 8.3 レイヤー 65MB→46MB に縮小」、atymic は「46MB→92MB に倍増」。前者は単一用途レイヤー同士、後者は fpm+console 統合後の比較と思われるが未確定。記事では他人の数値は引用せず、**自分の実測（269MB / dev 依存 76MB）だけを使う**

**技術情報（一次ソース）**:
- [Bref v3 Upgrade guide](https://bref.sh/docs/upgrading/v3): PHP 8.2+ 必須、Laravel 10/11/12、レイヤー統合（php-xx-fpm/console → php-xx）、レイヤー公開アカウント 534081306603→873528684822、`vendor/bin/bref` CLI 削除、Monolog formatter デフォルト有効、glibc langpack ロケール非同梱（setlocale 依存は注意）
- Lambda 上限: 関数コード + レイヤー合計 250MB（unzipped）（[Bref Deploy](https://bref.sh/docs/deploy)）

**反論材料（先回りすべき批判）**:
1. 「`--no-dev` してないのが悪い」→ その通り。ただし v2 では収まっていたので気付く契機がなかった。「動いていたものが major アップで限界を踏む」構造の話にする
2. 「view:cache を Lambda で使うのが間違い」→ 半分正しい。だが v2 が黙って上書きしていたので、無意味な view:cache が 2 年以上放置されていたことに誰も気付けなかった。**「前バージョンの寛容さが設定の腐敗を隠す」**が本記事の核
3. 「コンテナイメージにすれば全部解決」→ atymic ルート。package type は immutable なので Zip→Image は並行スタックが必要で、小規模案件には重い。Zip 維持の判断理由を書く
4. 「日付をそのまま信じるな」→ AWS は日程を変更し得る。記事内でも「実施直前に公式表を再確認」と明記

**守秘（2026-08-25 筆者確定）**: 案件の帰属（受託/自社）は書かない。「運用している Laravel on Lambda 環境」とだけ書く。サービス名・ドメイン・AWS アカウント ID・関数名・メトリクスフィルタ名は伏せる。実測数値もぼかす（「250MB 上限を超えた」「dev 依存が数十 MB」程度）。技術的事実（bridge のコード差分・AWS 日程・composer コマンド）は一次ソース付きで書く

## 候補 #3: AI が cycle doc を書いても、次の spec で思い出さない。「強制想起」を仕組みにした話

- **ステータス: ネタ登録（2026-08-25）**
- ソース: git（dev-crew 2026-07-23 `feat: spec 時強制想起 — 関連 cycle doc の自動提示` / `feat: Cycle-Doc トレーラー自動付与 + 想起漏れ計測設問`、2026-07-24 `codified insight 19 件を rule 条項化`）+ cycle doc `spec-forced-recall`（16 テスト）
  - 過去の失敗記録（cycle doc / retrospective）は存在するのに、AI は次の spec で参照しない。co-change + コミットトレーラーで関連 cycle doc を決定論的に候補生成し、spec 時に強制提示
- ラベル: フロー型（エンゲージ）。2026-08-04 公開「CLAUDE.md に思想を書いても AI は守らない。開発哲学は仕組みに埋める」の続編ポジション
- パターン: 主張型 + 体験記
- 読者メリット: Claude Code で記録は溜まるが再利用されない、という悩みへの回答。「記憶」を LLM に頼らず git メタデータで代替する設計
- ピックアップ狙い: 条件付き（不足: 想起漏れの計測数値。「想起漏れ計測設問」の集計結果があれば強い）
- 判断3条件: 実体験[○] 読者メリット[要確認] 差別化[要確認 — Claude Code memory 機能との比較が必要]
- **記事の核**: 「AI にメモリを持たせる」ではなく「AI に思い出させる契機を人間側のワークフローに固定する」。決定論的（LLM 非依存）な候補生成が肝

## 候補 #4: 10 歳への月 1 授業、第 3 回「継続力とやめる勇気」（保留）

- **ステータス: 公開済み（2026-08-27 https://note.com/morodomi/n/n1e8e3ade18ea 「面倒くさい」はやめる理由になるか）**
- ソース: session（2026-08-24〜25 授業設計）— 水泳 5 年継続の実例、注意された時の「魔法のひとこと」、祖父にも褒めてもらうロールプレイ
- 不足: 授業実施後の子どもの反応。第 1 回・第 2 回と同様、実施前は書けない

## 候補 #5: WordPress サイトの WAF ルール設計 — /wp-json/batch/v1 遮断とギャンブルスパム検索の Count 運用

- **ステータス: ネタ登録（2026-08-25）**
- ソース: git + cycle doc（2026-07-28 `WAF Rule 15 で wp2shell 攻撃経路 /wp-json/batch/v1 を遮断` → 7 日観測ゲート → 誤爆ゼロ確認、2026-08-14 `WAF にギャンブルスパム検索の専用ルールを Count で追加`）
  - Count モードで観測 → 誤爆ゼロを確認 → Block へ切替、という段階的カットオーバー手順が cycle doc に記録されている
- ラベル: ストック型（SEO: 「wp-json batch WAF」「WordPress スパム検索 WAF」）
- パターン: How-to + 体験記
- 読者メリット: WordPress を AWS WAF で守っている運用者向け。誤爆を恐れて Block に踏み切れない人への「Count で観測してから切る」実手順
- ピックアップ狙い: -
- 判断3条件: 実体験[○] 読者メリット[○] 差別化[要確認]
- 注意: クライアント案件。社名・ドメイン・ルール番号は伏せて一般化が必須。攻撃手法の詳細は防御側の記述に留める（攻撃手順の再現は書かない）
