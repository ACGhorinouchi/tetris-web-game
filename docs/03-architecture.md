# 03. システムアーキテクチャ

## 3.1 全体構成図

```
┌─────────────────────────────────────────────────────────────────┐
│                         クライアント層                            │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    Web ブラウザ                          │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │   │
│  │  │   React     │  │   Canvas    │  │   State     │     │   │
│  │  │   Components│  │   Renderer  │  │   Management│     │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘     │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       Firebase 層                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │  Firebase   │  │  Cloud      │  │  Firebase   │            │
│  │  Hosting    │  │  Firestore  │  │  Auth       │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        GCP 層（拡張用）                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │  Cloud      │  │  Cloud      │  │  Cloud      │            │
│  │  Functions  │  │  Storage    │  │  Logging    │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
└─────────────────────────────────────────────────────────────────┘
```

## 3.2 コンポーネント構成

### 3.2.1 フロントエンド構成

```
src/
├── components/           # UIコンポーネント
│   ├── Game/
│   │   ├── Board.tsx        # ゲームボード
│   │   ├── Tetromino.tsx    # テトリミノ表示
│   │   ├── NextPiece.tsx    # 次のピース表示
│   │   ├── HoldPiece.tsx    # ホールドピース表示
│   │   ├── ScoreBoard.tsx   # スコア表示
│   │   └── Controls.tsx     # 操作ボタン（モバイル用）
│   ├── UI/
│   │   ├── Header.tsx       # ヘッダー
│   │   ├── Modal.tsx        # モーダル
│   │   └── Button.tsx       # 共通ボタン
│   └── Auth/
│       ├── LoginButton.tsx  # ログインボタン
│       └── UserProfile.tsx  # ユーザープロフィール
├── hooks/                # カスタムフック
│   ├── useGame.ts           # ゲームロジック
│   ├── useAuth.ts           # 認証ロジック
│   ├── useScore.ts          # スコア管理
│   └── useKeyboard.ts       # キーボード入力
├── lib/                  # ユーティリティ
│   ├── firebase.ts          # Firebase初期化
│   ├── tetrominos.ts        # テトリミノ定義
│   └── scoring.ts           # スコア計算
├── pages/                # ページコンポーネント
│   ├── index.tsx            # タイトル画面
│   ├── game.tsx             # ゲーム画面
│   ├── ranking.tsx          # ランキング画面
│   └── settings.tsx         # 設定画面
├── store/                # 状態管理
│   └── gameStore.ts         # グローバルステート
└── styles/               # スタイル
    └── globals.css          # グローバルCSS
```

## 3.3 データフロー

### 3.3.1 ゲームループ

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Input      │───▶│  Game State │───▶│  Renderer   │
│  Handler    │    │  Update     │    │  (Canvas)   │
└─────────────┘    └─────────────┘    └─────────────┘
       ▲                  │
       │                  ▼
       │           ┌─────────────┐
       │           │  Collision  │
       │           │  Detection  │
       │           └─────────────┘
       │                  │
       │                  ▼
       │           ┌─────────────┐
       └───────────│  Game Timer │
                   │  (60fps)    │
                   └─────────────┘
```

### 3.3.2 スコア保存フロー

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Game Over  │───▶│  Calculate  │───▶│  Firestore  │
│  Event      │    │  Final Score│    │  Write      │
└─────────────┘    └─────────────┘    └─────────────┘
                                             │
                                             ▼
                                      ┌─────────────┐
                                      │  Ranking    │
                                      │  Update     │
                                      └─────────────┘
```

## 3.4 ステート管理

### 3.4.1 ゲームステート

```typescript
interface GameState {
  // ボード状態
  board: number[][];           // 10x20 のグリッド

  // 現在のテトリミノ
  currentPiece: {
    type: TetrominoType;
    position: { x: number; y: number };
    rotation: number;
  };

  // ゲーム進行
  nextPiece: TetrominoType;
  holdPiece: TetrominoType | null;
  canHold: boolean;

  // スコア
  score: number;
  level: number;
  lines: number;

  // ゲーム状態
  status: 'idle' | 'playing' | 'paused' | 'gameOver';
}
```

### 3.4.2 認証ステート

```typescript
interface AuthState {
  user: User | null;
  loading: boolean;
  error: Error | null;
}
```

## 3.5 API設計

### 3.5.1 Firestore コレクション

```
firestore/
├── users/                    # ユーザー情報
│   └── {userId}/
│       ├── displayName: string
│       ├── photoURL: string
│       ├── highScore: number
│       └── createdAt: timestamp
│
├── scores/                   # スコア記録
│   └── {scoreId}/
│       ├── userId: string
│       ├── displayName: string
│       ├── score: number
│       ├── level: number
│       ├── lines: number
│       └── createdAt: timestamp
│
└── rankings/                 # キャッシュ用ランキング
    └── global/
        └── topScores: array
```

## 3.6 セキュリティ設計

### 3.6.1 Firestore Security Rules

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // ユーザードキュメント
    match /users/{userId} {
      allow read: if true;
      allow write: if request.auth != null
                   && request.auth.uid == userId;
    }

    // スコアドキュメント
    match /scores/{scoreId} {
      allow read: if true;
      allow create: if request.auth != null
                    && request.resource.data.userId == request.auth.uid;
      allow update, delete: if false;
    }

    // ランキング（読み取りのみ）
    match /rankings/{doc} {
      allow read: if true;
      allow write: if false;
    }
  }
}
```
