---
id: 202511291440-load-balancing-exporter
title: OpenTelemetry Collector Load Balancing Exporterの概要
created: 2025-11-29 14:40:00
updated: 2025-11-29 14:40:00
tags:
  - zettel
  - opentelemetry
  - collector
  - exporter
  - load-balancing
  - scaling
source:
  type: article
  ref: "Perplexity research on OpenTelemetry Collector load balancing exporter"
---

## 要約（Summary）

- OpenTelemetry CollectorのLoad Balancing Exporterは、一貫性のあるハッシュに基づいてテレメトリデータを複数のバックエンドに分散するコンポーネントである。
- ステートフルな処理（tail samplingやDeltaToCumulative変換）をスケール可能にする。
- 静的、DNS、Kubernetesのresolverをサポートし、traceIDやserviceなどのrouting keyを使用。

## 本文（Body）

### 背景・問題意識

OpenTelemetry Collectorでテレメトリデータを処理する際、単一インスタンスではスケーリングが限界を迎える。特に、tail samplingやメトリクスのtemporality変換のようなステートフルな処理では、同じトレースやメトリクスストリームが同じインスタンスに到達する必要がある。

### アイデア・主張

Load Balancing Exporterは、一貫性のあるハッシュリングを使用して、routing key（traceID、service、metric nameなど）に基づいてデータを決定論的に分散する。これにより、ステートフル処理を水平スケーリング可能にし、データの一貫性を維持する。

### 具体例・ケース

- Tail sampling: traceID routingで同じトレースの全spanが同じprocessorに到達。
- DeltaToCumulative: serviceやstreamID routingで同じメトリクスストリームが同じprocessorに到達。
- Kubernetes環境: DNS resolverで動的にバックエンドを検出。

### 反論・限界・条件

- バックエンド変更時のデータ損失リスク（DNS更新時の不整合）。
- 設定の複雑さ（resilience設定がnested exporterレベル）。
- 高負荷時のハッシュ計算オーバーヘッド。

## 関連ノート（Links）

- [[202511291430-delta-to-cumulative-processor|OpenTelemetry DeltaToCumulative Processorの概要]] DeltaToCumulative Processorとの関連
- [[20251129172153-opentelemetry-temporality-definition|OpenTelemetry Temporalityの基本概念]] Temporalityの基本
- [[20251129172155-temporality-conversion-challenges|Temporality変換の課題と方法]] 変換の課題
- [[202511291450-deltatocumulative-spof-design|OpenTelemetry DeltaToCumulative ProcessorのSPOF回避設計]] SPOF回避設計

## To-Do / 次に考えること

- [ ] Kubernetes環境でのDNS resolver設定を試す
- [ ] バックエンド変更時のデータ損失対策を調査
- [ ] メトリクスrouting keyのstreamIDオプションを検証