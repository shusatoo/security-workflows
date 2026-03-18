# security-workflows

CodeQL などのセキュリティ関連ワークフローをまとめる共有リポジトリ

## 概要

- アプリケーション共通で保守できるワークフロー設定を、本リポジトリに定義する
    - actionのバージョン、setup手順、CodeQL実行手順、等
- トリガー条件、ビルドコマンドなどは、アプリケーション側で管理する
    - アプリケーション固有で変わりやすい設定は、アプリケーションのリポジトリで定義する

## バージョンタグ運用

- 本リポジトリのワークフローを外部リポジトリから参照するために、ワークフロー変更時のコミットにはバージョンタグをつける
- バージョンは、固定タグと可動タグの2種類で管理する
    - 固定タグは `v1.0`, `v1.1` のような個別リリースを表し、一度付けたら動かさない
    - 可動タグは `v1` のようなメジャー系列の最新安定版を表し、新しい安定版を出したときに更新する
    - 例: `v1.1` を新しいコミットに付けたら、必要に応じて `v1` もそのコミットへ移動する
- 呼び出す側のリポジトリ（アプリケーション側）は、変更を明示的に取り込みたい場合は固定タグを使い、
  同一メジャー内の改善を追従したい場合は可動タグを使う

### メジャーバージョンを上げる基準

- `v1.x` のまま進める場合
    - 既存の呼び出し側ワークフローの修正が不要（互換変更）
- `v2` を作る場合
    - 既存の呼び出し側ワークフローの修正が必要（非互換変更）
- 例:
  - 必須 input の追加
  - 既存 input 名の変更や削除
  - 既定値を互換性のない方向へ変更すること
  - 呼び出し元の `with:` 設定を変えないと動かなくなる実行手順変更

## CodeQL の使い方

以下のサンプルをコピーして、各リポジトリの `.github/workflows/codeql.yml` として配置します。

### Kotlin + Gradle 用サンプル

Kotlin と Gradle を使うリポジトリ向けのサンプル:

```yaml
name: CodeQL

on:
  push:
    # mainへのpushで実行する。
    branches: ["main"]
  pull_request:
    # main向けのPull Requestで実行する。
    branches: ["main"]
  schedule:
    # 毎週UTC月曜20時（JSTの火曜5時）に実行する。
    # デフォルトブランチ（main）の最新コミットが対象。
    - cron: '0 20 * * 1'

jobs:
  analyze:
    permissions:
      contents: read
      security-events: write
    uses: [organization]/security-workflows/.github/workflows/codeql-java-gradle.yml@v1
    with:
      java-version: '17'
      codeql-language: java-kotlin
      build-command: ./gradlew clean build --no-daemon
```

このサンプルをベースに、各リポジトリで `build-command` などを必要に応じて調整して使います。

## 脆弱性アラート通知の設定

CodeQLで検出された脆弱性アラートをSlack通知する場合は、以下のサンプルをコピーして、各リポジトリの `.github/workflows/notify-code-scanning-alerts.yml` として配置します。

サンプル:

```yaml
name: Notify Code Scanning Alerts

on:
  schedule:
    - cron: '0 21 * * 1'

jobs:
  notify:
    uses: [organization]/security-workflows/.github/workflows/notify-code-scanning-alerts.yml@v1
    secrets:
      slack_webhook_url: ${{ secrets.SLACK_WEBHOOK_URL }}
```

必要な secrets:

- `SLACK_WEBHOOK_URL`

補足:

- 上記サンプルをそのまま使う場合、Slack Incoming Webhook URL は呼び出し元リポジトリの GitHub Actions Secrets に保存します
- 各リポジトリへ個別に保存したくない場合は、`Organization secrets` や `Environment secrets` に集約する運用もできます
- GitHub に Slack Webhook URL を保存したくない場合は、外部の parameter / secret 管理サービス（Google Cloud の Secret Manager等）から実行時に取得する構成にします。その場合、このサンプルではそのまま使えず、別途実装が必要です。
