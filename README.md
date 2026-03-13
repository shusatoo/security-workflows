# security-workflows

CodeQL などのセキュリティ関連ワークフローをまとめる共有リポジトリ

## 設計方針

- アプリケーション固有で変わる設定だけを呼び出し元リポジトリに残す
- action のバージョン、setup 手順、CodeQL 実行手順のように
  組織共通で保守したい設定は reusable workflow 側に寄せる
- この方針により、共通タグの更新や action version の更新を
  できるだけ共通側だけで吸収し、呼び出し元の修正範囲を小さく保つ
- 一方で、トリガー条件、ビルドコマンド、必要に応じた解析対象言語の上書きなど、
  リポジトリ固有で変わりやすい設定は呼び出し元で管理する

## CodeQL の使い方

各リポジトリで CodeQL を使うときは、このリポジトリにあるサンプルを
コピーして、各リポジトリの `.github/workflows/codeql.yml`
として配置してください。

`codeql.yml` は、各リポジトリで CodeQL を実行するための
GitHub Actions ワークフロー定義です。
このファイルから、このリポジトリの reusable workflow を参照して使います。

### Kotlin + Gradle 用サンプル

Kotlin と Gradle を使うリポジトリ向けのサンプルは
[`codeql-kotlin-gradle.yml`](./.github/workflow-templates/codeql-kotlin-gradle.yml)
です。

このサンプルをベースに、各リポジトリで `build-command` などを必要に応じて調整して使ってください。
