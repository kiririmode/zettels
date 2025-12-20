---
id: 2025-11-29-165839
title: Coding agentによるインクリメンタル進捗
created: 2025-11-29 16:58:39
updated: 2025-11-29 16:58:39
tags:
  - zettel
  - ai
  - agent
  - incremental-progress
  - coding
source:
  type: article
  ref: "Anthropic - Effective harnesses for long-running agents"
---

## 要約（Summary）

- Coding agentは、各セッションで1つのフィーチャーだけを進捗させ、クリーンな状態を残す。
- gitコミットと進捗ログを更新し、次のセッションの基盤を整える。
- これにより、セッション間の混乱を防ぎ、効率を高める。

## 本文（Body）

### 背景・問題意識

エージェントが一度に多くのことをやろうとすると、コードが中途半端になり、次のセッションで復旧に時間がかかる。インクリメンタルアプローチがソフトウェア開発のベストプラクティス。

### アイデア・主張

各セッションで1フィーチャーを完了させ、テストで検証。クリーンなコードをコミットすることで、長期的な保守性を確保。

### 具体例・ケース

Webアプリ開発で、チャット機能の実装を1セッションで完了。次のセッションで別の機能を追加。

### 反論・限界・条件

フィーチャーが大きすぎるとインクリメンタルが難しい。適切な粒度設定が必要。

## 関連ノート（Links）

- [[20251129165838-initializer-agent-environment-setup|Initializer agentによる環境初期化]] Initializerとの連携
- [[20251129160320-ai-task-granularity|AIへのタスク粒度と効率の関係]] タスク粒度との関連

## To-Do / 次に考えること

- [ ] インクリメンタル開発の手法を自分のワークフローに適用
- [ ] gitコミットメッセージのベストプラクティスを学ぶ</content>
<parameter name="filePath">/Users/kiririmode/Library/Mobile Documents/iCloud~md~obsidian/Documents/zettelkasten/permanent/20251129165839-coding-agent-incremental-progress.md