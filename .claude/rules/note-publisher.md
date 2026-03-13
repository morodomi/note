# note-publisher

note.com記事のPlaywrightベース自動投稿ツール。

## 場所

```
automation/note-publisher/
```

## 環境設定

`.env`ファイル（実行ディレクトリの`.env`、通常は`note/.env`）:

```
NOTE_EMAIL=xxx@example.com
NOTE_PASSWORD=xxxxxxxxx
```

セッションは`data/note_session.json`にキャッシュされる。期限切れ時は自動で再ログインする。

## CLIコマンド

### 記事投稿

```bash
# 下書き保存（デフォルト）
note-publish post --file <mdファイルパス> --draft --verbose

# 公開
note-publish post --file <mdファイルパス> --publish --verbose

# サムネイル付き
note-publish post --file <mdファイルパス> --thumbnail <画像パス> --draft

# ハッシュタグ上書き（md内のタグを無視）
note-publish post --file <mdファイルパス> --hashtags "タグ1,タグ2"

# .envファイル・セッションファイルのパスを指定
note-publish post --file <mdファイルパス> --env-file <.envパス> --session <セッションJSONパス>
```

`--file`は実行時のカレントディレクトリからの相対パス。note/ディレクトリから実行する前提。

| オプション | デフォルト | 説明 |
|-----------|-----------|------|
| `--env-file <path>` | `.env` | `.env`ファイルのパス（CWD相対） |
| `--session <path>` | `data/note_session.json` | セッションJSONファイルのパス（CWD相対） |

### サムネイル生成

```bash
note-publish thumbnail --title "記事タイトル" --output images/thumb.png
```

`rsvg-convert`が必要（`brew install librsvg`）。

### セッション更新

```bash
note-publish session --refresh --verbose

# .envファイル・セッションファイルのパスを指定
note-publish session --refresh --env-file <.envパス> --session <セッションJSONパス>
```

sessionコマンドも`--env-file`/`--session`オプションに対応（デフォルト値はpostコマンドと同じ）。

## Markdownファイルのフォーマット

```markdown
# 記事タイトル

本文（Markdown）

## ハッシュタグ候補
タグ1, タグ2, タグ3
```

### パース規則

| 要素 | ルール |
|------|--------|
| タイトル | 先頭の`# `行。本文からは除去される |
| 本文 | タイトル行〜ハッシュタグセクションの間。前後の空行はトリム |
| ハッシュタグ | `## ハッシュタグ候補`以降。カンマ区切り。先頭`#`は自動除去 |

### ハッシュタグセクション

- 見出しは正確に`## ハッシュタグ候補`（完全一致）
- 本文の末尾に置くこと（これ以降は全てタグ扱い）
- カンマ区切り: `タグ1, タグ2, タグ3`
- `#`付きでもOK: `#タグ1, #タグ2`（自動除去される）

## 本文入力の仕組み

ProseMirrorエディタへのクリップボードペースト方式。

1. エディタ領域をクリック
2. `navigator.clipboard.writeText()`でmarkdownをクリップボードに書き込み
3. `Cmd+V`（macOS）/ `Ctrl+V`（Linux/Windows）でペースト
4. ProseMirrorがmarkdownを一括パース

### normalizeBody（ペースト前の正規化）

以下の変換がペースト前に適用される:

| パターン | 変換 | 理由 |
|---------|------|------|
| `# ～ ###### 見出し\n\n` | `# ～ ###### 見出し\n` | 見出し（h1-h6）直後の空行を除去。ProseMirrorが見出し後に余分な段落を作るのを防ぐ |
| `---\n\n` | `---\n` | 水平線直後の空行を除去。同上 |

通常の段落間の空行（`段落1\n\n段落2`）はそのまま保持される。

## 記事で避けるべきパターン

| NG | 理由 |
|----|------|
| 関連リンクのURL直書き | noteエディタで表示されない。noteの埋め込み機能を使うか、削除する |
| HTMLタグ | ProseMirrorのmarkdownパーサーが処理するため、生HTMLは非対応 |

## トラブルシューティング

### エラー時スクリーンショット

投稿失敗時、`data/error-{timestamp}.png` にスクリーンショットが自動保存される。デバッグ時はこのファイルを確認すること。
