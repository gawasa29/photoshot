# 技術コンテキスト: Photoshot

## 技術スタック

### フロントエンドフレームワーク

- **Next.js 13.5.6**: App Router を使用した React フレームワーク
- **React 18.2.0**: フックとコンテキストを使用した UI ライブラリ
- **TypeScript 4.9.3**: 型安全性と開発者体験
- **Chakra UI 2.4.2**: コンポーネントライブラリとデザインシステム
- **Framer Motion 10.16.4**: アニメーションとトランジション

### バックエンドと API

- **Next.js API Routes**: サーバーレス API エンドポイント
- **NextAuth.js 4.24.3**: 認証とセッション管理
- **Prisma 4.7.1**: データベース ORM とマイグレーション
- **PostgreSQL**: プライマリデータベース（DATABASE_URL 経由）

### 外部サービス

- **Replicate API**: AI モデル訓練と推論
- **AWS S3**: 画像ストレージと CDN
- **Stripe**: 決済処理
- **OpenAI API**: プロンプトウィザード機能
- **SMTP/Nodemailer**: メール配信

### 開発ツール

- **ESLint**: コードリンティングとフォーマット
- **Prisma CLI**: データベース管理
- **Docker Compose**: ローカル開発環境
- **Vercel**: デプロイプラットフォーム（スクリプトから推測）

## 環境設定

### 必要な環境変数

```bash
# データベース
DATABASE_URL=postgresql://user:pass@host:port/db

# 認証
NEXTAUTH_URL=https://your-domain.com
SECRET=random-secret-string

# AWS S3 ストレージ
S3_UPLOAD_KEY=aws-access-key
S3_UPLOAD_SECRET=aws-secret-key
S3_UPLOAD_BUCKET=bucket-name
S3_UPLOAD_REGION=us-east-1

# Replicate AI
REPLICATE_API_TOKEN=r_token
REPLICATE_USERNAME=username
REPLICATE_MAX_TRAIN_STEPS=3000
REPLICATE_NEGATIVE_PROMPT="cropped face, cover face, cover visage, mutated hands"
REPLICATE_HD_VERSION_MODEL_ID=model-id
NEXT_PUBLIC_REPLICATE_INSTANCE_TOKEN=abc

# メール
EMAIL_FROM=noreply@domain.com
EMAIL_SERVER=smtp://localhost:25

# Stripe 決済
STRIPE_SECRET_KEY=sk_test_or_live
NEXT_PUBLIC_STRIPE_STUDIO_PRICE=1000
NEXT_PUBLIC_STUDIO_SHOT_AMOUNT=100

# OpenAI（オプション）
OPENAI_API_KEY=sk-openai-key
OPENAI_API_SEED_PROMPT=seed-prompt
```

### 開発セットアップ

```bash
# 依存関係のインストール
yarn install

# ローカルサービス（PostgreSQL + MailDev）のセットアップ
docker-compose up -d

# 環境設定
cp .env.example .env.local
# .env.local を実際の値で編集

# データベースセットアップ
yarn prisma:migrate:dev

# 開発サーバー起動
yarn dev
```

## データベーススキーマ

### コアテーブル

- **users**: ユーザーアカウントとプロフィール
- **accounts/sessions**: NextAuth 認証
- **projects**: ユーザースタジオ/プロジェクト
- **shots**: 生成されたアバター画像
- **payments**: 決済追跡

### 主要な関係

```sql
users (1:N) projects (1:N) shots
projects (1:N) payments
users (1:N) accounts/sessions
```

### マイグレーション戦略

- `/prisma/migrations/` 内の Prisma マイグレーション
- バージョン管理されたスキーマ変更
- `prisma migrate deploy` による本番マイグレーション

## API アーキテクチャ

### ルート構造

```
/api/
├── auth/[...nextauth].ts    # NextAuth エンドポイント
├── projects/
│   ├── index.ts             # プロジェクトの一覧/作成
│   ├── [id]/
│   │   ├── index.ts         # プロジェクトの取得/更新/削除
│   │   └── predictions.ts   # ショット生成
└── checkout/
    └── index.ts             # Stripe 決済処理
```

### 認証フロー

1. NextAuth によるメールマジックリンク
2. データベースに保存されるセッション
3. `getServerSession` で保護される API ルート
4. `useSession` によるクライアントサイドセッション

## ファイルアップロード戦略

### 画像処理パイプライン

1. **クライアントアップロード**: 事前署名 URL 経由で S3 に直接
2. **画像処理**: リサイズ/最適化用 Sharp
3. **クロッピング**: React Advanced Cropper コンポーネント
4. **圧縮**: サイズ最適化用 Image Blob Reduce
5. **ストレージ**: 整理されたフォルダ構造での S3

### ファイル構成

```
s3://bucket/
├── users/
│   └── {userId}/
│       ├── uploads/         # オリジナル写真
│       ├── processed/       # クロップ/リサイズ済み
│       └── outputs/         # 生成されたアバター
└── temp/                    # 一時ファイル
```

## AI 統合

### Replicate ワークフロー

1. **訓練データ準備**: アップロード画像の ZIP 化
2. **モデル訓練**: パラメータ付きで Replicate に送信
3. **ステータスポーリング**: 訓練進捗の確認
4. **モデル準備完了**: モデルバージョン ID の保存
5. **予測**: 訓練済みモデルを使用した画像生成
6. **HD 処理**: オプションのアップスケーリングパイプライン

### モデルパラメータ

- **インスタンスプロンプト**: `"a photo of {instanceName} {instanceClass}"`
- **クラスプロンプト**: `"a photo of a {instanceClass}"`
- **訓練ステップ**: 設定可能（デフォルト 3000）
- **ネガティブプロンプト**: 品質制御プロンプト

## 決済統合

### Stripe 設定

- **チェックアウトセッション**: 一回払い
- **Webhook**: 決済確認
- **商品**: スタジオ作成、クレジットパッケージ
- **通貨**: USD（設定可能）

### クレジットシステム

- **スタジオクレジット**: プロジェクトあたりデフォルト 100
- **プロンプトウィザードクレジット**: プロジェクトあたりデフォルト 20
- **減算**: ショット生成毎
- **チャージ**: Stripe チェックアウト経由

## パフォーマンス考慮事項

### 画像最適化

- **Next.js Image**: 自動最適化
- **Blurhash**: プレースホルダー生成
- **段階的読み込み**: ギャラリーの遅延読み込み
- **CDN**: S3 CloudFront 配信

### データベースパフォーマンス

- **コネクションプーリング**: Prisma コネクション管理
- **インデックス**: 外部キーとクエリ最適化
- **ページネーション**: 大規模データセット処理

### キャッシュ戦略

- **静的ページ**: Next.js 静的生成
- **API レスポンス**: 条件付きキャッシュ
- **画像**: ブラウザと CDN キャッシュ
- **データベース**: 適切な場所でのクエリ結果キャッシュ

## セキュリティ実装

### データ保護

- **入力バリデーション**: API 入力用 Zod スキーマ
- **SQL インジェクション**: Prisma ORM 保護
- **XSS 防止**: React 組み込み保護
- **CSRF**: NextAuth CSRF トークン

### ファイルセキュリティ

- **アップロードバリデーション**: ファイルタイプとサイズ制限
- **S3 ポリシー**: 制限されたバケットアクセス
- **事前署名 URL**: 時間制限付きアップロードアクセス
- **コンテンツスキャン**: 基本的なファイルバリデーション

## 監視とログ

### エラー追跡

- **API エラー**: 構造化されたエラーレスポンス
- **クライアントエラー**: React エラーバウンダリ
- **外部サービスエラー**: Replicate/Stripe エラーハンドリング

### アナリティクス

- **Vercel Analytics**: 基本的な使用状況追跡
- **カスタムイベント**: ユーザーアクション追跡
- **パフォーマンス**: Core Web Vitals 監視

## デプロイ設定

### ビルドプロセス

```bash
# 本番ビルド
yarn vercel-build
# 実行内容: prisma generate && prisma migrate deploy && next build
```

### 環境要件

- **Node.js**: Next.js 13 と互換性
- **PostgreSQL**: 本番データベース
- **Redis**: セッションストレージ用（オプション）
- **CDN**: 静的アセット配信用

### スケーリング考慮事項

- **サーバーレス**: Next.js API ルートの自動スケール
- **データベース**: 同時ユーザー用コネクションプーリング
- **ファイルストレージ**: S3 は大容量ファイルを処理
- **AI 処理**: Replicate がコンピュートスケーリングを管理
