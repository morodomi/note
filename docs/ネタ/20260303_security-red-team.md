# 13エージェント並列セキュリティスキャン

## 角度

既出の「首輪」記事とは角度を変えて、OWASP Top 10の具体的な実装に焦点。
13種類の攻撃エージェントが並列でスキャンし、false-positive-filterで誤検知を除去する。

## 差別化

- 既出記事: AIに首輪をつける（ルールベースの制約）
- 今回: AIに攻撃させる（13エージェントの具体的な攻撃手法と検出ロジック）
- OWASP Top 10のどの項目をどのエージェントがカバーしているか

## 構成案

1. 問題: セキュリティレビューを人間がやると時間がかかる、漏れる
2. 13攻撃エージェント一覧: injection, xss, csrf, ssrf, xxe, ssti, auth, file, crypto, sca, api, wordpress, error
3. recon-agent: 偵察フェーズで攻撃優先度を付ける
4. false-positive-filter: 静的解析の誤検知を自動除去
5. dynamic-verifier: 静的解析結果をPlaywrightで動的検証
6. attack-scenario: 脆弱性チェーンから具体的な攻撃手順を生成
7. 実行コスト: 並列実行のトークン消費

## キーメッセージ

「AIにセキュリティレビューさせるなら、防御側ではなく攻撃側をやらせろ」

## 想定読者

セキュリティに関心のある開発者。OWASP Top 10を知っているが自動化していない人。

## 技術詳細（記事で触れる）

- 各攻撃エージェントのプロンプト設計（静的解析ベース）
- recon-agentの優先度付けロジック
- false-positive-filterの判定基準
- security-scanスキルのオーケストレーション
