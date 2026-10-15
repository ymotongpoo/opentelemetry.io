---
title: 機能
description: >-
  主要なサイト機能の概要と、それぞれの主なリファレンスへのリンク。
weight: 20
default_lang_commit: 0f387a04d1d4029af5139505eb4c1f780ae238cc
cSpell:ignore: otelwebtelemetry
---

## エージェントフレンドリーなコンテンツ配信 {#agent-friendly-content-delivery}

サイトのコンテンツをエージェントが発見しやすく、利用しやすくします。
現在の作業では、コンテンツページの Markdown 出力と `Accept: text/markdown` の HTTP ネゴシエーションを追加しています。

- ステータス: 進行中
- 設計: [Agent support](../design/agent-support/)
- 実装: `netlify/edge-functions/markdown-negotiation.ts` 配下のロジックとテスト用フォルダー。
- リファレンス:
  [opentelemetry.io#9449](https://github.com/open-telemetry/opentelemetry.io/issues/9449),
  [docsy#2596](https://github.com/docsy/docsy/issues/2596)

## ウェブサイトテレメトリー {#website-telemetry}

> [!NOTE]
>
> ウェブサイトテレメトリーは、プロジェクトの新しい Collector が準備できるまで一時停止中です（[#9559][]）。
> このエントリは設計段階の機能について記述しています。

サイトは OpenTelemetry JS SDK を使用してページの読み込み、ユーザーインタラクション、`fetch`/XHR 呼び出しを計装します。
スパンは OTLP 経由で `otelwebtelemetry.com` にエクスポートされ、ブラウザーコンソールに出力されます。

テレメトリー（トレーサー）コードは Netlify ビルドで有効になります。
本番、デプロイプレビュー、ブランチデプロイが対象です。
フォークの Netlify サイトではトレースされません。

- ステータス: アクティブ
- 設計: [Website telemetry](../design/telemetry/)
- 実装: [`assets/js/tracing.js`][]、`layouts/_partials/hooks/body-end.html` から読み込み
- ローカルビルド: [Build and serve](../build/npm-scripts/#build-and-serve)

<!-- prettier-ignore-start -->
[#9559]: https://github.com/open-telemetry/opentelemetry.io/issues/9559
[`assets/js/tracing.js`]: https://github.com/open-telemetry/opentelemetry.io/blob/main/assets/js/tracing.js
<!-- prettier-ignore-end -->
