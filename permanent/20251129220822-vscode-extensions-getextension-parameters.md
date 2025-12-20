---
id: 20251129220822
title: vscode.extensions.getExtensionのパラメータと戻り値
created: 2025-11-29 22:08:22
updated: 2025-11-29 22:08:22
tags:
  - zettel
  - vscode
  - extension-api
  - parameters
  - return-value
source:
  type: web
  ref: "https://code.visualstudio.com/api/references/vscode-api"
---

## 要約（Summary）

vscode.extensions.getExtension関数は、extensionIdという文字列パラメータを受け取り、Extension<T>またはundefinedを返す。extensionIdはpublisher.name形式で指定する。

## 本文（Body）

### 背景・問題意識

拡張APIを正確に呼び出すためには、パラメータの形式と戻り値の型を理解する必要がある。誤ったID指定でundefinedが返る場合の扱いも重要。

### アイデア・主張

パラメータはextensionIdのみで、戻り値はジェネリック型Extension<T>またはundefined。Tは拡張のエクスポート型を表す。

### 内容を視覚化するMermaid図

```mermaid
classDiagram
    class Extension {
        +id: string
        +extensionUri: Uri
        +extensionPath: string
        +isActive: boolean
        +packageJSON: any
        +exports: T
    }
    getExtension --> Extension : returns
    getExtension --> "undefined" : if not found
```

### 具体例・ケース

extensionId: "microsoft.python" → Python拡張を取得。戻り値がundefinedなら、拡張が未インストール。

### 反論・限界・条件

extensionIdが正しくない場合や拡張が無効化されている場合、undefinedが返る。アクティブ状態を確認するにはisActiveプロパティを使う。

## 関連ノート（Links）

- [[20251129220821-vscode-extensions-getextension-overview|vscode.extensions.getExtension APIの概要]]

## To-Do / 次に考えること

- [ ] Extensionオブジェクトの他のプロパティを調査する