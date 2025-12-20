---
id: 2025-11-29-165840
title: フィーチャーリストによるタスク管理
created: 2025-11-29 16:58:40
updated: 2025-11-29 16:58:40
tags:
  - zettel
  - ai
  - task-management
  - feature-list
  - json
source:
  type: article
  ref: "Anthropic - Effective harnesses for long-running agents"
---

## 要約（Summary）

- フィーチャーリスト（JSON形式）は、全機能を「failing」状態で定義し、タスクの全体像を提供。
- エージェントが完了を誤認せず、段階的に進捗できる。
- テスト後にのみステータスを「passing」に変更。

## 本文（Body）

### 背景・問題意識

高レベルプロンプトだけでは、エージェントが何を実装すべきか把握しにくい。リストがないと、早期完了や見落としが発生。

### アイデア・主張

Initializerがフィーチャーリストを作成し、Coding agentが1つずつ完了。これにより、構造化されたタスク管理が可能。

### 具体例・ケース

claude.aiクローンで200フィーチャーをリスト。エージェントが順次実装。

### 反論・限界・条件

リストが不完全だと効果がない。MarkdownよりJSONの方が改変されにくい。

## 関連ノート（Links）

- [[20251129165839-coding-agent-incremental-progress|Coding agentによるインクリメンタル進捗]] インクリメンタル進捗との連携
- [[20251129160320-ai-task-granularity|AIへのタスク粒度と効率の関係]] タスク粒度

## To-Do / 次に考えること

- [ ] プロジェクトのフィーチャーリスト作成を習慣化
- [ ] JSON形式の利点を検証
