# 04. 技術スタック

## 4.1 フロントエンド

### 4.1.1 コアフレームワーク

| 技術 | バージョン | 用途 |
|-----|-----------|------|
| React | 18.x | UIフレームワーク |
| TypeScript | 5.x | 型安全な開発 |
| Next.js | 14.x | SSG/SSR フレームワーク |

### 4.1.2 状態管理

| 技術 | バージョン | 用途 |
|-----|-----------|------|
| Zustand | 4.x | グローバル状態管理 |
| React Query | 5.x | サーバー状態管理 |

### 4.1.3 スタイリング

| 技術 | バージョン | 用途 |
|-----|-----------|------|
| Tailwind CSS | 3.x | ユーティリティファーストCSS |
| CSS Modules | - | コンポーネントスコープCSS |

### 4.1.4 ゲーム描画

| 技術 | 用途 |
|-----|------|
| HTML5 Canvas | ゲームボード描画 |
| requestAnimationFrame | ゲームループ |

## 4.2 バックエンド（Firebase）

### 4.2.1 Firebase サービス

| サービス | 用途 |
|---------|------|
| Firebase Authentication | ユーザー認証（匿名/Google） |
| Cloud Firestore | スコア・ランキングデータ保存 |
| Firebase Hosting | 静的ファイルホスティング |
| Firebase Analytics | ユーザー行動分析 |

### 4.2.2 Firebase SDK

| パッケージ | バージョン | 用途 |
|-----------|-----------|------|
| firebase | 10.x | Firebase JS SDK |
| firebase-admin | 12.x | 管理用SDK（必要時） |

## 4.3 GCP サービス（拡張用）

| サービス | 用途 | 導入時期 |
|---------|------|---------|
| Cloud Functions | サーバーレス関数 | フェーズ2 |
| Cloud Storage | アセット保存 | 必要時 |
| Cloud Logging | ログ管理 | フェーズ2 |
| Cloud Monitoring | パフォーマンス監視 | フェーズ2 |

## 4.4 開発ツール

### 4.4.1 ビルドツール

| ツール | 用途 |
|-------|------|
| npm/yarn/pnpm | パッケージ管理 |
| Vite / Next.js | ビルド・バンドル |
| ESLint | コード品質チェック |
| Prettier | コードフォーマット |

### 4.4.2 テストツール

| ツール | 用途 |
|-------|------|
| Vitest | ユニットテスト |
| Playwright | E2Eテスト |
| Testing Library | コンポーネントテスト |

### 4.4.3 CI/CD

| ツール | 用途 |
|-------|------|
| GitHub Actions | CI/CD パイプライン |
| Firebase CLI | デプロイ |

## 4.5 選定理由

### 4.5.1 React + TypeScript

- **型安全性**: ゲームロジックの複雑な状態管理に有効
- **エコシステム**: 豊富なライブラリとコミュニティサポート
- **パフォーマンス**: 仮想DOMによる効率的な再描画

### 4.5.2 Next.js

- **SSG対応**: 静的ファイル生成でFirebase Hosting最適化
- **画像最適化**: 自動的な画像最適化
- **開発体験**: Fast Refresh による高速開発

### 4.5.3 Zustand

- **軽量**: バンドルサイズが小さい
- **シンプル**: ボイラープレートが少ない
- **柔軟**: ゲーム状態管理に適した設計

### 4.5.4 Firebase

- **統合環境**: 認証・DB・ホスティングが一体化
- **リアルタイム**: Firestoreのリアルタイム同期
- **スケーラビリティ**: 自動スケーリング
- **無料枠**: 個人開発に十分な無料枠

### 4.5.5 Canvas API

- **パフォーマンス**: DOM操作より高速な描画
- **柔軟性**: ピクセルレベルの制御
- **標準技術**: 追加ライブラリ不要

## 4.6 パッケージ一覧

```json
{
  "dependencies": {
    "next": "^14.0.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "firebase": "^10.7.0",
    "zustand": "^4.4.0",
    "@tanstack/react-query": "^5.0.0"
  },
  "devDependencies": {
    "typescript": "^5.3.0",
    "tailwindcss": "^3.4.0",
    "eslint": "^8.56.0",
    "prettier": "^3.1.0",
    "vitest": "^1.0.0",
    "@types/react": "^18.2.0",
    "@types/node": "^20.0.0"
  }
}
```

## 4.7 ブラウザ互換性

| 機能 | Chrome | Firefox | Safari | Edge |
|------|--------|---------|--------|------|
| Canvas 2D | ✓ | ✓ | ✓ | ✓ |
| ES2020+ | ✓ | ✓ | ✓ | ✓ |
| CSS Grid | ✓ | ✓ | ✓ | ✓ |
| Touch Events | ✓ | ✓ | ✓ | ✓ |
| Web Audio | ✓ | ✓ | ✓ | ✓ |
