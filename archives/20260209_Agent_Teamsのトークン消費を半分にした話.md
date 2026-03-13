# レビューはSubagent、実装はAgent Teams

#ClaudeCode #AgentTeams #AI開発 #TDD

前回の記事で「Agent Teamsに載せ替えたら3時間でトークンを使い切った」と書いた。

あれから3日。レビュー系だけSubagentに戻した。レビュアー同士が議論する必要がなかったと気づいたからだ。

[tdd-skills](https://github.com/morodomi/tdd-skills) は Claude Code に TDD を強制するスキルだ。7フェーズを自動で回す。

```
INIT → PLAN → RED → GREEN → REFACTOR → REVIEW → COMMIT
```

前回、全フェーズを Agent Teams に載せ替えた。意思決定の質は上がったが、トークン消費が激しすぎた。

## 気づき: レビュアーに議論は要らなかった

最初は「レビュアー同士が議論すれば、指摘の精度が上がるはず」と思っていた。Agent Teams ならエージェント同士が直接議論できる。活かさない手はない。

tdd-skills には2つのレビュースキルがある。plan-review は PLAN フェーズ後に5人、quality-gate は REFACTOR フェーズ後に6人のレビュアーが並列で動く。

Agent Teams モードでは、これらのレビュアーが議論する。broadcast（一斉送信）で全員に指摘を共有し、反証ラウンドで意見を修正する。この仕組みは独自に実装したものだ。

見栄えはする。ただ3日間ログを見返して気づいた。議論で意見が変わることがほとんどない。

セキュリティレビュアーが「SQLインジェクションのリスクあり」と指摘する。パフォーマンスレビュアーは「N+1問題あり」と報告する。この2つは独立した指摘で、議論して合意を取る必要がない。最終判断はPdMが統合すればいい。

quality-gate なら6人、plan-review なら5人。この人数で3ラウンド議論すれば、膨大なトークンが消費される。

独立に結果を返すだけのレビューに、議論のオーバーヘッドをかける理由がなかった。

## やったこと: レビュー系だけSubagent + Sonnetに戻した

変更は単純だ。

```
Before: 環境変数 Agent Teams 有効 → 全フェーズ Agent Teams
After:  レビュー系は常に Subagent + Sonnet / それ以外は Agent Teams
```

plan-review と quality-gate のスキル定義から Agent Teams モードを削除した。環境変数の設定にかかわらず Subagent で動く。レビュアーのモデルは Opus から Sonnet に落とした。

```
Task(subagent_type: "tdd-core:security-reviewer", model: "sonnet", prompt: "...")
```

レビュアーに求めるのは指摘とスコアをJSONで返すこと。Opus の推論力は必要ない。Sonnet で十分だし、トークン単価も安い。

一方、PLAN や RED/GREEN/REFACTOR は Agent Teams のまま残した。PLAN は設計判断に Opus の推論力が要る。RED/GREEN/REFACTOR はコードを書き、テストを実行し、PdM 役の tdd-orchestrate と連携する。Agent Teams の恩恵がある。

判断軸は2つ。議論が必要か、コードを書くか。

- 議論が必要 + コードを書く → Agent Teams + Opus
- 議論不要 + スコアを返すだけ → Subagent + Sonnet

## 結果

Max 20x プランで、Agent Teams 全面採用時は3時間でトークンを使い切っていた。フェーズ別使い分けにしてからは、体感4時間以上は持つ。ただし、プロジェクトの同時起動数やコードベースのサイズで変わる。正確な数値は出せない。

Subagentに戻しても、レビューの質は変わらない。トークン消費だけが減った。

## 教訓: Subagentでも十分だった

エージェント同士が直接会話できることと、独立に実行してPdMに結果を返すこと。tdd-skills のワークフローでは、その違いはほとんどなかった。

Agent Teams が機能するのは、もっと流動的な場面だろう。サッカーのように、相手の動きに合わせてポジションを変える必要がある場面。計画があっても、状況に応じて動きを変えなければならない場面。

tdd-skills のワークフローは一直線だ。計画、実装、検証。PLAN フェーズでソクラテス式対話を使って方針を固めているから、実装フェーズで「どうする？」と相談する場面がない。強力な監督が一人いれば、選手同士が相談しなくても回る。

全部を Agent Teams にするのは、Slackで済む報告をわざわざ会議室を押さえてやるようなものだった。

- 独立に判断できるタスク → Subagent（安い、速い）
- 連携が必要なタスク → Agent Teams（高い、質が高い）

## まとめ

前回「燃費は悪い。でも使い続ける」と書いた。

今回はこう訂正する。「燃費を改善した。使い分ければいい」

Agent Teams は万能ツールではない。フェーズの特性に合わせて Subagent と使い分ける。レビューは Subagent + Sonnet、実装は Agent Teams + Opus。品質を落とさずにトークン消費を抑える方法を模索して、何とかこの方法にたどり着いた。

https://github.com/morodomi/tdd-skills
