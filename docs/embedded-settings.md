# embedded-Settings.json

`embedded-Settings.json` は、SIBL インタプリタとホストアプリケーションの契約を表す設定ファイルです。

このファイルには、言語実行設定、ホスト関数定義、権限、JSON 通信プロトコルなどを記述します。

## 役割

`embedded-Settings.json` は、ホスト関数の実装を書くファイルではありません。
実装本体はホストアプリケーション側にあります。

このファイルには、スクリプトから利用できるホスト関数の公開情報を書きます。

```text
embedded-Settings.json
  ↓
この関数をスクリプトへ公開してよい、という宣言

Host application
  ↓
実際の関数実装
```

## 最小例

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

スクリプト側では次のように呼び出せます。

```sibl
show_message("Hello")
```

## language

`language` には、インタプリタの実行設定を書きます。

```json
{
  "language": {
    "preAnalyze": true,
    "maxSteps": 10000,
    "maxRecursionDepth": 100
  }
}
```

代表的な項目:

| key | type | meaning |
|---|---|---|
| `preAnalyze` | boolean | 実行前に事前解析するか |
| `maxSteps` | number | 最大実行ステップ数 |
| `maxRecursionDepth` | number | 最大再帰深度 |

## hostFunctions

`hostFunctions` には、スクリプトから呼び出せるホスト関数を定義します。

```json
{
  "hostFunctions": {
    "play_sound": {
      "args": ["string"],
      "return": "void"
    },
    "get_player_x": {
      "args": [],
      "return": "number"
    },
    "set_player_position": {
      "args": ["number", "number"],
      "return": "void"
    }
  }
}
```

スクリプト例:

```sibl
play_sound("jump")

let x = get_player_x()
set_player_position(x + 10, 20)
```

## 型

SIBL のホスト連携で扱う基本型は、JSON と相性の良い型に制限します。

```text
null
bool
number
string
array
object
any
void
```

最初の仕様では、次の値はホスト連携で扱わない方針です。

```text
関数そのもの
循環参照
バイナリ
NaN
Infinity
```

## ホスト呼び出しメッセージ

スクリプトでホスト関数が呼ばれた場合、インタプリタは JSON メッセージを作成できます。

```sibl
show_message("Hello")
```

送信例:

```json
{
  "type": "host.call",
  "id": 1,
  "name": "show_message",
  "args": ["Hello"]
}
```

ホストは処理結果を返します。

```json
{
  "type": "host.result",
  "id": 1,
  "value": null
}
```

戻り値がある場合:

```sibl
let x = get_player_x()
```

```json
{
  "type": "host.call",
  "id": 2,
  "name": "get_player_x",
  "args": []
}
```

```json
{
  "type": "host.result",
  "id": 2,
  "value": 120
}
```

## ホストエラー

ホスト関数の処理に失敗した場合、ホストはエラーを返せます。

```json
{
  "type": "host.error",
  "id": 3,
  "error": {
    "code": "FunctionError",
    "message": "sound file not found"
  }
}
```

インタプリタはこれをランタイムエラーとして扱います。

```text
RuntimeError: block 4: sound file not found
```

## 事前解析との関係

事前解析 ON の場合、インタプリタは `embedded-Settings.json` を使って次のような明らかな問題を検出できます。

- 存在しないホスト関数の呼び出し
- 引数の数が違う
- 明らかに型が違う
- `void` 関数の戻り値を式で使っている

事前解析 OFF の場合、これらは呼び出し時点のランタイムエラーになります。
