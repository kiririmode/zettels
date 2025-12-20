---
id: 20251129172153
title: OpenTelemetry Temporalityの基本概念
created: 2025-11-29 17:21:53
updated: 2025-11-29 17:21:53
tags:
  - zettel
  - opentelemetry
  - metrics
  - temporality
source:
  type: article
  ref: "https://grafana.com/blog/2023/09/26/opentelemetry-metrics-a-guide-to-delta-vs.-cumulative-temporality-trade-offs/"
---

## 要約（Summary）

- OpenTelemetryメトリクスにおけるtemporalityは、測定値が時間に対してどのように報告されるかを指す。
- Cumulativeは測定開始からの累積値、Deltaは前回報告からの差分を報告する。
- どちらを選択するかはバックエンドのサポートとユースケースによる。

## 本文（Body）

### 背景・問題意識

OpenTelemetryでは、メトリクスのデータポイントが時間軸上でどのように表現されるかをtemporalityと呼ぶ。これにより、カウンターなどの加算測定がどのように扱われるかが決まる。

### アイデア・主張

- Cumulative temporality: 測定開始からの総計値を報告。例えばリクエスト数カウンターが[t0, 5], [t0+10, 10]のように累積する。
- Delta temporality: 前回報告からの差分を報告。例えば[t0, 5], [t0+10, 5]のように増分のみ。

### 具体例・ケース

カウンターでリクエスト数を測定する場合、Cumulativeでは時間が経つにつれて値が増え続ける。Deltaでは各間隔での増加分だけを送信。

### 反論・限界・条件

Deltaはデータ損失が発生しやすいが、メモリ効率が良い場合がある。Cumulativeはデータ損失が解像度低下に留まるが、メモリ使用が多い。

## 関連ノート（Links）

- [[20251129172154-backend-temporality-choice|バックエンドによるTemporality選択基準]] バックエンドによる選択基準
- [[20251129172155-temporality-conversion-challenges|Temporality変換の課題と方法]] 変換の課題
- [[202511291430-delta-to-cumulative-processor|OpenTelemetry DeltaToCumulative Processorの概要]] DeltaToCumulative Processorの概要
- [[202511291440-load-balancing-exporter|OpenTelemetry Collector Load Balancing Exporterの概要]] スケーリングのためのload balancing

## To-Do / 次に考えること

- [ ] OpenTelemetry SDKでのtemporality設定方法を調査
- [ ] 自プロジェクトでのメトリクス収集に適用検討