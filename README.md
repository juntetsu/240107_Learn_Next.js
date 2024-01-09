# 学習メモ

## clsx

[参考サイト]("https://qiita.com/gotomeltdown/items/11bfa9c17cf820eb3ccf")

例えば、stateが"error"の時は文字色を赤、"success"の時は緑にする。  
などのpropsやstateに基づいて動的にクラス名を変更したい時に役立つ。

```javascript
<div
  className={clsx('text-2xl font-bold', {
    'text-red-400': state === 'error',
    'text-green-400': state === 'success',
  })}
>
  テキスト
</div>
```

## `<Layout>`

`<Layout>`を使用する利点の1つは、`page`コンポーネントだけが更新されて、`layout`コンポーネントは再レンダリングされない（部分レンダリング）

**ルートレイアウト**は必須！！

## `<Link>`

`<a>`タグではなく`<Link>`タグを使うことで、遷移時にページ全体が再レンダリングされることなく、ユーザーが新しいルートに移動しても、ブラウザはページをリロードせず、変更されたルートセグメントだけが再レンダリングされる。

### アクティブなリンクを表示するUI

- `next/navigation`から`usePathname()`を使用（現在のURLのパス名を読み取るためのクライアントコンポーネントフック）
- 先頭に`use client`必要
- `clsx`ライブラリを使って、リンクがアクティブな時にクラスを適用させる。

```javascript
'use client';

import {
  UserGroupIcon,
  HomeIcon,
  DocumentDuplicateIcon,
} from '@heroicons/react/24/outline';
import Link from 'next/link';
import clsx from 'clsx';
import { usePathname } from 'next/navigation';

// Map of links to display in the side navigation.
// Depending on the size of the application, this would be stored in a database.
const links = [
  { name: 'Home', href: '/dashboard', icon: HomeIcon },
  {
    name: 'Invoices',
    href: '/dashboard/invoices',
    icon: DocumentDuplicateIcon,
  },
  { name: 'Customers', href: '/dashboard/customers', icon: UserGroupIcon },
];

export default function NavLinks() {
  const pathname = usePathname();

  return (
    <>
      {links.map((link) => {
        const LinkIcon = link.icon;
        return (
          <Link
            key={link.name}
            href={link.href}
            className={clsx(
              'flex h-[48px] grow items-center justify-center gap-2 rounded-md bg-gray-50 p-3 text-sm font-medium hover:bg-sky-100 hover:text-blue-600 md:flex-none md:justify-start md:p-2 md:px-3',
              {
                'bg-sky-100 text-blue-600': pathname === link.href,
              },
            )}
          >
            <LinkIcon className="w-6" />
            <p className="hidden md:block">{link.name}</p>
          </Link>
        );
      })}
    </>
  );
}
```

## データベース（@vercel/postgres）

まずは、`vercel`にサインイン→デプロイ

### DB作成

1. Continue to Dashboardをクリック
2. プロジェクトダッシュボードからStorage
3. Connect Store → Create New → Postgres → Continue
4. 任意のデータベース名→リージョンはワシントン（後からリージョンは変更不可。24.01現在Japanはなし）

### .env（.local）ファイル

1. 作成が終わったら`.env.local`タブに移動し`Show secret`→`Copy Snippet`
2. コードエディターに移動し`.env`ファイル作成→コピーしたもの貼り付け
3. `.gitignore`ファイルに`.env`記述忘れずに
4. ターミナルで`npm i @vercel/postgres`を実行し、`Vercel Postgres SDK`をインストールする。

### 初期データ挿入

seedとやら  
[Nextjs.org](https://nextjs.org/learn/dashboard-app/setting-up-your-database)参照

## 🔰Fetching Data

DB作成〜

ウォーターフォール（前の処理が終わるまで次の処理が実行されない。=待つ時間が発生する）を避ける方法。  
→JavaScriptでは、Promise.all()またはPromise.allSettled()関数を使用して、すべてのプロミスを同時に開始することができる。

## 🔰Static and Dynamic Rendering

- ウォーターフォール（前の処理が終わるまで待つ必要がある。↔︎ データ取得を同時に実行する"Parallel Data Fetching"）
- 静的レンダリングなので、データの更新はアプリケーションに反映されない

### 静的レンダリングとは

> 静的レンダリングでは、データのフェッチとレンダリングは、ビルド時（デプロイ時）または再検証時にサーバー上で行われる。その後、結果をコンテンツ・デリバリー・ネットワーク（CDN）に配信し、キャッシュすることができます。

利点

- 高速
- サーバー負荷の軽減
- SEO対策

更新頻度の低いブログや製品ページなど。

### 動的レンダリングとは

> リクエスト時（ユーザーがページにアクセスした時）に、各ユーザーに対してサーバー上でコンテンツがレンダリングされます。

#### 利点

- リアルタイムデータ
- User-Specific Content
- Request Time Information

### データの取得を静的→動的に

next/cacheからunstable_noStoreをインポート

```javascript
import { unstable_noStore as noStore } from 'next/cache';

export async function fetchRevenue() {
  // Add noStore() here to prevent the response from being cached.
  // This is equivalent to in fetch(..., {cache: 'no-store'}).
  noStore();

  // ...
}
```

しかし、極端に遅い処理がひとつでもある場合、結果的にページ全体の処理が遅くなる。

## 🔰Streaming

https://nextjs.org/learn/dashboard-app/streaming

前の章では、ダッシュボード・ページをダイナミックにしましたが、データ・フェッチの遅さがアプリケーションのパフォーマンスにどのような影響を与えるかについて説明しました。ここでは、遅いデータリクエストがあったときにユーザーエクスペリエンスを改善する方法を見ていきましょう。

Next.js の App Router では `<Suspense>` を使ったストリーミングがサポートされている。  
`<Suspense>` を使ったストリーミングではページの HTML を小さな塊に分解し、その塊をクライアントに順次送信でき、この `<Suspense>` を使うことで、ページの一部をより早く表示できる。

![Alt text](streaming.png)

以下でやること

1. ページレベルで`loading.tsx`ファイルを使用
2. 特定のコンポーネントでは`<Suspense>`を使用する

### loading.tsx

データ読み込みが完了するまで「Loading...」と表示させる。

loading.tsx

```javascript
export default function Loading() {
  return <div>Loading...</div>;
}
```

ローディングスケルトンを表示させる例
skeltons.tsx

```javascript
export default function DashboardSkeleton() {
  return (
    <>
      <div
        className={`${shimmer} relative mb-4 h-8 w-36 overflow-hidden rounded-md bg-gray-100`}
      />
      <div className="grid gap-6 sm:grid-cols-2 lg:grid-cols-4">
        <CardSkeleton />
        <CardSkeleton />
        <CardSkeleton />
        <CardSkeleton />
      </div>
      <div className="mt-6 grid grid-cols-1 gap-6 md:grid-cols-4 lg:grid-cols-8">
        <RevenueChartSkeleton />
        <LatestInvoicesSkeleton />
      </div>
    </>
  );
}
```

loading.tsx

```javascript
import DashboardSkeleton from '@/app/ui/skeletons';

export default function Loading() {
  return <DashboardSkeleton />;
}
```

こんな感じ
![Alt text](public/skeletons.png)

## ルートにおける()の意味

例えば上記の例で説明すると、ルートディレクトリが以下のようになっているとする。

```
dashboard
├─ invoices
│ 　└─ page.tsx
├─ customers
│ 　└─ page.tsx
├─ loading.tsx
└─ page.tsx
```

この場合、dashboardのみならず`invoices` と`customers`ページにもスケルトン（loading.tsx）が適用されてしまう。
そこで利用されるのが`Route Groups`といって、`dashboard`フォルダ内に`/(overview)`と新しいフォルダを作成し、その中にファイルを移動させることで、URLパス構造に影響を与えることがなくなる。

```
dashboard
├─ (overview)
│ 　├─ loading.tsx
│ 　└─ page.tsx
├─ invoices
│ 　└─ page.tsx
├─ customers
│ 　└─ page.tsx
├─ loading.tsx
└─ page.tsx
```

## Suspenseを使って、コンポーネント単位でストリーミングさせる
処理の遅いコンポーネントだけをストリーミングすることが可能。

1. Reactから`Suspense`をインポート
2. ストリーミングさせたいコンポーネントを`<Suspense>`で囲む
3. `fallback`でフォールバックコンポーネント（スケルトン）を渡せる