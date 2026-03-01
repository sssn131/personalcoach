# 体重ログ PWA — Next.js セットアップガイド

## 1. プロジェクト作成

```bash
npx create-next-app@latest weight-log --typescript --app --tailwind=no --eslint=no
cd weight-log
```

## 2. ファイル配置

```
weight-log/
├── app/
│   ├── layout.tsx       ← 下記参照
│   ├── page.tsx         ← index.html の中身を React 化
│   └── globals.css      ← CSS変数とグローバルスタイル
├── public/
│   ├── manifest.json    ← 同梱の manifest.json をコピー
│   ├── sw.js            ← 同梱の sw.js をコピー
│   ├── icon-192.png     ← アイコン画像（下記ツールで生成）
│   └── icon-512.png
└── next.config.js
```

## 3. app/layout.tsx

```tsx
import type { Metadata } from 'next';
import './globals.css';

export const metadata: Metadata = {
  title: '体重ログ',
  description: '毎日の体重・食事・運動を記録',
  manifest: '/manifest.json',
  appleWebApp: {
    capable: true,
    statusBarStyle: 'black-translucent',
    title: '体重ログ',
  },
  themeColor: '#1a1a2e',
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="ja">
      <head>
        <link rel="apple-touch-icon" href="/icon-192.png" />
        <meta name="apple-mobile-web-app-capable" content="yes" />
      </head>
      <body>{children}</body>
    </html>
  );
}
```

## 4. Service Worker 登録（app/page.tsx の先頭に追加）

```tsx
'use client';
import { useEffect } from 'react';

export default function Page() {
  useEffect(() => {
    if ('serviceWorker' in navigator) {
      navigator.serviceWorker.register('/sw.js');
    }
  }, []);
  // ... rest of component
}
```

## 5. アイコン生成

以下のどちらかで PNG アイコンを生成してください：
- [PWA Builder Image Generator](https://www.pwabuilder.com/imageGenerator)
- [Favicon.io](https://favicon.io)

文字「体」や体重計の絵文字ベースで 192×192 / 512×512 の PNG を作成し `public/` に配置。

## 6. next.config.js（ヘッダー設定）

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  async headers() {
    return [
      {
        source: '/sw.js',
        headers: [
          { key: 'Service-Worker-Allowed', value: '/' },
          { key: 'Cache-Control', value: 'no-cache' },
        ],
      },
    ];
  },
};

module.exports = nextConfig;
```

## 7. 起動

```bash
npm run dev   # 開発
npm run build && npm start  # 本番（PWA の Service Worker は本番でのみ推奨）
```

## iPhone での「ホーム画面に追加」手順

1. Safari で URL を開く
2. 共有ボタン（□↑）をタップ
3. 「ホーム画面に追加」を選択
4. 「追加」でアプリとしてインストール完了

---

## 判定ロジック仕様

| 条件 | スコア変動 |
|------|-----------|
| 体重入力あり | +1（最低△） |
| トレーニング実施 OR 8,000歩以上 | +1 |
| 夕食にたんぱく質テンプレ | +1 |
| 外に出ない＋歩数0＋高糖質夕食 | -1 |

| スコア | 判定 |
|--------|------|
| 3以上 | ◎ |
| 2 | ◯ |
| 1 | △ |
