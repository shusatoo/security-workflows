# security-workflows

このディレクトリは、CodeQL などのセキュリティ関連ワークフローをまとめる
共有リポジトリ `security-workflows` に配置する想定のファイル群です。

想定している移行先の構成:

- `.github/workflows/codeql-java-kotlin.yml`

移行手順の概要:

1. このディレクトリ配下の `.github/workflows/` を、新しい
   `security-workflows` リポジトリへそのままコピーする
2. 呼び出し元リポジトリの `uses:` を
   `<owner>/security-workflows/.github/workflows/codeql-java-kotlin.yml@<ref>`
   に置き換える
3. 各リポジトリは、トリガー条件とリポジトリ固有の `with:` 入力だけを持つ

補足:

- このサンプルリポジトリでは、動作確認しやすいように同じ reusable workflow を
  `.github/workflows/reusable-codeql-java-kotlin.yml` にも置いている
- 将来共通リポジトリへ移したら、呼び出し元はそちらを参照するだけでよい
- 将来的に Secret Scanning や Dependabot 向けのテンプレートやドキュメントを
  追加する場合も、このディレクトリを起点に整理できる

このサンプルでは、呼び出し元から次の項目を上書きできるようにしています。

- `runs-on`
- `java-version`
- `java-distribution`
- `codeql-language`
- `build-command`

呼び出し例:

```yaml
jobs:
  analyze:
    permissions:
      contents: read
      security-events: write
    uses: your-org/security-workflows/.github/workflows/codeql-java-kotlin.yml@main
    with:
      java-version: '17'
      build-command: ./scripts/build-app.sh
```
