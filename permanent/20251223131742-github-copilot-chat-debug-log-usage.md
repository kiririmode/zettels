---
id: 2025-12-23-131742
title: GitHub Copilot Chat Debug logの使い方と機密情報レビュー
created: 2025-12-23 13:17:42
updated: 2025-12-23 13:17:42
tags:
  - zettel
  - github-copilot
  - vscode
  - debugging
  - troubleshooting
  - security
source:
  type: web
  ref: "VS Code GitHub Copilot Chat Debug log に関する説明"
---

## 要約（Summary）

- VS CodeのGitHub Copilot Chat Debug viewは、LLMに送ったプロンプトとレスポンスの詳細を確認できるデバッグツール
- `Developer: Show Chat Debug View` で開き、`panel/askAgent` や `panel/editAgent` ノードから詳細を確認・エクスポート可能
- ログにはファイル内容やターミナル出力などの機密情報が含まれるため、共有前の必須レビューが重要

## 本文（Body）

### 背景・問題意識

GitHub Copilot Chat（チャット/エージェント機能）で問題が発生した場合、通常のOutputログでは不十分なことがある。特に以下のような症状では、LLMに何が送られ、何が返ってきたかの詳細確認が必要：

- Copilotの返答がおかしい
- 文脈が伝わっていない
- Agentが途中で止まる
- 編集内容が意図と異なる
- ツール実行が変

このような場合、**Chat Debug log（Chat Debug view）**を使うことで、実際のLLMリクエスト/レスポンスの中身を追うことができる。

### アイデア・主張

**GitHub Copilot Chat Debug logは、AI挙動の問題を診断する強力なツールだが、機密情報を含むため共有時には慎重な取り扱いが必須である。**

#### Chat Debug logで見られる情報

- **LLMに送信されたプロンプト全文**
- **付与されたコンテキスト**（ファイル内容、ターミナル出力等）
- **LLMからのレスポンス詳細**
- **リクエスト単位の詳細情報**

これらは、通常のOutputログ（GitHub Copilot / GitHub Copilot Chatチャンネル）では確認できない。

#### 具体的な使い方

**ステップ1: Chat Debug viewを開く**
1. Copilot Chatで適当に1回やり取りする（ログを生成するため）
2. コマンドパレット（`Cmd+Shift+P` / `Ctrl+Shift+P`）を開く
3. `Developer: Show Chat Debug View` を実行

**ステップ2: 該当ノードを探す**
- Chat Debug viewにツリー構造でログが表示される
- 末尾あたりの以下のノードを探す：
  - `panel/askAgent`: チャットでの質問・やり取り
  - `panel/editAgent`: エディタ上での編集操作
  - その他のエージェント操作

**ステップ3: 詳細を確認**
- ノードを選択すると、そのリクエスト/レスポンスの詳細が表示される
- プロンプト、コンテキスト、LLMの応答などを確認

**ステップ4: 必要に応じてエクスポート**
- ノードを右クリック
- `Export As…` を選択
- ログファイルとして保存

#### OutputログとChat Debug logの使い分け

| 症状の種類 | 使うべきログ | 確認内容 |
|----------|------------|---------|
| Copilotが起動しない | Output（GitHub Copilot） | 認証トークン、拡張機能のロード状態 |
| チャットが応答しない | Output（GitHub Copilot Chat） | 接続エラー、タイムアウト、リクエストID |
| 返答の内容がおかしい | Chat Debug log | 送信されたプロンプト、コンテキスト |
| 編集が意図と違う | Chat Debug log | エージェントのツール呼び出し、判断過程 |
| 文脈が伝わっていない | Chat Debug log | 付与されたコンテキストの内容 |

**判断基準**:
- 「動かない・接続できない」→ Outputログ
- 「返答がおかしい・文脈が変」→ Chat Debug log

#### 機密情報レビューの必須性

**⚠️ 重要な注意事項**

VS Codeの公式ドキュメントには、以下の警告が明記されている：

> このログには **ファイル内容やターミナル出力などの個人情報/機密が含まれ得る** ので、共有前に必ずレビューしてください。

**ログに含まれる可能性がある機密情報：**
1. **クレデンシャル**: API キー、アクセストークン、パスワード、データベース接続文字列
2. **個人情報（PII）**: ユーザー名、メールアドレス、テストデータ内の個人情報
3. **企業秘密**: 未公開機能のコード、ビジネスロジック、アーキテクチャ設計
4. **インフラ情報**: ホスト名、IPアドレス、内部ネットワーク構造

**共有前の必須チェックリスト：**
- [ ] ログ全体を読み、機密情報の有無を確認
- [ ] API キー、トークン、パスワード等を `***REDACTED***` で置換
- [ ] 個人情報（名前、メール等）をマスキング
- [ ] 企業秘密や未公開情報を削除
- [ ] 問題の再現に必要な最小限の情報のみを抽出
- [ ] 第三者の視点で「これを見られて問題ないか」を再確認

### 内容を視覚化するMermaid図

```mermaid
graph TB
    subgraph "GitHub Copilot デバッグログの種類"
        A["問題発生"] --> B{"症状の分類"}

        B -->|"接続・認証<br/>起動しない"| C["Outputログ"]
        B -->|"AI挙動<br/>返答がおかしい"| D["Chat Debug log"]

        C --> E["確認内容:<br/>・認証トークン<br/>・接続状態<br/>・リクエストID<br/>・エラーログ"]
        D --> F["確認内容:<br/>・送信プロンプト<br/>・付与コンテキスト<br/>・LLMレスポンス<br/>・ツール呼び出し"]
    end

    subgraph "Chat Debug log の使い方"
        G["1. コマンドパレット"] --> H["Developer: Show<br/>Chat Debug View"]
        H --> I["2. ツリーから<br/>ノード選択"]
        I --> J["panel/askAgent<br/>panel/editAgent"]
        J --> K["3. 詳細確認"]
        K --> L{"エクスポート<br/>必要？"}
        L -->|Yes| M["右クリック<br/>Export As..."]
        L -->|No| N["画面で確認"]
    end

    subgraph "機密情報レビュープロセス"
        O["ログエクスポート"] --> P["全体を読む"]
        P --> Q{"機密情報<br/>あり？"}
        Q -->|Yes| R["マスキング<br/>削除"]
        Q -->|No| S["最小限に絞る"]
        R --> S
        S --> T["再確認"]
        T --> U{"安全？"}
        U -->|No| P
        U -->|Yes| V["共有OK"]
    end

    style D fill:#9cf
    style F fill:#9cf
    style V fill:#9f9
    style Q fill:#fc9
    style R fill:#faa
```

### 具体例・ケース

**ケース1: 公式の不具合報告フロー**

GitHub Copilotの公式Issue報告ガイドでは、以下の手順が案内されている：

1. `Developer: Show Chat Debug View` を開く
2. `panel/askAgent` や `panel/editAgent` のノードを選ぶ
3. 詳細を確認
4. 必要なら `Export As…` で保存
5. **⚠️ 個人情報/機密をレビュー**
6. Issueに添付

この手順により、報告者はLLMに何が送られたかを具体的に示すことができる。

**ケース2: 症状別の診断例**

**症状**: Copilotが同じファイルを何度も編集しようとする

1. Chat Debug view を開く
2. 該当の `panel/editAgent` ノードを選択
3. ツール呼び出しの引数を確認
4. → エージェントが受け取ったコンテキストに、ファイルパスが重複していることを発見
5. → 原因: ワークスペース設定の問題と判明

**症状**: チャットの返答が質問と無関係

1. Chat Debug view を開く
2. 該当の `panel/askAgent` ノードを選択
3. 送信されたプロンプトを確認
4. → プロンプトが意図と異なる形で構築されている
5. → 原因: 拡張機能のバグまたはバージョン問題

**ケース3: 機密情報の適切な処理**

**悪い例**（そのままエクスポート→共有）:
```json
{
  "context": {
    "files": [
      {
        "path": ".env",
        "content": "API_KEY=sk-1234567890abcdef\nDATABASE_URL=postgresql://user:pass@prod-db.internal:5432/main"
      }
    ]
  }
}
```

**良い例**（マスキング後）:
```json
{
  "context": {
    "files": [
      {
        "path": ".env",
        "content": "API_KEY=***REDACTED***\nDATABASE_URL=***REDACTED***"
      }
    ]
  }
}
```

または、問題に無関係な `.env` ファイル部分は完全に削除する。

### 反論・限界・条件

**ログが見つからないケース**

Chat Debug viewを開いても、該当するノードが見つからない場合がある：
- Copilotとのやり取りが記録されていない（セッションが古い）
- ログのツリーが深く、目的のノードを見つけにくい

**対応**:
- 問題を再現してから、すぐにChat Debug viewを確認
- ツリーを展開して、タイムスタンプで該当のやり取りを特定

**レビューの負担**

大量のログを手動でレビューするのは時間がかかる。緊急時には開発速度を下げる可能性がある。

**対応**:
- 自動マスキングツール（git-secrets、detect-secrets等）を事前に導入
- 社内での共有は比較的緩く、外部（GitHub Issue、サポート）への共有は厳格にレビュー
- 問題に関連する部分のみを抽出（5000行全体ではなく50行程度）

**ツールの進化**

VS CodeやGitHub Copilotの仕様は頻繁に変わるため、この手順が将来的に変更される可能性がある。

**成立条件**

この手順が有効に機能するためには：
- VS Codeの比較的新しいバージョンを使用している
- GitHub Copilot拡張機能が正しくインストールされている
- 開発者モードのコマンド（Developer: ...）が利用可能

## 関連ノート（Links）

- [[20251129165841-ai-agent-end-to-end-testing|AIエージェントにおけるエンドツーエンドテスト]] エージェントのテスト・検証手法
- [[20251215004321-ai-output-quality-check-responsibility|AI生成コンテンツの品質チェック責任]] AI出力の品質管理と責任
- [[20251214140418-effective-agent-config-design-principles|効果的なAIエージェント設定ファイル設計の原則]] エージェント設定とコンテキスト管理
- [[20251206000000-ai-coding-invisible-problems|AIコーディング時代の「見えない問題」]] AI開発における透明性・可視性の課題
- [[20251215004400-reader-centric-ai-usage|読み手負担を考慮したAI活用]] 情報共有時の責任と配慮

## To-Do / 次に考えること

- [ ] VS CodeでChat Debug viewを実際に開いて、操作に慣れる
- [ ] 過去のCopilot問題をChat Debug logで再診断してみる
- [ ] チーム内でログ共有時のレビューガイドラインを策定
- [ ] 自動マスキングツールの導入を検討
- [ ] Copilot問題発生時のトラブルシューティングチェックリストを作成
