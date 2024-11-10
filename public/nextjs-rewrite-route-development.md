---
title: Next.js で /api で始まるすべてのリクエストを別サーバーに回す
tags:
  - Next.js
private: false
updated_at: '2024-11-10T13:52:07+09:00'
id: 5e78b00ebba16d29b57b
organization_url_name: null
slide: false
ignorePublish: false
---

開発中など、 `api` で始まるパスは .Net や Go で作成したサーバーでハンドルさせたい場合に便利。

ここでは、開発中は `api` で始まるパスを `http://localhost:5001` にプロキシさせることを目的とする。

# バージョン

* node: 22.11.0
* npm: 10.9.0
* next: 15.0.2

# ひとまずプロジェクトの作成

以下のコマンドで Next.js プロジェクトを作成する。

```bash
npx create-next-app@latest
```

ここではプロジェクト名を `client` ととした。それ以外はデフォルト。以下実行例。

```text
What is your project named? client
Would you like to use TypeScript? No / Yes <= Yes
Would you like to use ESLint? No / Yes <= Yes
Would you like to use Tailwind CSS? No / Yes <= No
Would you like your code inside a `src/` directory? No / Yes <= Yes
Would you like to use App Router? (recommended) No / Yes <= Yes
Would you like to use Turbopack for `next dev`?  No / Yes <= No
Would you like to customize the import alias (`@/*` by default)? No / Yes <= No
What import alias would you like configured? @/*
```

プロジェクト作成が完了すると `client` ディレクトリ内に `package.json` を含むいろんなファイルが生成される。
このディレクトリ内で `npm run dev` を実行すると Next.js プロジェクトを実行できる。

# テスト用のコンポーネントを作成する

以下のような、ボタンを押したら `/api/system/ping` に GET リクエストを送信して、リスポンスをコンソールに出力するだけのクライアントコンポーネントを作成する。

```tsx:components/TestFetchButton.tsx
"use client";

export default function TestFetchButton() {
  const fetchData = async () => {
    console.log("fetching data");
    const res = await fetch("/api/system/ping", { method: "GET" });
    if (res.ok) {
      console.log(await res.json());
    } else {
      console.log(res.statusText, res.status);
    }
  };
  return (
    <button type="button" onClick={fetchData}>
      フェッチ！
    </button>
  );
}
```

次に `app/page.tsx` を以下のように変更して、作成してテスト用コンポーネントをレンダリングする。

```tsx
import TestFetchButton from "@/components/TestFetchButton";
import styles from "./page.module.css";

export default function Home() {
  return (
    <div className={styles.page}>
      <main className={styles.main}>
        <div>TEST</div>
        <div>
          <TestFetchButton/>
        </div>
      </main>
    </div>
  );
}
```

これで実行すると「フェッチ！」というボタンが出現する。
現状では `api/system/ping` というパスをハンドルする実装やプロキシの設定を入れていないので、ボタンを押しても 404 エラーが返ってくる。

# 設定に `rewrites` を追加する

開発中に限り `/api` にきたリクエストを `http://localhost:5001` に回したい。
そのためには `next.config.ts` を以下のように修正する。

```ts
import type { NextConfig } from "next";
import { PHASE_DEVELOPMENT_SERVER } from "next/constants";

const nextConfig: NextConfig = {
  output: 'export'
  /* config options here */
};

const getConfig = (phase: string): NextConfig => {
  if (phase === PHASE_DEVELOPMENT_SERVER) {
    return {
      ...nextConfig,
      rewrites: async () => {
        return [
          {
            source: "/api/:path*",
            destination: "http://localhost:5001/api/:path*",
          },
        ];
      }
    };
  }
  return nextConfig;
}

export default getConfig;
```

ポイントをまとめると、、、

## デフォルト設定を定義しておく

各環境で共通する設定を `NextConfig` 型のオブジェクトとして定義する。

ここでは `rewrites` を含めない。

## 文字列を引数で受け取って設定を返す関数をエクスポートする

引数で文字列を受け取るようにすると、各実行でのフェーズが渡されてくる。
利用可能なフェーズは以下参考。

<https://github.com/vercel/next.js/blob/5e6b008b561caf2710ab7be63320a3d549474a5b/packages/next/shared/lib/constants.ts#L19-L23>

ここでは開発のフェーズである `PHASE_DEVELOPMENT_SERVER` であった場合に別の設定を返すようにする。

## 文字列が開発フェーズの場合にデフォルト設定の一部を変更する

ここでは `/api/` で始まるすべてのリクエストを `http://localhost:5001` にパスをそのままくっつけて回すようにしている。
この設定により `/api/system/ping` へのリクエストは `http://localhost:5001/api/system/ping` に書き換えられる。

`rewrites` の詳しい書き方は以下参考。

<https://nextjs.org/docs/app/api-reference/next-config-js/rewrites>

# 実行

これであとは `npm run dev` で実行するだけ。
設定した `http://localhost:5001` をリッスンしているサーバーがあれば、そのサーバーでリクエストがハンドルされて結果が返される。
