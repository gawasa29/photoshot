# アクティブコンテキスト: Photoshot

## 現在のプロジェクト状態

### プロジェクトステータス: **本番稼働中のアプリケーション**

- **ライブアプリケーション**: photoshot.app で利用可能
- **オープンソース**: 活発な開発を行う GitHub リポジトリ
- **本番準備完了**: 完全な機能セットが実装済み
- **ユーザーベース**: AI アバターを生成するアクティブユーザー

### 最近のコンテキスト（メモリーバンク初期化）

- **日付**: 2025 年 6 月 11 日
- **アクション**: 既存プロジェクトのメモリーバンク初期化
- **目的**: 将来の開発のためのドキュメント基盤確立
- **現在の焦点**: 既存システムの理解と文書化

## アクティブな作業領域

### 即座の焦点

1. **メモリーバンクセットアップ**: 包括的なドキュメント構造の作成
2. **システム理解**: 既存コードベースとアーキテクチャのマッピング
3. **コンテキスト確立**: 将来の開発タスクのための基盤構築

### 現在アクティブな主要コンポーネント

- **Next.js 13 App Router**: モダンな React フレームワーク実装
- **Prisma データベース**: 包括的なスキーマを持つ PostgreSQL
- **Replicate 統合**: AI モデル訓練と推論パイプライン
- **Stripe 決済**: クレジットベース収益化システム
- **AWS S3 ストレージ**: 画像アップロードと管理システム

## 現在の技術的決定

### アーキテクチャの選択

- **App Router**: Pages Router より Next.js 13 App Router を使用
- **サーバーコンポーネント**: 適切な場所で React Server Components を活用
- **API Routes**: バックエンドロジック用サーバーレス関数
- **データベース**: 型安全性のため Prisma ORM を使用した PostgreSQL

### UI/UX パターン

- **Chakra UI**: 一貫したデザインシステム実装
- **レスポンシブデザイン**: モバイルファーストアプローチ
- **プログレッシブエンハンスメント**: 低速接続での優雅な劣化
- **ローディング状態**: 非同期操作の包括的フィードバック

### 統合戦略

- **Replicate API**: モデル訓練と推論のプライマリ AI サービス
- **NextAuth**: マジックリンクによるメールベース認証
- **Stripe Checkout**: シンプルさのためのホスト型決済フロー
- **S3 直接アップロード**: 事前署名 URL によるクライアントサイドアップロード

## 現在のユーザーエクスペリエンスフロー

### スタジオ作成プロセス

1. ユーザーがメールマジックリンクで登録/ログイン
2. 名前と設定で新しいスタジオ（プロジェクト）を作成
3. クロッピングインターフェースで 10-20 枚の個人写真をアップロード
4. システムが画像を処理し AI モデル訓練を開始
5. ユーザーが訓練完了を待機（ステータスポーリング）
6. 様々なスタイルプロンプトでアバターショットを生成
7. プレミアム品質のためのオプション HD 強化
8. 生成されたアバターをダウンロードと共有

### クレジットシステム実装

- **デフォルトクレジット**: スタジオあたり 100 ショット、プロンプトウィザード 20 回使用
- **決済フロー**: 追加クレジット用 Stripe チェックアウト
- **使用状況追跡**: 生成毎のリアルタイムクレジット減算
- **透明な価格設定**: ユーザー向けの明確なコスト構造

## アクティブなパターンと設定

### コード構成

- **コンポーネント構造**: 機能とページ別に整理
- **API 設計**: 一貫したエラーハンドリングを持つ RESTful エンドポイント
- **型安全性**: 全体を通じた包括的な TypeScript 使用
- **データベースクエリ**: 適切なリレーションを持つ最適化された Prisma クエリ

### 開発ワークフロー

- **環境管理**: ローカル開発用 Docker Compose
- **データベースマイグレーション**: バージョン管理されたスキーマ変更
- **ビルドプロセス**: Vercel による自動デプロイ
- **エラーハンドリング**: 優雅なエラーバウンダリとユーザーフィードバック

### パフォーマンス最適化

- **画像処理**: サーバーサイド最適化用 Sharp
- **キャッシュ**: Next.js キャッシュメカニズムの戦略的使用
- **データベース**: コネクションプーリングとクエリ最適化
- **CDN**: グローバル画像配信用 S3 CloudFront

## 現在の課題と考慮事項

### 技術的負債領域

- **エラーハンドリング**: より包括的なエラー追跡の恩恵を受ける可能性
- **テスト**: 重要なユーザーフローの限定的なテストカバレッジ
- **パフォーマンス**: 大規模ギャラリーの潜在的最適化機会
- **監視**: 基本的なアナリティクスの強化可能性

### スケーラビリティ考慮事項

- **データベース**: 現在のスキーマは成長をうまく処理
- **ファイルストレージ**: S3 は自動的にスケール
- **AI 処理**: Replicate がコンピュートスケーリングを処理
- **API 制限**: レート制限の実装が必要な可能性

### ユーザーエクスペリエンス改善

- **オンボーディング**: 初回ユーザー体験の合理化可能
- **進捗フィードバック**: 訓練中の強化されたステータス更新
- **バッチ操作**: 一括ダウンロードと生成機能
- **モバイル体験**: ネイティブアプリの検討可能性

## 次のステップと優先順位

### 即座のアクション

1. **メモリーバンク完成**: ドキュメント構造の完了
2. **システム監査**: 改善のための現在実装のレビュー
3. **パフォーマンス分析**: 最適化機会の特定
4. **ユーザーフィードバック**: 強化優先順位のための洞察収集

### 中期目標

- **機能強化**: ユーザーフィードバックと使用パターンに基づく
- **パフォーマンス最適化**: 特定されたボトルネックへの対処
- **テストカバレッジ**: 包括的なテストスイートの実装
- **監視**: 強化されたアナリティクスとエラー追跡

### 長期ビジョン

- **API アクセス**: サードパーティ統合用開発者 API
- **高度な機能**: カスタムプロンプト作成、チームアカウント
- **モバイルアプリ**: ネイティブモバイルアプリケーション開発
- **コミュニティ機能**: ユーザーギャラリー、共有、コラボレーション

## 主要な学習と洞察

### 成功パターン

- **シンプルさ**: クリーンで直感的なユーザーインターフェースが採用を促進
- **信頼性**: 堅牢なエラーハンドリングがユーザーの信頼を維持
- **パフォーマンス**: 高速な画像処理がユーザーエンゲージメントを維持
- **透明性**: 明確な価格設定とプロセスが信頼を構築

### 成長領域

- **ユーザーオンボーディング**: 初回体験のより多くのガイダンス可能
- **機能発見**: ユーザーがすべての機能を発見しない可能性
- **コミュニティ構築**: ユーザー生成コンテンツ共有の可能性
- **収益化**: クレジット以外の追加収益源

### 技術的優秀性

- **型安全性**: TypeScript が多くのランタイムエラーを防止
- **モダンスタック**: Next.js 13 が優秀な開発者体験を提供
- **外部サービス**: Replicate と Stripe が複雑な操作をうまく処理
- **データベース設計**: Prisma スキーマが現在と将来のニーズをサポート

## 現在の環境ステータス

- **開発**: 完全に機能するローカル開発セットアップ
- **本番**: ユーザーにサービスを提供するライブアプリケーション
- **ドキュメント**: 進行中のメモリーバンク初期化
- **コミュニティ**: 貢献の可能性を持つオープンソースプロジェクト
