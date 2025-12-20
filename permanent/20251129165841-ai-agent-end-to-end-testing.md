---
id: 2025-11-29-165841
title: AIエージェントにおけるエンドツーエンドテスト
created: 2025-11-29 16:58:41
updated: 2025-11-29 16:58:41
tags:
  - zettel
  - ai
  - testing
  - end-to-end
  - browser-automation
source:
  type: article
  ref: "Anthropic - Effective harnesses for long-running agents"
---

## 要約（Summary）

- エージェントはコード変更後にエンドツーエンドテストを実施し、機能を検証。
- ブラウザ自動化ツール（Puppeteer）を使って人間のようにテスト。
- これにより、隠れたバグを検知し、品質を確保。

## 本文（Body）

### 背景・問題意識

エージェントはユニットテストは行うが、エンドツーエンドで機能するか確認しない傾向がある。これにより、完成と見せかけてバグが残る。

### アイデア・主張

各セッションでブラウザテストを実施。フィーチャーを「passing」にマークするのはテスト成功後。

### 具体例・ケース

claude.aiクローンで、チャット送信と応答受信をPuppeteerでテスト。

### 反論・限界・条件

ブラウザネイティブのモーダルは検知しにくい。ツールの限界を考慮。

## 関連ノート（Links）

- [[20251129165840-feature-list-task-management|フィーチャーリストによるタスク管理]] フィーチャーリストとの連携
- [[20251129160319-ai-guardrails|AI開発におけるガードレールの重要性]] ガードレール

## To-Do / 次に考えること

- [ ] ブラウザ自動化ツールの導入を検討
- [ ] テストスクリプトの作成方法を学ぶ</content>
<parameter name="filePath">