# About this Skill

A Claude Code skill that generates wireframe from natural language descriptions.

Describe a screen layout in plain language, and the skill produces a JSON wireframe definition + a self-contained HTML preview you can open in any browser.

## How to Use

Use `/wireframe` command or type in natural language.

```
/wireframe dashboard with sidebar navigation and a data table
```


## What It Produces

Two files per screen:

| File | Purpose |
|------|---------|
| `{name}.wireframe.json` | Machine-readable layout definition |
| `{name}.wireframe.html` | Preview in browser |

## Why JSON

When you ask an LLM to wireframe, the default output is ASCII art. It's human-readable at a glance, but LLMs can't reliably parse it back into structure, and it's difficult to adjust programmatically. One misaligned character breaks the whole layout.

JSON solves this. It's a structured format that both humans and machines can read, edit, and reason about. And critically, AI-powered design tools like Figma Make and Google Stitch already accept JSON to render designs — so JSON wireframes become the bridge between conversation and visual output.

## HTML Preview Capabilities

The generated HTML file is **fully self-contained** — no server, no build step, no external dependencies. Just open it in a browser.

### Interactive Editing

- **Drag-and-drop reordering** — drag any element to reorder it within its parent container
- **Live JSON sync** — the JSON panel updates in real-time as you rearrange elements
- **Edit tracking** — an "Edited" badge appears when you've made changes

### Export Options

All export buttons are grouped in the JSON panel header:

| Action | Browser Support | Description |
|--------|----------------|-------------|
| **Copy** | All browsers | Copy JSON to clipboard |
| **Save** | Chrome / Edge | Write directly to a file (first save opens picker, then overwrites) |
| **Download** | All browsers | Standard browser download |

## Element Types

The wireframe JSON supports these element types, each rendered as a distinct placeholder:

| Type | Rendered As |
|------|------------|
| `text` | Gray label (with `variant` for hierarchy: display, heading, caption) |
| `image` | Gray box with image icon |
| `icon` | Small gray circle |
| `button` | Rounded rectangle |
| `link` | Underlined text |
| `input` | Bordered rectangle |
| `card` | Bordered box with shadow |
| `table` | Header row + placeholder data rows |
| `divider` | Thin horizontal line |
| _(none)_ | Transparent container (structural grouping) |

## Requirements

- **Python 3** — used to splice JSON into the HTML template (a one-line string replacement)
- **A web browser** — to view the HTML preview

No npm, no build tools, no external libraries.

## File Structure

```
.claude/skills/wireframe/
  SKILL.md                  # Skill definition and JSON schema
  wireframe-designer.md     # Cognitive model (7-phase design reasoning)
  wireframe-template.html   # Self-contained HTML template (~1000 lines)
  README.md                 # This file
```

---

# このスキルについて

自然言語の説明からワイヤーフレームを生成する Claude Code スキル。

画面レイアウトを言葉で説明すると、JSONワイヤーフレーム定義と、ブラウザで開けるHTML プレビューを生成します。

## 使い方

`/wireframe` とコマンドを入れるか、自然言語でも生成できます。

```
/wireframe サイドバーナビゲーションとデータテーブルのあるダッシュボード
```

## 生成されるファイル

画面ごとに2つのファイルが生成されます：

| ファイル | 用途 |
|---------|------|
| `{name}.wireframe.json` | 機械可読なレイアウト定義 |
| `{name}.wireframe.html` | ブラウザでプレビュー |

## なぜJSONか

LLMにワイヤーフレームを頼むと、デフォルトの出力はASCIIアートです。人間にはぱっと見で読めますが、LLMが構造として正確に解析し直すのは難しく、調整も困難です。1文字ずれるだけでレイアウト全体が崩れてしまいます。

JSONはこの問題を解決します。人間にも機械にも読み書きでき、推論可能な構造化フォーマットです。また、Figma MakeやGoogle StitchといったAIデザインツールがすでにJSONを受け取ってデザインを描画できること。JSONワイヤーフレームは、会話とビジュアルアウトプットをつなぐ橋渡しになります。

## HTMLプレビューの機能

生成されるHTMLファイルは**完全に自己完結**しています。サーバー不要、ビルドステップ不要、外部依存なし。ブラウザで開くだけです。

### インタラクティブ編集

- **ドラッグ&ドロップ並べ替え** — 要素をドラッグしてコンテナの並べ替え
- **JSONリアルタイム同期** — 要素の並べ替えに合わせてJSONパネルが即座に更新
- **編集トラッキング** — 変更を加えると「Edited」バッジが表示

### エクスポート

すべてのエクスポートボタンはJSONパネルヘッダーにまとめられています：

| アクション | ブラウザ対応 | 説明 |
|-----------|------------|------|
| **Copy** | 全ブラウザ | JSONをクリップボードにコピー |
| **Save** | Chrome / Edge | ファイルに直接保存（初回はピッカー表示、以降は上書き） |
| **Download** | 全ブラウザ | 標準のブラウザダウンロード |

## 要素タイプ

ワイヤーフレームJSONは以下の要素タイプをサポートし、それぞれ固有のプレースホルダーとして描画されます：

| タイプ | 描画 |
|-------|------|
| `text` | グレーのラベル（`variant` で階層指定: display, heading, caption） |
| `image` | 画像アイコン付きグレーボックス |
| `icon` | 小さなグレーの円 |
| `button` | 角丸の矩形 |
| `link` | 下線付きテキスト |
| `input` | 枠線付き矩形 |
| `card` | 影付き枠線ボックス |
| `table` | ヘッダー行 + プレースホルダーデータ行 |
| `divider` | 細い水平線 |
| _（なし）_ | 透明コンテナ（構造的グルーピング用） |


## 必要な環境

- **Python 3** — JSONをHTMLテンプレートに埋め込むために使用（1行の文字列置換のみ）
- **Webブラウザ** — HTMLプレビューの表示用

npm、ビルドツール、外部ライブラリは一切不要です。

## ファイル構成

```
.claude/skills/wireframe/
  SKILL.md                  # スキル定義とJSONスキーマ
  wireframe-designer.md     # 認知モデル（7フェーズのデザイン推論）
  wireframe-template.html   # 自己完結型HTMLテンプレート（約1000行）
  README.md                 # このファイル
```
