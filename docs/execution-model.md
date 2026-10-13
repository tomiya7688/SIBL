# Execution model

SIBL は、ソースコードを上から順に実行するインタプリタ言語です。
ただし、SIBL における「1行」は、物理的な改行ではなく、構文的に完結した1つのブロックを意味します。

## 用語

```text
物理行 = ソースコード上の改行された1行
実行行 = インタプリタが1ステップとして扱う1ブロック
ブロック = 構文的に完結した実行単位
```

## 基本実行

単一命令は、それ自体が1ブロックです。

```sibl
let x = 10
show_message("hello")
```

この場合、実行ブロックは次の2つです。

```text
Block 1: let x = 10
Block 2: show_message("hello")
```

## `{ ... }` ブロック

複数命令をまとめる場合や、内部処理を持つ構文では `{ ... }` を使います。

```sibl
if x > 0 {
    show_message("positive")
    show_message("x is greater than zero")
}
```

この `if` 全体が1つの実行ブロックです。

## ブロック完成判定

インタプリタは `{` と `}` の対応を見て、ブロックが完成したかを判断します。

```text
{ が出たらブロック深さ +1
} が出たらブロック深さ -1
深さが 0 に戻ったらブロック完成
```

ネストしたブロックも同じ規則で扱います。

```sibl
if x > 0 {
    if x > 10 {
        show_message("large")
    }
    show_message("positive")
}
```

## 事前解析 OFF

事前解析 OFF の場合、インタプリタは次のように動きます。

```text
読む
↓
ブロックが完成したら即実行
↓
次のブロックを読む
↓
ファイル終端まで繰り返す
```

ブロックが閉じられていない場合、インタプリタは `}` が来るまで読み続けます。
ファイル終端に到達してもブロックが完成しない場合、ランタイムエラーになります。

```sibl
show_message("start")

if x > 0 {
    show_message("positive")

show_message("end")
```

この場合、最初の `show_message("start")` は実行されます。
その後、`if x > 0 {` のブロックが閉じられないまま EOF に到達するため、ランタイムエラーになります。

```text
RuntimeError: block 2: block is not closed before end of file
```

## 事前解析 ON

事前解析 ON の場合、インタプリタは実行前に全ブロックを確認します。

```text
全ブロックを読む
↓
明らかなエラーを確認
↓
エラーがなければ先頭から実行
```

事前解析で検出する対象は、明らかな構文エラー、未完了ブロック、存在しない命令、ホスト関数定義との明らかな不一致などです。

## エラー分類

SIBL のエラーは大きく2種類に分けます。

```text
AnalyzeError = 事前解析 ON の実行前エラー
RuntimeError = 実行中に発生するエラー
```

例:

```text
AnalyzeError: block 2: invalid syntax
RuntimeError: block 3: undefined variable 'x'
RuntimeError: block 4: division by zero
```

## REPL での扱い

REPL では、ブロックが閉じられていない状態をただちにエラーにせず、入力継続として扱えます。

```text
> if x > 0 {
... show_message("positive")
... }
```

ファイル実行では EOF に到達した時点で未完了ブロックをエラーにします。
