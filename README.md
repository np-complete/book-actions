# book-actions

書籍（Markdown / Re:VIEW）をビルド・公開するための Composite GitHub Actions 集です。

## 構成

- `build/` - 書籍 PDF をビルドする Action
- `publish/` - ビルド済み PDF を S3 に公開する Action

## `build` Action

`np-complete/book-actions/build` は、作業ディレクトリに共通ファイルを展開したうえで Ruby 環境を準備し、Markdown 変換と Re:VIEW による PDF 生成を行います。

### Inputs

| 名前 | 必須 | デフォルト | 説明 |
| --- | --- | --- | --- |
| `config` | Yes | `config.yml` | 使用する Re:VIEW 設定ファイル |
| `working-directory` | No | `.` | ビルド対象の作業ディレクトリ |

### 処理内容

1. `build/shares/` の内容を作業ディレクトリへコピー
2. `ruby/setup-ruby` で Ruby 環境をセットアップ（`bundler-cache: true`）
3. `bundle exec rake convert_md` を実行
4. `config.yml` の先頭に `inherit: ["config.common.yml"]` を追加
5. `vvakame/review:5.9` コンテナで `review-pdfmaker config.yml` を実行

### 使用例

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: np-complete/book-actions/build@v1
        with:
          config: config.yml
          working-directory: .
```

## `publish` Action

`np-complete/book-actions/publish` は、AWS 認証を行ったあと `book.pdf` を S3 バケットへアップロードします。

### 前提

- ワークフロー実行環境で OIDC 等により AWS ロールを引き受けられること
- カレントディレクトリにアップロード対象の `book.pdf` が存在すること

### 処理内容

1. `arn:aws:iam::009775665146:role/publish-np-complete-book-role`（`us-west-2`）を Assume Role
2. `github.repository` からリポジトリ名を取得
3. `book.pdf` を `s3://np-complete-books/pdf/<repository-name>.pdf` に公開アップロード  
   （`--acl public-read --cache-control "max-age=900"`）

### 使用例

```yaml
jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - uses: actions/checkout@v4
      - uses: np-complete/book-actions/build@v1
      - uses: np-complete/book-actions/publish@v1
```

## ライセンス

[MIT](./LICENSE)
