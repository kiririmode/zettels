---
id: 20251129172158
title: Temporalityの最終推奨
created: 2025-11-29 17:21:58
updated: 2025-11-29 17:21:58
tags:
  - zettel
  - opentelemetry
  - metrics
  - recommendation
  - temporality
source:
  type: article
  ref: "https://grafana.com/blog/2023/09/26/opentelemetry-metrics-a-guide-to-delta-vs.-cumulative-temporality-trade-offs/"
---

## 要約（Summary）

- OpenTelemetryメトリクスのtemporality選択は、バックエンドのサポートを最優先に考える。
- バックエンドが両方をサポートする場合、Cumulative temporalityを推奨する。データ損失に対する耐性が高く、収集パイプラインがシンプルになるため。
- 低メモリモードはバックエンドがDeltaとCumulativeの両方を十分にサポートする場合にのみ使用し、そうでなければ避ける。

## 本文（Body）

### 背景・問題意識

temporalityの選択は、単なる好みではなく、エンドツーエンドの観測可能性パイプライン全体に影響を与える。バックエンドの制約、変換の複雑さ、メモリ使用、データ信頼性などを総合的に考慮する必要がある。

### アイデア・主張

- **バックエンド優先の原則**: 最も重要な指標はバックエンドのtemporalityサポート。PrometheusのようなCumulative専用バックエンドではCumulativeを、DatadogのようなDelta推奨バックエンドではDeltaを選択する。
- **Cumulative推奨の理由**: 両方サポート可能な場合、Cumulativeをデフォルトに。Deltaはサンプルドロップでデータ損失が発生するが、Cumulativeは解像度の低下に留まる。また、集約や変換がシンプルで、スケーリングしやすい。
- **低メモリモードの注意**: SDKの低メモリモード（同期インストゥルメントにDelta、非同期にCumulative）は、メモリ制約環境で有用だが、バックエンドが両temporalityを完全にサポートしないと問題になる。ほとんどのユースケースで実用的利点は限定的。

### 具体例・ケース

- Prometheusバックエンドを使用する場合: Cumulativeを選択。変換不要で安定。
- 複数バックエンド（例: PrometheusとDatadog）へ送信する場合: アプリケーションでCumulativeを設定し、CollectorでDeltaに変換。ただし、スケーリングが課題。
- サーバーレス環境（AWS Lambdaなど）: Deltaが適するが、データ損失リスクを許容できる場合のみ。

### 反論・限界・条件

- Deltaの利点: メモリリークしにくく、サーバーレスや高カーディナリティ環境に適するが、Collectorダウン時のデータ損失が致命的。
- 変換の限界: Cumulative to Deltaはスケーリング可能だが、Delta to Cumulativeはout-of-order問題や遅延データで不完全。60秒バッファで大部分をカバーするが、完全ではない。
- 適用条件: バックエンド変更時はアプリケーション再起動が必要。複数バックエンド時は変換パイプラインの複雑さを避けるため、単一temporalityに統一することを検討。

## 関連ノート（Links）

- [[20251129172157-memory-usage-tradeoffs]] メモリ使用のトレードオフ
- [[20251129172153-opentelemetry-temporality-definition|OpenTelemetry Temporalityの基本概念]] Temporalityの基本概念
- [[20251129172154-backend-temporality-choice|バックエンドによるTemporality選択基準]] バックエンドによる選択基準
- [[202511291430-delta-to-cumulative-processor|OpenTelemetry DeltaToCumulative Processorの概要]] DeltaToCumulative Processorの概要

## To-Do / 次に考えること

- [ ] 現在のプロジェクトのバックエンド（例: Prometheus, Grafana Mimirなど）のtemporalityサポートを確認
- [ ] 複数バックエンド使用時の変換パイプライン設計を検討
- [ ] サーバーレス環境でのDelta使用のリスク評価を実施