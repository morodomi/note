# 2026-06-12 ネタ候補

## 採用（執筆予定）

### 候補 #1: Next.js 16でCSPのnonce化を進めたら、style-srcだけ断念した話
- **ステータス: ドラフト作成済み（2026-06-15）→ `note/20260615_Next.jsのCSPをnonce化したら本番で2回死んだ.md`。地雷は2つだった（①style-src nonce+unsafe-inline併用は仕様で不可=Tailwind崩れ ②RSC streaming __next_f に nonce 付かず白画面→request headerへ転送）。レビュー未実施**
- ソース: git + cycle (ShaReco) — `c37e790 style-src の nonce 化を断念 (CSP 仕様で nonce + unsafe-inline 併記が不可)`, `adff388 style-src に 'unsafe-inline' を併用 (Next.js 16 + Tailwind 4 制約)`, `11e80e9 CSP に Vercel Live 許可`, `f3845cb CSP unsafe-inline → nonce-based 移行 (#62 SEC-008)`, `63cedd0 CSP violation 検知 smoke spec`
- 参考 cycle: `SaaS/ShaReco/docs/cycles/069-csp-nonce.md`
- ラベル: ストック型（SEO検索流入）
- パターン: 体験記（トラブルシューティング型）
- 二重の罠:
  1. React の `style={{...}}` は nonce 付与不可（React 仕様制約）→ unsafe-inline 併用が必要
  2. CSP 仕様上 nonce と 'unsafe-inline' を併記すると 'unsafe-inline' が無視される → 妥協自体が成立しない
- 最終着地: `style-src-attr 'unsafe-inline'` を残し、CSP violation 検知 smoke spec で監視
- 読者メリット: Next.js + Tailwind で CSP 厳格化する人が必ず踏む地雷の回避ルート
- 検索語彙: 「Next.js CSP nonce」「style-src unsafe-inline 効かない」「Tailwind CSP」
- 一次情報度: 高 / エバーグリーン度: 高
- ピックアップ狙い: 可能
- 判断3条件: 実体験[○] 読者メリット[○] 差別化[要確認]
- 類似実績: CloudFront https問題 991PV、Lambda+SQS 602PV（同型のSEO地雷記事）

## ストック

### 候補 #3: AIのレスポンス待ち、何してる？「完了をすぐ知りたい」を解決するまで
- 角度（筆者指定）: ツール紹介ではなく**問題から書く**。AIに仕事を投げた後の待ち時間に何をするか問題 / 並列セッションのどれが完了したかをすぐ知りたい問題。その解決としての cc-session-manager（Slack通知 + メニューバー表示）+ Terminal 分割
- ソース: session + git (automation/cc-session-manager) — `cc70e59 fix: セッション状態検出の正確性を修正`, `112afda feat: セッション0件時にメニューバーへCC表示`, `5be1a7d state-detection-accuracy サイクル記録`
- ディテール（session より）: UltraWide (3440x1440) で Terminal 4x3=12分割を検討 → 1ペイン約80文字幅でギリギリ実用的 → `automation/tile-terminals.applescript` に着地。tmux 流行への言及も可
- ラベル: フロー型（Claude Code workflow 系）
- パターン: 体験記 + How-to
- 読者メリット: 複数セッション並列運用時の待ち時間ロスと「完了に気づかない」ロスの解消
- 一次情報度: 高 / エバーグリーン度: 中
- ピックアップ狙い: 可能（12分割ターミナル + Slack通知 + メニューバーのスクリーンショット映え）
- 判断3条件: 実体験[○] 読者メリット[○] 差別化[要確認]
- 関連ネタ: docs/ネタ/20260222_multi-project-workflow.md（合体可能）

### 候補 #7: AIは問題を「見つける」のは得意で「選ぶ」のは苦手。だから意思決定ループを作った
- **ステータス: 保留 — project-ops がまだ作成途中（2026-06-12 筆者）。完成・運用実績が出てから書く**
- ソース: git (agents/project-ops) — probe / vet / propose / prioritize / ticket-draft / orchestrate の6スキル
- 参考: `agents/project-ops/CONSTITUTION.md`, `docs/decision-model.md`
- ラベル: フロー型（主張型 + 体験記）
- 核となる主張:
  - 枠の中の実行者は自分の枠を監査できない（実装AIは spec の枠内でしか成功を測れない）
  - AI は surface（発見）は得意、select（どれが正しい問いか）は苦手 → 発見と選択を分離
  - AI の破滅的失敗は「Complex を Clear と誤判定して自信満々に自動処理」→ Cynefin を自律の dial にする
- 判断3条件: 実体験[○] 読者メリット[○] 差別化[○に近い]

### 候補 #9: Claude Codeの効率的な使い方ガイド — 設定ファイルからプロンプトのコツまで
- 角度（筆者指定）: 「Claude Code の効率的な使い方の紹介」として構成。流れ: CLAUDE.md などの設定ファイル → CONSTITUTION.md の意味（Why/What/How の3層） → skills → プロンプトのコツ。**Claude Code 公式ドキュメントを参照しつつ、筆者オリジナルの運用（CONSTITUTION.md、8事業横断運用）を混ぜる**
- ソース: git (WebSite/MeWriteDocs `44d0f5f Two-File Model化しCONSTITUTION追加`) + 全事業で運用中（AGENTS.md「CONSTITUTION.md is authoritative」） + 公式 docs（要リサーチ: note-research で code.claude.com/docs を当たる）
- ラベル: ストック型寄り（「Claude Code 使い方」検索）+ フロー型
- パターン: How-to + 主張型
- 読者メリット: CLAUDE.md が肥大化して困っている人・公式 docs を読み切れていない人向けの整理。公式の機能一覧ではなく「何をどこに書くか」の設計基準
- 一次情報度: 中〜高（公式参照部分は中、CONSTITUTION 運用は高） / エバーグリーン度: 中（Claude Code の更新で陳腐化リスク）
- ピックアップ狙い: 条件付き（不足: ビフォーアフター具体例）
- 判断3条件: 実体験[○] 読者メリット[○] 差別化[要確認]（入門ガイドは競合多。CONSTITUTION.md 層の独自性で差別化）
- 類似実績: 入門・ガイド系は中堅（壊して覚えるClaudeCode入門 46PV、スキル損 120PV、自作スキル10個捨てた 118PV）
- 検討メモ: 1本に詰めると長大。「設定ファイル編」「skills編」「プロンプト編」の分割も視野

### 候補 #11: MCPは廃れると思う。AIエージェントのツール連携はCLIで十分
- ソース: session (agents/dev-crew) — 「MCPは廃れると思う。CLIベース。ToolUseを提供するという意味ではMCPでもいいんだけど、それなら、ToolUseの説明書を提供すると思う」。実践: codex / gemini を MCP でなく CLI (`codex exec`, `gemini -p`) で統合した dev-crew の設計
- ラベル: フロー型（主張型・議論喚起）
- パターン: 主張型
- 核となる主張: ツール連携に必要なのはプロトコルではなく「説明書 + 実行可能なCLI」。AIはmanとhelpが読める。MCPサーバを書くよりCLIを置く方が速い
- 根拠となる実体験: dev-crew の codex/gemini 統合を MCP なしで実装・運用（competitive review が CLI 呼び出しで回っている）
- 読者メリット: MCP サーバを書くべきか迷っている開発者への判断材料
- 一次情報度: 高 / エバーグリーン度: 中（MCP の趨勢次第。旬は今）
- ピックアップ狙い: 条件付き（不足: 画像。アーキ図で補える）
- 判断3条件: 実体験[○] 読者メリット[○] 差別化[○に近い]（逆張り主張 + 実運用の裏付き）
- 注意: 煽りタイトルに寄せすぎない。「MCPを使うべき場面」も併記して知的誠実性を保つ

### 候補 #12: Claude Codeのスキルを、CodexとGeminiからも使えるようにした（.agents/skills と symlink 設計）
- ソース: session + git (agents/dev-crew `sync-skills`) — .claude / .codex / .agents の互換問題を調査し、`.agents/skills/{skill_name}` への symlink 方式に着地。AGENTS.md 標準への追従
- ラベル: ストック型寄り（「AGENTS.md Codex skills」検索）
- パターン: 体験記 + How-to
- 読者メリット: Claude Code / Codex / Gemini CLI を併用する人向け。スキル資産を複数エージェントで共有する具体的方法と、試して駄目だった案（.codex を .claude の symlink にする等）
- 一次情報度: 高 / エバーグリーン度: 中（.agents 仕様の動きが速い）
- ピックアップ狙い: -
- 判断3条件: 実体験[○] 読者メリット[要確認]（併用者がまだ少数） 差別化[○に近い]
- 関連: 候補 #11 と世界観が繋がる（CLI ベース統合の各論）

### 候補 #13: Claude Codeのcronだけで回る全自動Webメディアを3ヶ月運用した
- ソース: git (WebSite/KaigaiHoudou) — `d8b56b2 記事自動収集 (heartbeat-pipeline初回実行)` から毎日の `feat: add articles` 自動コミット + フォーカス「CronCreate全自動パターンをMeWrite系に横展開。Hugo差分デプロイ対応」
- ラベル: フロー型
- パターン: 体験記
- 読者メリット: Claude Code の CronCreate / heartbeat 運用の実例。人間の作業ゼロでコンテンツパイプラインが回る構成と、止まった・壊れたポイント
- 一次情報度: 高 / エバーグリーン度: 中
- ピックアップ狙い: 条件付き（不足: 収益・PV実績の数字。AdSense 実績が弱いと「動いただけ」になる）
- 判断3条件: 実体験[○] 読者メリット[要確認] 差別化[要確認]
- 注意: 「AI全自動生成サイト」は読者の心象が分かれるテーマ。品質管理ゲートの説明を厚めにして信頼性資産を守る（倫理フィルタ自体は通過: スクレイピング回避等のグレー手法なし）

### 候補 #14: 10歳の息子に「自律と恥」を話したら、納得した顔をした
- **ステータス: 統合済み（2026-06-14）。6/08 設計編ドラフトに反応編を接合し `note/20260608_10歳の息子に月1回の授業を始めた.md` を「設計→実施→反応」の完結版に更新。タイトルを「始めた。初回『自律と恥』をやってみた」に変更**
- 反応編の核: 派手な反応はなし。「2種類の恥」と「10〜18は準備期間」の2点に納得した顔。特別な一言はなし（誇張せず記述）
- ソース: 筆者の実体験（2026-06-14 報告）+ 既存ドラフト `note/20260608_*.md`「10歳の息子に、月1回の授業を始めることにした。初回は『自律と恥』」(draft)
- ラベル: フロー型（子育て・エンゲージ）
- パターン: 体験記
- 核となる体験:
  - 自律と恥について子供に話したら、理解して納得したような表情をした
  - 筆者の見立て: 特に10〜18歳は「大人になるための準備期間」で、子供は変わっていかなければいけない。その自覚を恥（自律の裏返し）として持たせる
  - エリクソンの発達段階「自律性 対 恥・疑惑」を、月1授業という形で10歳に伝える試み（初回テーマ）
- 読者メリット: 子育て中の親に向けた「抽象的な概念（自律・恥）を子供にどう伝えるか」の実例。発達段階を意識した対話の記録
- 一次情報度: 高（完全な一次体験） / エバーグリーン度: 高（子育て論は普遍的）
- ピックアップ狙い: 可能（独自体験 / 対象読者と結論を冒頭明示可。画像は要工夫＝授業のホワイトボード等）
- 判断3条件: 実体験[○] 読者メリット[○] 差別化[○に近い]（AI/技術系筆者が子育ての発達段階論を書く意外性）
- 類似実績: 子育て系は安定して読まれる（「ヤバい」しか言えない 30PV/スキ5、ゲーム時間ポイント制 43PV/スキ5、9歳腕立て）。スキ率が高い＝エンゲージ強め
- 既存ドラフトとの関係: 6/08 ドラフトは「授業を始める」という枠組みの話。本ネタは「初回授業の中身と子供の反応」。ドラフトに統合 or 続編のどちらか。**書く前に 6/08 ドラフトの内容を確認すること**

## 不採用（2026-06-12 筆者判断）

- 候補 #2: 競馬AI ROI100%断念 → 書かない（Keiba事業はnote記事の題材にしない方針）
- 候補 #4: 「ツール呼び出し記法が間違っている」20回 → 未解決のため記事にならない
- 候補 #5: Vercel Hobby cron上限 → 書かない
- 候補 #6: カクヨム連載AIパイプライン → 書かない
- 候補 #8: 50記事PV分析データ公開 → 不要
- 候補 #10: 並列レビューの prompt 契約 + Findings Synthesis → 微妙（読者層が薄い）
