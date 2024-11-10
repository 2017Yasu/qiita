---
title: .NET ウェブサーバーから静的ファイルを提供する
tags:
  - .NET
  - ASP.NET
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

以下の記事の続きからやっている前提として進める。

<https://qiita.com/2017Yasu/items/0deaf14cfa23e58a8085>

現在のディレクトリ構成はこんな感じ。

```text
.
├── .editorconfig
└── src
    └── server
        ├── .gitignore
        ├── TestWebApp.Server.Api
        │   ├── Controllers
        │   │   └── SystemController.cs
        │   ├── Program.cs
        │   ├── Properties
        │   │   └── launchSettings.json
        │   ├── TestWebApp.Server.Api.csproj
        │   ├── TestWebApp.Server.Api.http
        │   ├── appsettings.Development.json
        │   └── appsettings.json
        └── TestWebApp.Server.sln
```

# 静的ファイルをサーブする

以下のコードを `Program.cs` ファイルに追加。

```diff
 app.UseHttpsRedirection();

+app.UseStaticFiles();

 app.UseAuthorization();
```

これにより `wwwroot` ディレクトリ内のファイルを `localhost:{PORT}/path/to/file.html` などのアドレスで取得することができる。

一方で `localhost:{PORT}/` などのファイルを直接指定しないアドレスでは何も返してこない。この時に `index.html` などのデフォルトのファイルを返すようにするには以下のコードを追記する。

```diff
 app.UseHttpsRedirection();

+app.UseDefaultFiles();
 app.UseStaticFiles();

 app.UseAuthorization();
```

試しに以下のようなファイルを `wwwroot/index.html` として作成して実行すると `localhost:{PORT}` で作成したページを参照することができる。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Test</title>
</head>
<body>
    <h1>Test</h1>
</body>
</html>
```

# クライアントをフレームワークを使って実装する

ここではクライアントファイルを [Next.js](https://nextjs.org/) を使って構築して、サーバーと繋げてみます。

node はインストールされているものとする。

以下のコマンドを `src` ディレクトリで実行して Next.js プロジェクトを作成する。

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

現在のフォルダ構成は以下のような感じ。

```text
.
└── src
    ├── client
    │   ├── README.md
    │   ├── next-env.d.ts
    │   ├── next.config.ts
    │   ├── package-lock.json
    │   ├── package.json
    │   ├── public
    │   │   ├── file.svg
    │   │   ├── globe.svg
    │   │   ├── next.svg
    │   │   ├── vercel.svg
    │   │   └── window.svg
    │   ├── src
    │   │   └── app
    │   │       ├── favicon.ico
    │   │       ├── fonts
    │   │       │   ├── GeistMonoVF.woff
    │   │       │   └── GeistVF.woff
    │   │       ├── globals.css
    │   │       ├── layout.tsx
    │   │       ├── page.module.css
    │   │       └── page.tsx
    │   └── tsconfig.json
    └── server
        ├── TestWebApp.Server.Api
        │   ├── Controllers
        │   │   └── SystemController.cs
        │   ├── Program.cs
        │   ├── Properties
        │   │   └── launchSettings.json
        │   ├── TestWebApp.Server.Api.csproj
        │   ├── TestWebApp.Server.Api.http
        │   ├── appsettings.Development.json
        │   ├── appsettings.json
        │   └── wwwroot
        │       └── index.html
        └── TestWebApp.Server.sln
```

Next.js の場合は静的ページを生成するためには以下のように `next.config.ts` を修正する必要がある ([参考](https://nextjs.org/docs/app/building-your-application/deploying/static-exports))。

```ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  output: 'export'
  /* config options here */
};

export default nextConfig;
```

以下のコマンドでビルドする。

```bash
npm run build
```

すると `out` ディレクトリが `client` ディレクトリ下にできて、その中に `html` や `js` ファイルなどが生成される。これを `server/TestWebApp.Server.Api/wwwroot` ディレクトにコピーすればサーバー側でサーブできるようになる。

## 開発のための設定

### サーバー側

開発中何度もコピーするでは面倒なので、ここではシンボリックを使って `server/TestWebApp.Server.Api/wwwroot` と `client/out` ディレクトリを繋げる。

`server/TestWebApp.Server.Api` ディレクトリに移動して、ひとまず `wwwroot` ディレクトリを削除する。

```bash
rm -rf wwwroot
```

以下のコマンドでシンボリックリンクを作成する。

```bash
ln -s ../../client/out wwwroot
```

すると `wwwroot` という名前で `../../client/out` にアクセスすることができる。

この状態で `dotnet run` を実行して `http://localhost:{PORT}` にアクセスすると Next.js のスタートページが表示される。

※ Windows で同様のシンボリックリンクを作成するには以下のコマンドを管理者権限で実行する ([参考](https://dev.classmethod.jp/articles/make_windows_symbolic_link/))。

```powershell
mklink /D wwwroot ../../client/out
```

### クライアント側

`api` で始まるパスは .Net サーバー側で、それ以外はフレームワーク側で処理したいなどが多くあると考えられる。
そういう時は rewrite の設定やプロキシサーバーを利用する。

Next.js の場合のやり方は以下参考。

[Next.js で api で始まるリクエストは別サーバーに回す](./option01-NextJsRewrite.md)
