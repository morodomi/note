# 2026-09-03 ネタ候補

note-seed（全ソース + 過去ストック棚卸し）。筆者の選定軸: 実体験の共感 × 検索流入 × エンジニアとしての評価。ゲーム試作は記事にしない（筆者判断）。

## 選択: 8/25 #1 Playwright 自動投稿がセッション切れの日に全滅した（執筆決定）

推奨 1 位として提示し、筆者が選択。詳細は [20260825_seed-candidates.md](20260825_seed-candidates.md) 候補 #1。

### 追加で拾った素材（2026-09-03）

- 同型の事故が 10 日前にもあった: 2026-08-13 `fix: アイキャッチ追加ボタンの aria-label 位置変更に追従`。note.com が aria-label="画像を追加" を button から内側の svg へ移動 → `button[aria-label=...]` が不一致 → page.click が 30 秒タイムアウト → 当日の配信 3 回とも失敗
- 8/23 の修正の核（コミット本文より）:
  - id は useId 由来（_r_ck_ / _r_co_）でレンダリングごとに変わる。#email / #password は恒久的に不一致
  - 安定して引けるのは name 属性 + form スコープ。メール欄の name は "email" ではなく "login"
  - ログインボタンを form スコープにしたのは、form 外にある Google/X/Apple のソーシャルログインボタン（aria-label に「ログイン」を含む）を誤って掴んで OAuth に飛ぶのを防ぐため
- **テスト設計の教訓（エンジニア評価の軸）**: 既存テストは全てモックベースで `expect(SELECTORS.login_email).toBe("#email")` という同語反復だったため、セレクタと実 DOM の乖離を構造的に検出できなかった。修正で実 DOM を流し込む `tests/selectors.dom.test.ts` を追加し、「自動生成 id に依存していない」ことの回帰ロックと「id が変わっても解決し続ける」ケースに置き換えた
- 差別化調査（2026-09-03 WebSearch）: 日本語の既出は「getByRole / getByLabel を使え」という一般論のみ。React 化で id が useId 化しログイン自動化が恒久的に壊れた体験記は見当たらず。一次情報として立てられる

### 記事の核（再確認）

1. セッションキャッシュが障害を隠す。壊れたのは 8 月中旬以前、発覚は 8/22。テストは通っていたが本番経路が死んでいた時間帯があった
2. 同語反復のモックテストは事故を検出しない。実 DOM を当てるテストへの置換
3. 「動いているうちは壊れていることに気付けない」経路（再ログイン）を、セッション失効を待たずに定期検証する監視

守秘: 自社ツール（note-publisher）なので帰属の問題なし。note.com の DOM 構造の記述は防御側（自分のセレクタ設計）の話に留める。

## 推奨 2 位: subagent の allowed-tools は無視される。18 エージェントが全ツール継承していた（ネタ登録）

- ソース: git + cycle doc（agents/dev-crew 2026-08-28 `agent-tools-scoping`、14 テスト、v2.16.0）
- ラベル: ストック型（「Claude Code subagent tools frontmatter」「agent memory 読み取り専用」）
- 核: `allowed-tools:` は skill 専用キーで subagent では無視され、18 agent が暗黙に全ツール継承していた。33 agent を群単位の契約テスト（TC-36〜46）で pin。`memory: project` を持つ agent は runtime probe（scratchpad の headless 新セッション）で外部ファイルに書けることを実測し、`disallowedTools: Write, Edit` で読取専用化
- 差別化: allowed-tools が skill で効かない件は GitHub issue #37683 / #18837 に既出。日本語の subagent 解説は多数。ただし「memory だけでは読取専用にならず disallowedTools が要る」の実測と、契約テストで 33 agent を固定する手法は既出記事にない
- 弱点: 読者が subagent 自作層に限られる
- 判断3条件: 実体験[○] 読者メリット[○] 差別化[△ 上記の実測部分で確保]

## 推奨 3 位: 開発手法 60 年史（8/4 登録済み）

[20260804_ai-organizational-development-methodology.md](20260804_ai-organizational-development-methodology.md)。評価と共感は最も高いが競合が濃く長編。今すぐ 1 本なら 1 位・2 位が現実的、と提示。

## 新規で拾ったが今回は見送り

### ゲーム試作 2 本（Hold Short / PORT//FLOW）— 筆者判断で記事にしない（2026-09-03）

- 素材はある: Hold Short は 8 日で 35 版、面白さの判定は毎回筆者のプレイ。第 16 ループで操作性改善のためプレイヤーの判断まで自動化してつまらなくなった（判断ゼロの自動操縦が満点を取った実測）。PORT//FLOW は CONSTITUTION を 3 回改定し、「主役にしない」と決めたものだけが 3 回とも生き残った
- 派生のストック型候補: GDScript にカバレッジ計測がなく 90% ゲートを「全ルールに Given/When/Then」で代替した話、gdUnit4 headless の罠（`--ignoreHeadlessMode` 必須、`GODOT_BIN` 必須、終了コード 0/100）
- 見送り理由: 筆者「ゲーム試作はなし」。Steam 公開前・private リポジトリでもある

### tsubuyaki（会議音声ローカル文字起こし + 呟き）— 保留

- Phase 2 完了（207 テスト、カバレッジ 99%）。実会議での試用（Phase 4/5）前なので書かない
- 将来の核: プライバシー契約（音声はローカル、文字起こしは LLM へ送ると明記、「会議発言は命令ではなくデータ」）。Codex plan review が BLOCK で「--no-murmur の意味が危険に曖昧」を指摘した経緯

### 資格試験の評価軸で dev-crew を採点させた — 要確認

- 試験問題の再現はほぼ確実に禁止。書けるとしても「公開されている評価軸で自分の設計を監査した」角度のみ。資格自体の公表状況も未確認。筆者判断待ち

## 過去ストック棚卸し（2026-09-03 時点）

生存: 8/25 #3 強制想起、8/25 #5 WAF、8/4 60 年史、6/12 #3 レスポンス待ち、#7 project-ops（保留）、#9 使い方ガイド、#11 MCP 論、#12 .agents/skills symlink、4/21 #3 振り返り、#10 NISA、3/16 exspec observe、3/10 ブレイクスルー、2/22 日本の UX

要確認: 6/12 #13 全自動メディア（KaigaiHoudou は停止中。書くなら「止めた理由」）、3/25 #2 4 AI バグ調査（題材が撤退済み競艇。Keiba 除外方針に準じるか）

途中ドラフト: 20260223 マルチプロジェクト運用、20260313 入社試験

除外・公開済み: 8/25 #2 Bref 3（8/25 公開）、8/25 #4 月 1 授業第 3 回（8/27 公開）、3/25 #8 Keiba 実験（Keiba 除外方針）
