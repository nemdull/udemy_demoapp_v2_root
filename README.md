# Udemy Demo App v2 Root

このリポジトリは、Rails API バックエンド（`api`）と Nuxt.js フロントエンド（`front`）をサブモジュールとして管理し、Docker でローカル開発環境を起動するためのルートプロジェクトです。

## ディレクトリ/ファイル構成

- `api/`: Rails API アプリ（Gitサブモジュール）
  - `Dockerfile`: Alpine + Ruby 2.7 のビルド定義。Nokogiri をネイティブビルドできるよう `libxml2-dev`/`libxslt-dev` 等を追加済み。
  - `Gemfile`, `Gemfile.lock`: API の gem 依存関係。Nokogiri 1.15系に更新済み。
  - `app/`, `config/`, `db/` など: 一般的な Rails 構成。

- `front/`: Nuxt.js フロントエンド（Gitサブモジュール）
  - `Dockerfile`: Node 16 (alpine) ベースのビルド定義。
  - `nuxt.config.js`, `pages/`, `components/` など: 一般的な Nuxt 2 構成。

- `docker-compose.yml`: DB/API/Front の3サービスを起動する Compose 定義。
  - `db`: `postgres:13.1-alpine`
  - `api`: Rails サーバ（`bundle exec rails s -p 3000 -b 0.0.0.0`）
  - `front`: Nuxt 開発サーバ（`yarn dev`）

- `.env`: Compose で参照する環境変数ファイル（ルート直下）。例:
  - `WORKDIR=app`
  - `API_PORT=3001`（ホスト側で API を公開するポート）
  - `FRONT_PORT=3100`（ホスト側で Front を公開するポート。ローカルの 3000 競合回避のため 3100 に設定済み）
  - `POSTGRES_PASSWORD=postgres`

- `.gitmodules`: `api` と `front` のサブモジュール定義。
- `.gitignore`: ルートの無視設定（`.env` など）。

## 前提

- Docker Desktop が動作していること
- Git サブモジュールを利用できること（本リポジトリで初回は `git submodule update --init --recursive` が必要）

## セットアップ（初回）

1) ルート直下に `.env` を作成（値例は上記参照）。

2) ビルドと起動

```
docker compose build
docker compose up -d
```

3) 依存関係の導入（フロントのみ初回必要な場合あり）

```
docker compose run --rm front yarn install
docker compose up -d front
```

4) データベース初期化（API）

```
docker compose exec api rails db:migrate
# （必要なら）
docker compose exec api rails db:seed
```

## アクセス

- Front: `http://localhost:${FRONT_PORT}`（デフォルト: `http://localhost:3100`）
- API:   `http://localhost:${API_PORT}`（デフォルト: `http://localhost:3001`）

## よくあるトラブル/メモ

- ポート競合で Front が起動しない場合: `.env` の `FRONT_PORT` を変更し、`docker compose up -d front` で再作成。
- Compose の `version` キーは非推奨のため警告が出ますが、動作には影響ありません。
- Apple Silicon/Alpine 上での Nokogiri 関連エラーに対応するため、`api/Dockerfile` にビルド依存（`libxml2-dev`, `libxslt-dev`）追加と `BUNDLE_FORCE_RUBY_PLATFORM=1` を設定済みです。

