---
id: 20251129181131
title: deltatocumulative Processor の設定引数
created: 2025-11-29 18:11:31
updated: 2025-11-29 18:11:31
tags:
  - zettel
  - opentelemetry
  - grafana-alloy
  - configuration
  - max_stale
  - max_streams
source:
  type: article
  ref: "https://grafana.com/docs/alloy/latest/reference/components/otelcol/otelcol.processor.deltatocumulative/"
---

## 要約（Summary）

- `otelcol.processor.deltatocumulative` の主な引数は `max_stale` と `max_streams` で、ストリームの管理とリソース制御を行う。
- `max_stale` はストリームの有効期間を制御し、`max_streams` は追跡するストリーム数の上限を設定。

## 本文（Body）

### 背景・問題意識

Delta to Cumulative 変換では、過去の値を記憶する必要があり、無限にストリームを追跡するとメモリ消費が増大する。これを防ぐためのパラメータが必要。

### アイデア・主張

- `max_stale`: 新しいサンプルが来ない場合の待機時間（デフォルト 5m）。これを超えるとストリームを破棄。
- `max_streams`: 追跡するストリームの上限（デフォルト 非常に大きい数）。上限に達すると新しいストリームをドロップ。

### 具体例・ケース

- 高負荷環境では `max_streams` を低く設定してメモリを節約。

### 反論・限界・条件

- `max_stale` は 0s より大きい値に設定する必要がある。短すぎるとデータ損失が増大。

## 関連ノート（Links）

- [[202511291450-deltatocumulative-spof-design|OpenTelemetry DeltaToCumulative ProcessorのSPOF回避設計]] SPOF 回避
- [[202511291440-load-balancing-exporter|OpenTelemetry Collector Load Balancing Exporterの概要]] ロードバランシング
- [[20251129172155-temporality-conversion-challenges|Temporality変換の課題と方法]] 変換課題

## To-Do / 次に考えること

- [ ] 自環境での最適な `max_stale` と `max_streams` の値を探る