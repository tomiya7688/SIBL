# SIBL

**SIBL** は **Simple Interpreter Block Language** の略です。

SIBL は、アプリケーションへ軽量に組み込めるユーザースクリプト向けインタプリタ言語です。
インタプリタごと配布しやすく、ホストアプリケーションとの連携を単純に保つことを目的とします。

## 目的

SIBL は次の用途を想定します。

- アプリケーションのユーザースクリプト
- 軽量な組み込み用インタプリタ
- インタプリタ本体ごとの配布
- JSON メッセージによるホストアプリケーション連携
- 配布時にホスト API 定義を固定する実行環境

## 基本方針

SIBL はソースコードを上から順に実行します。
ただし、SIBL における「1行」は物理的な改行ではなく、構文的に完結した **1つのブロック** です。

```sibl
let name = "SIBL"

show_message("Hello, " + name)

if true {
    show_message("block executed")
}
```

この例では、`if true { ... }` 全体が1つの実行ブロックです。

## ブロック

SIBL では、複数命令をまとめる場合や、内部処理を持つ構文で `{ ... }` を使います。

```sibl
if x > 0 {
    show_message("positive")
}
```

インタプリタは `{` と `}` の対応を見て、ブロックが完結したかを判断します。

## 事前解析

SIBL は事前解析を ON/OFF できます。

- 事前解析 ON: 実行前に明らかな構文エラーやホスト関数定義の不一致を検出します。
- 事前解析 OFF: ブロックが完成した時点で順次実行し、到達した時点でエラーを出します。

事前解析 OFF の場合、閉じられていないブロックはファイル終端まで読んだあと、ランタイムエラーになります。

## Host integration

ホスト関数は `embedded-Settings.json` で定義します。

```json
{
  "language": {
    "preAnalyze": true,
    "maxSteps": 10000
  },
  "hostFunctions": {
    "show_message": {
      "args": ["string"],
      "return": "void"
    }
  }
}
```

SIBL インタプリタは、ホスト関数呼び出しを JSON メッセージとしてホストアプリケーションへ送信できます。

```json
{
  "type": "host.call",
  "id": 1,
  "name": "show_message",
  "args": ["Hello"]
}
```

## Embedded lock

配布時には `embedded-Settings.json` をインタプリタ内に取り込み、外部設定による上書きを禁止できます。
この機能を **embedded lock** と呼びます。

embedded lock により、配布用インタプリタで利用可能なホスト関数、型、実行制限、事前解析設定を固定できます。

## Repository structure

```text
docs/
  language-purpose.md
  execution-model.md
  embedded-settings.md
  embedded-lock.md
examples/
  hello.sibl
  block-execution.sibl
  host-call.sibl
schemas/
  embedded-Settings.schema.json
```

## Status

Early design.
