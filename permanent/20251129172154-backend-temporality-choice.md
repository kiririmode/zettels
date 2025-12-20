---
id: 20251129172154
title: バックエンドによるTemporality選択基準
created: 2025-11-29 17:21:54
updated: 2025-11-29 17:21:54
tags:
  - zettel
  - opentelemetry
  - metrics
  - backend
  - temporality
source:
  type: article
  ref: "https://grafana.com/blog/2023/09/26/opentelemetry-metrics-a-guide-to-delta-vs.-cumulative-temporality-trade-offs/"
---

## 要約（Summary）

- OpenTelemetryメトリクスのtemporality選択は、主にバックエンドのサポートに依存する。
- PrometheusはCumulativeのみ、DatadogはDeltaを推奨、一部は両方をサポート。
- バックエンド変更時にアプリケーション再設定が必要になる場合がある。

## 本文（Body）

### 背景・問題意識

メトリクスバックエンドが異なるtemporalityをサポートするため、アプリケーション側でtemporalityを設定する必要がある。複数バックエンドへの送信時は変換が必要。

### アイデア・主張

バックエンドの推奨temporalityを採用するのが最適。例: Prometheus/Grafana MimirはCumulative、DatadogはDelta。

### 具体例・ケース

Prometheusを使用する場合、Cumulativeを選択。Datadogへ送信時はDeltaに変換。

### 反論・限界・条件

複数バックエンドで異なるtemporalityが必要な場合、Collectorでの変換が複雑になる。

## 関連ノート（Links）

- [[20251129172153-opentelemetry-temporality-definition|OpenTelemetry Temporalityの基本概念]] Temporalityの基本
- [[20251129172155-temporality-conversion-challenges|Temporality変換の課題と方法]] 変換方法
- [[202511291430-delta-to-cumulative-processor|OpenTelemetry DeltaToCumulative Processorの概要]] DeltaToCumulative Processorの概要

## To-Do / 次に考えること

- [ ] 使用中のバックエンドのtemporalityサポートを確認