# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

テトリス Web ゲーム - Firebase/GCP を活用したブラウザベースのテトリスゲーム。

## Build & Development Commands

```bash
# 開発サーバー起動
npm run dev

# ビルド（静的ファイル生成）
npm run build

# Lint
npm run lint

# フォーマット
npm run format

# テスト実行
npm run test

# 単一テストファイル実行
npx vitest run src/hooks/useGame.test.ts

# E2Eテスト
npx playwright test

# Firebase デプロイ
firebase deploy --only hosting

# Firebase プレビュー環境
firebase hosting:channel:deploy preview
```

## Architecture

### 技術スタック
- **フロントエンド**: Next.js 14 + React 18 + TypeScript 5
- **状態管理**: Zustand（ゲーム状態）、React Query（サーバー状態）
- **スタイリング**: Tailwind CSS
- **ゲーム描画**: HTML5 Canvas + requestAnimationFrame
- **バックエンド**: Firebase（Authentication, Firestore, Hosting）

### ディレクトリ構成
```
src/
├── components/Game/     # ゲームUI（Board, Tetromino, ScoreBoard等）
├── components/Auth/     # 認証UI
├── hooks/               # useGame, useAuth, useScore, useKeyboard
├── lib/                 # firebase.ts, tetrominos.ts, scoring.ts
├── pages/               # Next.js ページ
└── store/               # Zustand ストア
```

### ゲームループ
Input → GameState Update → Collision Detection → Canvas Render (60fps)

### Firestore コレクション
- `users/{userId}` - ユーザー情報・ハイスコア
- `scores/{scoreId}` - スコア記録（作成のみ、更新不可）
- `rankings/global` - ランキングキャッシュ

## Firebase 環境変数

`.env.local` に以下を設定:
```
NEXT_PUBLIC_FIREBASE_API_KEY=
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=
NEXT_PUBLIC_FIREBASE_PROJECT_ID=
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=
NEXT_PUBLIC_FIREBASE_APP_ID=
```
