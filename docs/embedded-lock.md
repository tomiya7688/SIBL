# Embedded lock

**embedded lock** は、`embedded-Settings.json` の内容をインタプリタ本体へ取り込み、配布用に固定する機能です。

開発中は外部の `embedded-Settings.json` を読み、配布時にはその設定をインタプリタ内へ埋め込んでロックします。

## 目的

embedded lock の目的は、配布されるインタプリタの実行環境を固定することです。

```text
開発時:
  interpreter
  embedded-Settings.json
  script.sibl

配布時:
  locked-interpreter
  script.sibl
```

配布時は `embedded-Settings.json` を別ファイルとして持たなくても実行できます。
また、利用可能なホスト関数や実行制限を後から外部設定で差し替えられないようにできます。

## 固定される情報

embedded lock では、次のような情報を固定します。

- 公開されるホスト関数
- 各ホスト関数の引数型
- 戻り値型
- 権限設定
- 事前解析の既定値
- 最大ステップ数
- 最大再帰深度
- 使用可能な標準機能
- JSON 通信プロトコルのバージョン

## ロック済みインタプリタの挙動

embedded lock が有効なインタプリタでは、外部の `embedded-Settings.json` による設定上書きを禁止します。

外部設定ファイルが存在しない場合:

```text
ロック済み設定を使って実行する
```

外部設定ファイルが存在する場合:

```text
RuntimeError: external embedded-Settings.json is not allowed because embedded lock is enabled
```

外部ファイルを黙って無視するのではなく、エラーにする方針です。
理由は、利用者が「設定を書いたのに反映されない」と誤解するのを避けるためです。

## 内部メタデータ

ロック済みインタプリタは、内部的に次のような情報を持てます。

```json
{
  "embedded": {
    "locked": true,
    "settingsVersion": 1,
    "settingsHash": "sha256:..."
  }
}
```

`settingsHash` は、ホストアプリケーションとの整合性確認に使えます。

## ホストとのハンドシェイク

JSON メッセージでホストとやり取りする場合、起動時にハンドシェイクできます。

インタプリタからホストへ:

```json
{
  "type": "host.hello",
  "protocol": "sibl-host-json",
  "version": 1,
  "embeddedLocked": true,
  "settingsHash": "sha256:..."
}
```

ホストからの返答:

```json
{
  "type": "host.ready",
  "ok": true
}
```

設定が合わない場合:

```json
{
  "type": "host.error",
  "error": {
    "code": "SettingsMismatch",
    "message": "host API does not match embedded settings"
  }
}
```

## 注意

embedded lock は、設定を固定する機能です。
インタプリタのバイナリそのものを強力に保護する完全な改ざん防止機能ではありません。

より強い保護が必要な場合は、次の仕組みを組み合わせます。

- インタプリタ本体の署名検証
- 設定ハッシュの検証
- スクリプト署名
- ホスト側での API 制限

## 仕様文

```text
embedded lock は、embedded-Settings.json をインタプリタ本体へ取り込み、
実行時に設定を固定する機能である。

embedded lock が有効なインタプリタでは、外部 embedded-Settings.json による設定変更は禁止される。

スクリプトから利用できるホスト関数、型情報、権限、実行制限は、
ロック時に取り込まれた設定に従う。
```
