# Claude CodeのSkill()チェーンがトークンを爆食いする問題と対策

## ステータス: ネタメモ（実装中）

issue: morodomi/dev-crew#27
cycle doc: /Users/morodomi/Documents/AgentSkills/dev-crew/docs/cycles/20260217_2200_token-optimization-auto-transition.md

## 記事の種

### 核となる知見

Skill()はメイン会話のコンテキストをフォーク（継承）する。Skill(A)の中でSkill(B)を呼ぶと、キャッシュ読み取りが累積して爆発する。

```
メイン会話: 50K tokens
  └─ Skill(A): 50K 継承 + 10K 追加 = 60K
       └─ Skill(B): 60K 継承 + 5K 追加 = 65K
```

一方、Task()は新規プロンプトのみで起動するので軽い（固定コスト）。

### 数字

- red→green→refactor→review を手動実行した場合: auto-transitionあり 275K vs なし 200K
- 差分: 75K節約（27%削減）
- X.comのポストでは3スキルを1つにバッチ化して70%+改善の報告あり

### 実際に踏んだバグ

- plan-reviewが2回実行される（architect内で自動発火 + PdMが再度実行 = reviewerが倍起動）
- quality-gateが2回実行される可能性（同構造）

### 対策

- 個別スキルからauto-transition（Skill()チェーン）を除去
- orchestrateレイヤーでTask()委譲に統一
- 完了メッセージで次のステップを案内（手動時は /next-skill で）

## 切り口案

1. 「Claude Codeのスキルを連鎖させたらトークンが爆発した話」（体験記）
2. 「Skill()とTask()、使い分けを間違えると27%損する」（How-to寄り）
3. 「AIエージェントのコスト最適化、やったのは引き算だった」（主張型）

## 関連する既存記事

- 「Claude Codeのスキル、作らないのは損してる」（2026-02-10, PV 77）
- 「レビューはSubagent、実装はAgent Teams」（2026-02-10, PV 59）
- 「Claude Codeが暴走しなくなった。tdd-skillsで1チケットを処理する流れ」（2026-02-10, PV 67）

## メモ

- 実装が完了してから書く。結果の数字が出てから
- Skill()とTask()のコンテキスト継承の違いは、多くのClaude Codeユーザーが知らないはず
- X.comのポストの元ネタも引用できるとよい
