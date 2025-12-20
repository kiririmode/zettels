---
id: 202511291430-delta-to-cumulative-processor
title: OpenTelemetry DeltaToCumulative Processorの概要
created: 2025-11-29 14:30:00
updated: 2025-11-29 14:30:00
tags:
  - zettel
  - opentelemetry
  - metrics
  - temporality
  - processor
source:
  type: article
  ref: "Perplexity research on OpenTelemetry DeltaToCumulative processor"
---

## 要約（Summary）

- OpenTelemetryのDeltaToCumulative Processorは、Delta temporalityのメトリクスをCumulative temporalityに変換するコンポーネントである。
- PrometheusなどのCumulativeを要求するバックエンドとの互換性を確保しつつ、アプリケーションのメモリ効率を維持する。
- ステートフルな処理が必要で、スケーリングには特別なアーキテクチャが必要。

## 本文（Body）

### 背景・問題意識

OpenTelemetryでは、メトリクスのtemporality（Delta vs Cumulative）がアプリケーションの設計とバックエンドの要件でトレードオフを生む。Delta temporalityはメモリ効率が良く、ステートレスなアプリケーションに適するが、PrometheusなどのバックエンドはCumulativeを要求する。このギャップを埋めるために、DeltaToCumulative Processorが開発された。

### アイデア・主張

DeltaToCumulative Processorは、DeltaメトリクスをCumulativeに変換することで、アプリケーションがDeltaを使用しつつCumulativeバックエンドに送信できるようにする。Grafana Labsがコントリビュートしたこのプロセッサは、OpenTelemetry Collectorのcontribパッケージで利用可能。

### 具体例・ケース

- ステートレスプロセッサ（例: count connector）が生成するDeltaメトリクスをPrometheusに送信する場合。
- DatadogからPrometheusへのマイグレーション時に、アプリケーションの再計装を避ける。
- サーバレス環境で高カーディナリティのメトリクスを扱う。

### 反論・限界・条件

- ステートフルであるため、同じメトリクスシリーズが常に同じプロセッサインスタンスにルーティングされる必要があり、スケーリングが複雑。
- メモリ消費が高カーディナリティで問題化する可能性。
- プロセッサ再起動時に状態が失われ、時系列の不連続が発生。

## 関連ノート（Links）

- [[20251129172153-opentelemetry-temporality-definition|OpenTelemetry Temporalityの基本概念]] 基本概念の理解
- [[20251129172154-backend-temporality-choice|バックエンドによるTemporality選択基準]] バックエンド選択の基準
- [[20251129172155-temporality-conversion-challenges|Temporality変換の課題と方法]] 変換の課題
- [[20251129172158-temporality-final-recommendation|Temporalityの最終推奨]] 推奨事項
- [[20251129173333-delta-to-cumulative-detailed-analysis|Delta to Cumulative Temporality変換の詳細な分析]] 詳細な分析
- [[202511291440-load-balancing-exporter|OpenTelemetry Collector Load Balancing Exporterの概要]] スケーリングのためのload balancing
- [[202511291450-deltatocumulative-spof-design|OpenTelemetry DeltaToCumulative ProcessorのSPOF回避設計]] SPOF回避設計

## To-Do / 次に考えること

- [ ] 実際の設定例を試す
- [ ] 高カーディナリティ環境でのメモリ監視方法を調査
- [ ] 2-tierアーキテクチャのデプロイメントパターンを検証