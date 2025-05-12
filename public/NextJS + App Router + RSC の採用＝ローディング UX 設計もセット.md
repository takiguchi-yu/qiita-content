---
title: NextJS + App Router + RSC の採用＝ローディング UX 設計もセット
tags:
  - Next.js
private: false
updated_at: '2025-05-12T21:32:36+09:00'
id: 1012670ec8c514c69fc5
organization_url_name: null
slide: false
ignorePublish: false
---

# NextJSのページ遷移UX問題と解決策

## 😵 問題：ユーザーが混乱するページ遷移

- NextJS App Router + RSCでは**ページ遷移時の視覚的フィードバックが不足**
- ユーザーにとって以下が分かりづらい：
  - ページが遷移中なのか
  - データをロード中なのか
  - それとも単に何も起きていないのか

以下はブラウザのリロード中の図です。これによりユーザーは「何かが起きている」と思えるのですが、RSCを使用しているとこのようなリロードが発生しません。

![ブラウザのリロード中の図](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/59081/c17202e6-e2a6-4220-9cab-77910de4c1cf.png)

## 🔍 原因：RSCのフェッチ方式

- **App Router + RSCの場合**、ページ遷移が通常のHTMLリロードではなく**fetch**で行われる
- パフォーマンス向上のため、**変更箇所のみを最適化して取得する仕組み**
- ブラウザの開発者ツールで確認すると `/?_rsc=*****` というリクエストが発生

![RSCフェッチリクエスト](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/59081/e7cc5c9f-e113-4a6d-9144-dd475e89c777.png)

### RSCリクエストの仕組み

| 項目            | 説明                                                             |
| --------------- | ---------------------------------------------------------------- |
| `?_rsc=` の正体 | React Server Components用のストリーミングデータフェッチ用内部API |
| 発生タイミング  | App Router (`/app`ディレクトリ)でServer Componentsを使用時       |
| 用途            | サーバーコンポーネントのデータストリームをクライアントに送信     |
| 特徴            | 完全なページリロードなしで差分のみ更新                           |

:::note warn
Next.js App Router + RSCのページ遷移では、従来のような「完全なリロード」がないため、**ユーザーにとって視覚的なフィードバックが不足**し、体験が悪化しがちです。
:::

## 🎯 問題の本質

| 要因                       | 詳細                                                           |
| -------------------------- | -------------------------------------------------------------- |
| **fetch()ベースの遷移**    | ブラウザのローディングインジケーターが表示されない             |
| **差分更新の仕組み**       | ページ全体ではなく変更部分だけ更新されるため変化が分かりにくい |
| **視覚フィードバック不足** | 特にサーバー処理が遅い場合、ページが固まったように見える       |
| **ユーザー誤解**           | 「遅延＝バグ」と誤解され、離脱につながりやすい                 |

## ✅ 解決策：3段階のローディングUI設計

### 1. ページ全体のローディング表示（`loading.tsx`）

該当フォルダに`loading.tsx`を配置するだけで、遷移中の全体ローディングUIを設定できます。

```jsx
// app/site/detail/loading.tsx
export default function Loading() {
  return (
    <div className="p-8 text-center">
      <p>読み込み中です…</p>
    </div>
  );
}
```

### 2. プログラムによる遷移時の状態表示（`useTransition`）

ボタンクリックなどでページ遷移させる場合は、明示的に遷移中状態を管理します。

```jsx
'use client';
import { useTransition } from 'react';
import { useRouter } from 'next/navigation';

export function NavigateButton({ id }) {
  const router = useRouter();
  const [isPending, startTransition] = useTransition();

  const handleClick = () => {
    startTransition(() => {
      router.push(`/site/detail/${id}`);
    });
  };

  return (
    <button onClick={handleClick} disabled={isPending}>
      {isPending ? '読み込み中...' : '詳細を見る'}
    </button>
  );
}
```

### 3. 部分的なローディング表示（`Suspense`）

ページ内の特定コンテンツだけを非同期で表示する場合に使用します。

```jsx
// app/site/detail/[id]/page.tsx
import { Suspense } from 'react';
import { DetailContent } from './DetailContent';
import { Skeleton } from '@/components/ui/Skeleton';

export default function DetailPage({ params }) {
  return (
    <Suspense fallback={<Skeleton />}>
      <DetailContent id={params.id} />
    </Suspense>
  );
}
```

## 💡 重要なポイント

| ローディング種別       | 実装方法        | 適用場面                               |
| ---------------------- | --------------- | -------------------------------------- |
| グローバルローディング | `loading.tsx`   | ページ全体の遷移時                     |
| コンポーネント単位     | `<Suspense>`    | ページ内の一部だけ非同期読み込み時     |
| アクション単位         | `useTransition` | ボタンクリックなどの操作に紐づく遷移時 |

## 📝 結論：RSC採用＝ローディングUX設計は必須

**App Router + RSCの高速な差分更新の仕組みは、適切なローディングUIがセットになって初めて真価を発揮します**。

ユーザー視点では：

- ✅ 何が起きているかが常に分かる
- ✅ システムが反応していることが視覚的に確認できる
- ✅ 離脱防止につながる

**Next.js App Router + RSC の採用 = ローディングUX設計もセットで考える**。これはもはや必須の設計原則です。
