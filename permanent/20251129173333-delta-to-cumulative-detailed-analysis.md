---
id: 20251129173333
title: Delta to Cumulative Temporality変換の詳細な分析
created: 2025-11-29 17:33:33
updated: 2025-11-29 17:33:33
tags:
  - zettel
  - opentelemetry
  - metrics
  - temporality
  - delta-to-cumulative
  - conversion
source:
  type: web
  ref: "Perplexity research: Delta to Cumulative temporality conversion in OpenTelemetry metrics"
---

## 要約（Summary）

- OpenTelemetryにおけるDeltaからCumulativeへのtemporality変換は、状態管理が必要でスケーリングが難しい。
- 主な課題はout-of-orderデータ、データ損失、状態フルな処理。
- 変換方法としてCollectorのプロセッサやエクスポーターを使用し、60秒遅延バッファで対応。
- バックエンドの要件に合わせたtemporality選択が推奨され、変換は最終手段。

## 本文（Body）

### 背景・問題意識

Delta temporalityはメモリ効率が良くサーバーレス環境に適するが、PrometheusなどのバックエンドはCumulativeを要求する。変換は単なる数学的変換を超え、状態管理、スケーリング、信頼性の問題を引き起こす。

### アイデア・主張

- **変換の非対称性**: Delta to CumulativeはCumulative to Deltaより難しく、Collectorプロセッサとして未実装。エクスポーターで一部対応。
- **状態管理の必要性**: 各メトリクスの累積値を維持し、out-of-order到着を扱う。60秒遅延で大部分をカバー。
- **スケーリング課題**: 同じメトリクスを同じCollectorにルーティングする必要があり、ロードバランシングを使用。
- **データ損失リスク**: Deltaの欠点が変換で増幅され、Collectorダウンでデータが失われる。

### 具体例・ケース

- **E-commerceメトリクス**: DeltaカウンターをCumulativeに変換。プロセス再起動時のギャップ処理が必要。
- **ヒストグラム変換**: バケット値を累積し、min/maxの扱いに注意。
- **レート計算**: Deltaは直接レート計算可能、Cumulativeは微分が必要。

### 反論・限界・条件

- **out-of-order問題**: 遅延データで過去値を変更できない。TSDBで高コスト。
- **メモリ使用**: 高カーディナリティで状態管理がメモリを消費。
- **バックエンド互換**: PrometheusはCumulative推奨、New RelicはDelta。複数バックエンド時は変換が必要。
- **信頼性**: Cumulativeの方がデータ損失耐性が高い。

## 関連ノート（Links）

- [[20251129172155-temporality-conversion-challenges|Temporality変換の課題と方法]] 変換の課題と方法
- [[20251129173025-delta-to-cumulative-conversion]] Delta to Cumulative変換の詳細
- [[20251129172158-temporality-final-recommendation|Temporalityの最終推奨]] 最終推奨
- [[202511291430-delta-to-cumulative-processor|OpenTelemetry DeltaToCumulative Processorの概要]] DeltaToCumulative Processorの概要

## To-Do / 次に考えること

- [ ] CollectorのDelta to Cumulativeプロセッサの実装状況を確認
- [ ] 自プロジェクトでの変換パイプライン設計を検討
- [ ] バックエンドのtemporality要件を整理