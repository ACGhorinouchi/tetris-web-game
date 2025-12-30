# 06. Firebase / GCP 設計

## 6.1 Firebase プロジェクト構成

### 6.1.1 プロジェクト環境

| 環境 | プロジェクト名 | 用途 |
|-----|--------------|------|
| 開発 | tetris-game-dev | 開発・テスト用 |
| 本番 | tetris-game-prod | 本番運用 |

### 6.1.2 使用サービス

```
Firebase Project
├── Authentication      # ユーザー認証
├── Cloud Firestore     # データベース
├── Firebase Hosting    # Webホスティング
├── Firebase Analytics  # 分析（任意）
└── Cloud Functions     # サーバーレス関数（拡張用）
```

## 6.2 Firebase Authentication

### 6.2.1 有効化する認証プロバイダー

| プロバイダー | 設定 | 備考 |
|------------|------|------|
| 匿名認証 | 有効 | ゲストプレイ用 |
| Google | 有効 | メインの認証方法 |
| Email/Password | 無効 | 今回は不使用 |

### 6.2.2 認証フロー

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  ゲーム開始  │────▶│  匿名認証   │────▶│  プレイ可能  │
└─────────────┘     └─────────────┘     └─────────────┘
                           │
                           ▼
                    ┌─────────────┐
                    │  Google     │
                    │  アカウント  │
                    │  リンク     │
                    └─────────────┘
                           │
                           ▼
                    ┌─────────────┐
                    │  匿名→正規  │
                    │  アカウント  │
                    │  移行完了   │
                    └─────────────┘
```

### 6.2.3 実装コード例

```typescript
// lib/firebase.ts
import { initializeApp } from 'firebase/app';
import {
  getAuth,
  signInAnonymously,
  signInWithPopup,
  GoogleAuthProvider,
  linkWithPopup,
} from 'firebase/auth';

const firebaseConfig = {
  apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY,
  authDomain: process.env.NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN,
  projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,
  storageBucket: process.env.NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: process.env.NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID,
  appId: process.env.NEXT_PUBLIC_FIREBASE_APP_ID,
};

export const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);

// 匿名認証
export const signInAsGuest = () => signInAnonymously(auth);

// Google認証
export const signInWithGoogle = () => {
  const provider = new GoogleAuthProvider();
  return signInWithPopup(auth, provider);
};

// 匿名アカウントをGoogleアカウントにリンク
export const linkToGoogle = async () => {
  const user = auth.currentUser;
  if (user?.isAnonymous) {
    const provider = new GoogleAuthProvider();
    return linkWithPopup(user, provider);
  }
};
```

## 6.3 Cloud Firestore

### 6.3.1 データモデル

```
firestore/
│
├── users/                          # ユーザーコレクション
│   └── {userId}/
│       ├── displayName: string     # 表示名
│       ├── photoURL: string        # アバターURL
│       ├── highScore: number       # ハイスコア
│       ├── gamesPlayed: number     # プレイ回数
│       ├── createdAt: timestamp    # 作成日時
│       └── updatedAt: timestamp    # 更新日時
│
├── scores/                         # スコアコレクション
│   └── {scoreId}/
│       ├── userId: string          # ユーザーID
│       ├── displayName: string     # 表示名（非正規化）
│       ├── score: number           # スコア
│       ├── level: number           # 到達レベル
│       ├── lines: number           # 消去ライン数
│       ├── duration: number        # プレイ時間（秒）
│       └── createdAt: timestamp    # 記録日時
│
└── stats/                          # 統計コレクション
    └── global/
        ├── totalGames: number      # 総プレイ回数
        ├── totalUsers: number      # 総ユーザー数
        └── updatedAt: timestamp    # 更新日時
```

### 6.3.2 インデックス設定

```
// firestore.indexes.json
{
  "indexes": [
    {
      "collectionGroup": "scores",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "score", "order": "DESCENDING" },
        { "fieldPath": "createdAt", "order": "DESCENDING" }
      ]
    },
    {
      "collectionGroup": "scores",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "userId", "order": "ASCENDING" },
        { "fieldPath": "createdAt", "order": "DESCENDING" }
      ]
    }
  ]
}
```

### 6.3.3 Security Rules

```javascript
// firestore.rules
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // ユーザードキュメント
    match /users/{userId} {
      // 誰でも読み取り可能
      allow read: if true;

      // 本人のみ作成・更新可能
      allow create: if request.auth != null
                    && request.auth.uid == userId
                    && validUserData(request.resource.data);

      allow update: if request.auth != null
                    && request.auth.uid == userId
                    && validUserUpdate(request.resource.data);

      allow delete: if false;
    }

    // スコアドキュメント
    match /scores/{scoreId} {
      // 誰でも読み取り可能
      allow read: if true;

      // 認証済みユーザーのみ作成可能（自分のスコアのみ）
      allow create: if request.auth != null
                    && request.resource.data.userId == request.auth.uid
                    && validScoreData(request.resource.data);

      // 更新・削除は不可
      allow update, delete: if false;
    }

    // 統計ドキュメント（読み取りのみ）
    match /stats/{document} {
      allow read: if true;
      allow write: if false;
    }

    // ヘルパー関数
    function validUserData(data) {
      return data.keys().hasAll(['displayName', 'createdAt'])
             && data.displayName is string
             && data.displayName.size() <= 50;
    }

    function validUserUpdate(data) {
      return data.diff(resource.data).affectedKeys()
             .hasOnly(['displayName', 'photoURL', 'highScore', 'gamesPlayed', 'updatedAt']);
    }

    function validScoreData(data) {
      return data.keys().hasAll(['userId', 'score', 'level', 'lines', 'createdAt'])
             && data.score is number
             && data.score >= 0
             && data.score <= 999999999
             && data.level is number
             && data.level >= 1
             && data.lines is number
             && data.lines >= 0;
    }
  }
}
```

### 6.3.4 クエリ例

```typescript
// lib/firestore.ts
import {
  getFirestore,
  collection,
  doc,
  setDoc,
  getDoc,
  addDoc,
  query,
  where,
  orderBy,
  limit,
  onSnapshot,
  serverTimestamp,
} from 'firebase/firestore';
import { app } from './firebase';

export const db = getFirestore(app);

// スコア保存
export const saveScore = async (
  userId: string,
  displayName: string,
  score: number,
  level: number,
  lines: number,
  duration: number
) => {
  return addDoc(collection(db, 'scores'), {
    userId,
    displayName,
    score,
    level,
    lines,
    duration,
    createdAt: serverTimestamp(),
  });
};

// トップ10ランキング取得
export const getTopScores = async (count = 10) => {
  const q = query(
    collection(db, 'scores'),
    orderBy('score', 'desc'),
    limit(count)
  );
  // ... 実行
};

// リアルタイムランキング監視
export const subscribeToRanking = (
  callback: (scores: Score[]) => void,
  count = 10
) => {
  const q = query(
    collection(db, 'scores'),
    orderBy('score', 'desc'),
    limit(count)
  );
  return onSnapshot(q, (snapshot) => {
    const scores = snapshot.docs.map((doc) => ({
      id: doc.id,
      ...doc.data(),
    }));
    callback(scores as Score[]);
  });
};
```

## 6.4 Firebase Hosting

### 6.4.1 設定ファイル

```json
// firebase.json
{
  "hosting": {
    "public": "out",
    "ignore": ["firebase.json", "**/.*", "**/node_modules/**"],
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ],
    "headers": [
      {
        "source": "**/*.@(js|css)",
        "headers": [
          {
            "key": "Cache-Control",
            "value": "public, max-age=31536000, immutable"
          }
        ]
      },
      {
        "source": "**/*.@(jpg|jpeg|gif|png|svg|webp|ico)",
        "headers": [
          {
            "key": "Cache-Control",
            "value": "public, max-age=31536000, immutable"
          }
        ]
      }
    ]
  }
}
```

### 6.4.2 デプロイコマンド

```bash
# ビルド
npm run build

# Firebase にデプロイ
firebase deploy --only hosting

# プレビューチャンネル（ステージング）
firebase hosting:channel:deploy preview
```

## 6.5 Cloud Functions（拡張用）

### 6.5.1 想定する関数

| 関数名 | トリガー | 用途 |
|-------|---------|------|
| onScoreCreate | Firestore onCreate | ハイスコア更新、統計更新 |
| updateRanking | Scheduled | ランキングキャッシュ更新 |
| cleanupOldScores | Scheduled | 古いスコアの削除 |

### 6.5.2 実装例

```typescript
// functions/src/index.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

admin.initializeApp();
const db = admin.firestore();

// スコア作成時にユーザーのハイスコアを更新
export const onScoreCreate = functions.firestore
  .document('scores/{scoreId}')
  .onCreate(async (snap, context) => {
    const score = snap.data();
    const userRef = db.doc(`users/${score.userId}`);

    const userDoc = await userRef.get();
    const userData = userDoc.data();

    if (!userData || score.score > (userData.highScore || 0)) {
      await userRef.update({
        highScore: score.score,
        gamesPlayed: admin.firestore.FieldValue.increment(1),
        updatedAt: admin.firestore.FieldValue.serverTimestamp(),
      });
    } else {
      await userRef.update({
        gamesPlayed: admin.firestore.FieldValue.increment(1),
        updatedAt: admin.firestore.FieldValue.serverTimestamp(),
      });
    }
  });
```

## 6.6 コスト見積もり

### 6.6.1 Firebase 無料枠（Spark プラン）

| サービス | 無料枠 | 想定使用量 |
|---------|-------|-----------|
| Authentication | 無制限 | - |
| Firestore 読み取り | 50,000/日 | ランキング表示 |
| Firestore 書き込み | 20,000/日 | スコア保存 |
| Firestore ストレージ | 1GB | スコアデータ |
| Hosting ストレージ | 10GB | 静的ファイル |
| Hosting 転送 | 360MB/日 | ゲームアセット |

### 6.6.2 スケーリング時の費用（Blaze プラン）

| サービス | 単価 | 備考 |
|---------|------|------|
| Firestore 読み取り | $0.06/100,000 | 無料枠超過分 |
| Firestore 書き込み | $0.18/100,000 | 無料枠超過分 |
| Hosting 転送 | $0.15/GB | 無料枠超過分 |
| Cloud Functions | $0.40/100万回 | 呼び出し |

## 6.7 環境変数

### 6.7.1 開発環境（.env.local）

```bash
# Firebase Configuration
NEXT_PUBLIC_FIREBASE_API_KEY=your-api-key
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=your-project.firebaseapp.com
NEXT_PUBLIC_FIREBASE_PROJECT_ID=your-project-id
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=your-project.appspot.com
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=your-sender-id
NEXT_PUBLIC_FIREBASE_APP_ID=your-app-id

# Environment
NEXT_PUBLIC_ENV=development
```

### 6.7.2 本番環境

- GitHub Secrets または Firebase 環境変数で管理
- CI/CD パイプラインで注入

## 6.8 監視・運用

### 6.8.1 Firebase Console で監視

- Authentication: アクティブユーザー数
- Firestore: 読み取り/書き込み数、ストレージ使用量
- Hosting: リクエスト数、帯域幅
- Analytics: ユーザー行動

### 6.8.2 アラート設定

- 使用量が無料枠の80%に達したら通知
- エラー率が閾値を超えたら通知
