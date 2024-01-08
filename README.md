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
