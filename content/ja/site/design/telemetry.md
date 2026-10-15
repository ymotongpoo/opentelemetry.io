---
title: ウェブサイトテレメトリー
description: >-
  プロジェクトの Netlify サイトのすべてのデプロイでトレーサーが読み込まれ、それ以外では読み込まれない理由、コンテキスト伝搬が Zone フリーを維持する理由、および Collector がエクスポーターに遅れた場合にエクスポート失敗がどのように表面化するか。
weight: 30
default_lang_commit: 0f387a04d1d4029af5139505eb4c1f780ae238cc
---

[ウェブサイトテレメトリー](/site/features/#website-telemetry)機能の設計メモです。

## デプロイメントゲート {#deployment-gate}

[プロジェクトの Netlify サイトのすべてのデプロイ](/site/features/#website-telemetry)でトレーサーを読み込み、それ以外のデプロイでは読み込みません。
body-end パーシャルは、Netlify が設定する `SITE_NAME` ビルド変数によってそのサイトを識別します。

- プレビューはトレースされる必要があります。
  サイトの ID でゲートし、Hugo 環境ではゲートしないことで、本番、デプロイプレビュー、ブランチデプロイを同じ条件に置き、トレースの変更がデプロイプレビューに反映されるようにします。
- フォークはトレースされてはなりません。
  フォークを自前の Netlify サイトにデプロイすると独自の `SITE_NAME` を持つため、ゲートは継承しますがトレースは継承しません。
- これは単純なデプロイメントゲートです。
  Netlify の値は変更不可ですが、自身のビルドを管理する人なら任意の変数を設定できます。
  偽造されたテレメトリーの拒否は Collector の仕事であり、サイトの仕事ではありません。
- 未設定の `SITE_NAME` はシグナルなしでゲートを閉じます。
  Netlify サイトのリネームも、テンプレートが追従するまで同様です。
  ローカルビルドは変数を設定することでオプトインします（[Build and serve](/site/build/npm-scripts/#build-and-serve)）。

## コンテキストマネージャー {#context-manager}

JS SDK の [3.x migration guide][] が `sdk-trace-web` の後継として挙げている `@opentelemetry/sdk-trace` パッケージの `StackContextManager` を、`ZoneContextManager` のかわりに使用します。

- `zone.js` はブラウザーの一般的な非同期 API（`Promise`、タイマー、イベントリスナー）をパッチし、すべてのページに約13KB（gzip 圧縮後）を追加します。
  コールバックを通じたコンテキスト伝搬は実際に機能しますが、ネイティブの `async`/`await` をまたぐことはできず、1ページあたり1つか2つの fetch しかない静的サイトでは不要です。

コンテキストは `await` をまたいだり、バインドされていないコールバック内で自動的に伝搬されません。
そこで開始されたスパンは、コードが親コンテキストを明示的に渡さない限り、独自のルートになります（fetch 計装がプリフライトスパンに対して行うように）。
このサイトのテレメトリーではそれで問題ありません。

## Collector {#collector}

エクスポート先の Collector はサイトとは異なるオーナーとリリーストレインを持つため、その OTLP バージョンはサイトのエクスポーターバージョンに固定されておらず、エクスポーターのアップグレードにより Collector がまだ受け入れないフィールドが送出される可能性があります。
エクスポート失敗はページに影響しません。
SDK はそれを報告しません（診断ロガーは設定されていません）。
そのため、唯一の兆候はブラウザー自体が記録する拒否されたリクエストであり、DevTools のネットワークパネルで確認できます（Chromium コンソールはこれをログ出力しますが、Firefox のコンソールは出力しません）。
コンソールエクスポーターが出力するスパンは、Collector がそれらを受け入れたかどうかについて何も示しません。

<!-- prettier-ignore-start -->
[3.x migration guide]: https://github.com/open-telemetry/opentelemetry-js/blob/main/doc/3.x/migration-guide.md#opentelemetrysdk-trace-web-package-removed
<!-- prettier-ignore-end -->
