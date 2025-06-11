# システムパターン: Photoshot

## アーキテクチャ概要

### 高レベルアーキテクチャ

```
フロントエンド (Next.js) → API ルート → データベース (PostgreSQL)
                      ↓
外部サービス: Replicate, AWS S3, Stripe, OpenAI
```

### コアコンポーネント

1. **Web アプリケーション**: App Router を使用した Next.js 13
2. **データベース層**: PostgreSQL を使用した Prisma ORM
3. **認証**: メール/OAuth を使用した NextAuth.js
4. **ファイルストレージ**: 画像用 AWS S3
5. **AI 処理**: モデル訓練と推論用 Replicate API
6. **決済**: Stripe 統合
7. **メール**: Nodemailer を使用した SMTP

## 主要な設計パターン

### データフローパターン

#### プロジェクト作成フロー

1. ユーザーがプロジェクト作成 → データベースレコード作成
2. 写真アップロード → S3 ストレージ → URL をデータベースに保存
3. Replicate 訓練開始 → モデル ID を追跡
4. 訓練ステータスポーリング → データベースでステータス更新
5. モデル準備完了 → ユーザーがショット生成可能

#### ショット生成フロー

1. ユーザーがプロンプト送信 → API バリデーション
2. クレジット確認 → 利用可能な場合はクレジット減算
3. Replicate 予測 → 予測 ID を追跡
4. 結果ポーリング → 出力 URL を保存
5. オプション HD 処理 → 追加予測

### データベースパターン

#### エンティティ関係

- **User** (1:N) **Project** (1:N) **Shot**
- **Project** (1:N) **Payment**
- **User** (1:N) **Account/Session** (NextAuth)

#### 主要エンティティ

```typescript
User {
  id, email, name, projects[]
}

Project {
  id, name, userId, instanceName, instanceClass
  replicateModelId, modelStatus, imageUrls[]
  credits, promptWizardCredits
  shots[], payments[]
}

Shot {
  id, projectId, prompt, replicateId
  status, outputUrl, hdOutputUrl
  bookmarked, blurhash, seed
}
```

### API パターン

#### RESTful エンドポイント

- `/api/projects` - プロジェクトの CRUD 操作
- `/api/projects/[id]/predictions` - ショット生成
- `/api/checkout` - 決済処理
- `/api/auth/*` - NextAuth エンドポイント

#### エラーハンドリングパターン

```typescript
try {
  // API 操作
  return { success: true, data }
} catch (error) {
  return { success: false, error: error.message }
}
```

## コンポーネントアーキテクチャ

### ページ構造

```
app/
├── (auth)/          # 保護されたルート
│   ├── dashboard/   # ユーザーダッシュボード
│   └── studio/      # プロジェクト管理
├── (public)/        # パブリックページ
│   ├── login/
│   ├── gallery/
│   └── faq/
└── api/            # API ルート
```

### コンポーネント階層

```
Layout
├── Header (ナビゲーション、認証状態)
├── PageContainer
│   ├── HomePage (ヒーロー、機能、価格)
│   ├── DashboardPage (プロジェクト一覧)
│   └── StudioPage (プロジェクト詳細、ショット)
└── Footer
```

### 状態管理パターン

#### プロジェクトコンテキスト

- React Context を使用したグローバルプロジェクト状態
- 現在のプロジェクト、ショット、ローディング状態を管理
- ショットの CRUD 操作を提供

#### フォームパターン

- バリデーション用 React Hook Form
- Chakra UI フォームコンポーネント
- ドラッグ&ドロップ付きファイルアップロード

## 統合パターン

### Replicate 統合

```typescript
// 訓練パターン
const training = await replicate.trainings.create({
  version: "model-version-id",
  input: {
    instance_prompt: `a photo of ${instanceName} ${instanceClass}`,
    class_prompt: `a photo of a ${instanceClass}`,
    instance_data: zipUrl,
    max_train_steps: maxTrainSteps,
  },
})

// 予測パターン
const prediction = await replicate.predictions.create({
  version: modelVersionId,
  input: {
    prompt: `${prompt}, ${instanceName} ${instanceClass}`,
    negative_prompt: negativePrompt,
    num_inference_steps: 50,
    guidance_scale: 7.5,
  },
})
```

### S3 アップロードパターン

```typescript
// 直接アップロード用の事前署名 URL
const { url, fields } = await s3.createPresignedPost({
  Bucket: bucket,
  Fields: { key: `${userId}/${filename}` },
  Conditions: [
    ["content-length-range", 0, maxSize],
    ["starts-with", "$Content-Type", "image/"],
  ],
})
```

### 決済フローパターン

```typescript
// Stripe チェックアウトセッション
const session = await stripe.checkout.sessions.create({
  mode: "payment",
  payment_method_types: ["card"],
  line_items: [
    {
      price_data: {
        currency: "usd",
        product_data: { name: "Studio Credits" },
        unit_amount: price,
      },
      quantity: 1,
    },
  ],
  success_url: `${baseUrl}/dashboard?success=true`,
  cancel_url: `${baseUrl}/dashboard?canceled=true`,
})
```

## セキュリティパターン

### 認証フロー

1. NextAuth によるメールベースマジックリンク認証
2. セッション管理
3. ミドルウェアによるルート保護
4. API ルート認証チェック

### データ保護

- userId によるユーザーデータ分離
- 画像アクセス用 S3 バケットポリシー
- 環境変数管理
- 入力バリデーションとサニタイゼーション

## パフォーマンスパターン

### 画像最適化

- 最適化用 Next.js Image コンポーネント
- プレースホルダー画像用 Blurhash
- ギャラリーの段階的読み込み
- 高速配信用 S3 CDN

### データベース最適化

- Prisma クエリ最適化
- 外部キーの適切なインデックス
- コネクションプーリング
- 大規模データセットのページネーション

### キャッシュ戦略

- パブリックページの静的生成
- 適切な場所での API レスポンスキャッシュ
- CDN による画像キャッシュ
- クライアントサイド状態キャッシュ

## エラーハンドリングと監視

### エラーバウンダリ

- コンポーネント障害用 React エラーバウンダリ
- API 障害の優雅な劣化
- ユーザーフレンドリーなエラーメッセージ

### ステータス追跡

- モデル訓練ステータスポーリング
- 予測ステータス監視
- 決済ステータス検証
- 長時間操作のユーザーフィードバック
