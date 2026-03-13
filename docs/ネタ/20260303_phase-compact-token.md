# コンテキスト圧縮 (phase-compact + Cycle doc)

## 角度

既出の「トークン半減」記事とは角度を変えて、Cycle docによるフェーズ間永続化に焦点。
「コンテキストが溢れる」問題を、会話履歴ではなくファイルで解決した。

## 差別化

- 既出記事: SKILL.md圧縮とProgressive Disclosureによるトークン削減
- 今回: フェーズ境界でCycle docに状態を永続化し、/compactで会話を圧縮、次フェーズでCycle docから復元
- OpenClawインスパイアのphase-boundary compaction

## 構成案

1. 問題: TDDサイクルが長くなるとコンテキストウィンドウが溢れる
2. Cycle doc: 各フェーズの出力をMarkdownファイルに永続化
3. phase-compact: フェーズ境界で/compact実行を案内
4. reload: /compact後にCycle docからコンテキストを復元
5. plan mode → approve → auto-compact の自然な圧縮フロー
6. 効果: 中規模タスクでもコンテキスト溢れなしで完走

## キーメッセージ

「コンテキストウィンドウは制約ではなくフェーズの区切り。溢れるなら書き出して忘れればいい」

## 想定読者

Claude Codeで長いタスクを扱う開発者。コンテキスト管理に困っている人。

## 技術詳細（記事で触れる）

- Cycle docのフォーマット
- phase-compactスキルのトリガー条件
- reloadスキルの復元ロジック
- auto-compactの仕組み（plan mode approve後）
