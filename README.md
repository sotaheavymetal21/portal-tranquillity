# portal-tranquillity

## 技術スタック
### バックエンド
- Go
- Gin Webフレームワーク
- MongoDB
- Caddy（ウェブサーバー）
- クリーンアーキテクチャ
### フロントエンド
- Next.js
- React
- TypeScript
- Tailwind CSS

## プロジェクト構成
```
portal-tranquillity/
├── backend/             # Goバックエンドアプリケーション
│   ├── cmd/             # エントリーポイント
│   ├── internal/        # 内部パッケージ
│   │   ├── domain/      # ドメインモデル
│   │   ├── repository/  # データアクセス層
│   │   ├── usecase/     # ビジネスロジック
│   │   └── delivery/    # HTTPハンドラー
│   ├── pkg/             # 外部から利用可能なパッケージ
│   ├── Dockerfile       # バックエンド用Dockerfile
│   └── go.mod           # Goの依存関係
│
├── frontend/            # Next.jsフロントエンド
│   ├── components/      # Reactコンポーネント
│   ├── pages/           # ページコンポーネント
│   ├── public/          # 静的アセット
│   ├── styles/          # CSSスタイル
│   ├── Dockerfile       # フロントエンド用Dockerfile
│   └── package.json     # NPM依存関係
│
├── caddy/               # Caddyサーバー設定
│   ├── Caddyfile        # Caddy設定ファイル
│   └── Dockerfile       # Caddy用Dockerfile
│
├── docker-compose.yml           # 開発用Docker Compose設定
├── docker-compose.prod.yml      # 本番用Docker Compose設定
├── Makefile                     # 開発用コマンドショートカット
└── README.md                    # プロジェクト説明（本ファイル）
```

## Docker環境でのセットアップ
### 前提条件
- Docker
- Docker Compose
### 手順
1. リポジトリをクローン
   ```
   git clone https://github.com/yourusername/portal-tranquillity.git
   cd portal-tranquillity
   ```
2. 環境変数ファイルの準備
   ```
   # バックエンド
   cp backend/.env.example backend/.env

   # フロントエンド
   cp frontend/.env.example frontend/.env.local
   ```
3. Dockerコンテナの起動
   ```
   docker-compose up
   ```
4. アプリケーションへのアクセス
   - フロントエンド: http://localhost:3000
   - バックエンドAPI: http://localhost:8080
   - MongoDB管理画面: http://localhost:8081
### 開発時の便利なコマンド
- コンテナをバックグラウンドで起動
  ```
  docker-compose up -d
  ```
- コンテナのログを確認
  ```
  docker-compose logs -f
  ```
- 特定のサービスのログを確認
  ```
  docker-compose logs -f frontend
  docker-compose logs -f backend
  docker-compose logs -f caddy
  ```
- コンテナの停止
  ```
  docker-compose down
  ```
- コンテナとボリュームの削除（データベースも削除）
  ```
  docker-compose down -v
  ```

## 従来の方法でのセットアップ
### 前提条件
- Go 1.21以上
- Node.js 18以上
- MongoDB 6.0以上
- Caddy 2.0以上
### バックエンドのセットアップ
1. バックエンドディレクトリに移動
   ```
   cd portal-tranquillity/backend
   ```
2. 依存関係のインストール
   ```
   go mod download
   ```
3. 環境変数の設定
   ```
   cp .env.example .env
   # .envファイルを編集して必要な設定を行う
   ```
4. サーバーの起動
   ```
   go run cmd/api/main.go
   ```
### フロントエンドのセットアップ
1. フロントエンドディレクトリに移動
   ```
   cd ../frontend
   ```
2. 依存関係のインストール
   ```
   npm install
   ```
3. 環境変数の設定
   ```
   cp .env.example .env.local
   # .env.localファイルを編集して必要な設定を行う
   ```
4. 開発サーバーの起動
   ```
   npm run dev
   ```
### Caddyのセットアップ
1. Caddyのインストール（公式サイトの手順に従ってください）
2. Caddyfileの設定
   ```
   cd ../caddy
   cp Caddyfile.example Caddyfile
   # Caddyfileを編集して必要な設定を行う
   ```
3. Caddyの起動
   ```
   caddy run
   ```

## 本番環境へのデプロイ
### Dockerを使用したデプロイ
1. 本番用の環境変数を設定
   ```
   # バックエンド
   cp backend/.env.example backend/.env.prod
   # 本番用の設定を編集
   
   # フロントエンド
   cp frontend/.env.example frontend/.env.prod
   # 本番用の設定を編集
   
   # Caddy
   cp caddy/Caddyfile.example caddy/Caddyfile.prod
   # 本番用の設定を編集
   ```
2. 本番用のDockerイメージをビルド
   ```
   docker-compose -f docker-compose.prod.yml build
   ```
3. 本番環境でコンテナを起動
   ```
   docker-compose -f docker-compose.prod.yml up -d
   ```
### 従来の方法でのデプロイ
#### バックエンド
1. バイナリのビルド
   ```
   cd backend
   go build -o portal-tranquillity-api cmd/api/main.go
   ```
2. サーバーへの転送とサービス設定
   ```
   # サーバー固有の手順に従ってください
   ```
#### フロントエンド
1. 本番用ビルド
   ```
   cd frontend
   npm run build
   ```
2. 静的ファイルのデプロイ
   ```
   npm run start
   # または、Vercel、Netlifyなどのサービスを利用
   ```
#### Caddy
1. Caddyfileの本番設定を作成
2. 本番サーバーにCaddyをインストール
3. Caddyサービスを設定して起動
   ```
   caddy start
   ```

## Caddyの設定例
以下は基本的なCaddyfileの例です：

```
# Caddyfile
:80 {
  # 本番環境では自動的にHTTPSにリダイレクト
  redir https://{host}{uri} permanent
}

:443 {
  # フロントエンドアプリケーションへのリバースプロキシ
  handle /api/* {
    reverse_proxy backend:8080
  }

  # バックエンドAPIへのリバースプロキシ
  handle /* {
    reverse_proxy frontend:3000
  }

  # 圧縮を有効化
  encode gzip

  # セキュリティヘッダー
  header {
    Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
    X-Content-Type-Options nosniff
    X-Frame-Options DENY
    Referrer-Policy no-referrer-when-downgrade
  }

  # ログ設定
  log {
    output file /var/log/caddy/access.log
    format json
  }
}
```

## ライセンス
MIT

## 貢献
プロジェクトへの貢献は大歓迎です。Issue報告や機能提案、プルリクエストなどお気軽にどうぞ。

## Makefileを使った簡単な起動方法
プロジェクトのルートディレクトリに`Makefile`を用意しています。以下のコマンドで簡単にアプリケーションを起動・管理できます。
### 基本的なコマンド
```bash
# 開発環境のコンテナを起動
make up
# 開発環境のコンテナを停止
make down
# 開発環境のコンテナを再起動
make restart
# コンテナのログを表示
make logs
# 実行中のコンテナを表示
make ps
# コンテナをビルド
make build
```
### 本番環境用コマンド
```bash
# 本番環境のコンテナを起動
make prod-up
# 本番環境のコンテナを停止
make prod-down
# 本番環境のコンテナを再起動
make prod-restart
```
### その他のユーティリティコマンド
```bash
# 未使用のDockerリソースを削除
make clean
# フロントエンドコンテナのシェルに接続
make frontend-shell
# バックエンドコンテナのシェルに接続
make backend-shell
# MongoDBコンテナのシェルに接続
make mongo-shell
# Caddyコンテナのシェルに接続
make caddy-shell
# 使用可能なコマンド一覧を表示
make help
```