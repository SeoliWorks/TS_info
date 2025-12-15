<br>

▷"Type Sharing"ってなに？

<br>

▶TypeScriptにおける**Type Sharing（型共有）とは、主にフロントエンド（クライアント）とバックエンド（サーバー）の間で、TypeScriptの型定義（インターフェースや型エイリアス）を共有する手法**のことです。

通常、APIのレスポンスの型などはサーバー側とクライアント側でそれぞれ定義しがちですが、Type Sharingを行うことで「一つの真実（Single Source of Truth）」を作り、開発効率と堅牢性を劇的に向上させることができます。

-----

### 1\. なぜType Sharingをするのか？（メリット）

  * **整合性の保証:** サーバー側のレスポンス形式が変わった瞬間、フロントエンド側でコンパイルエラーが出るため、バグを未然に防げます。
  * **開発効率の向上:** フロントエンド開発時に、バックエンドの型情報に基づいた自動補完（IntelliSense）が効くようになります。
  * **二重管理の解消:** 「サーバー用の型」と「クライアント用の型」を別々に書く必要がなくなります。

-----

### 2\. Type Sharingの主な実現パターン

大きく分けて3つのアプローチがあります。

#### A. モノレポ（Monorepo）での共有（標準的）

プロジェクト全体を1つのリポジトリで管理し（モノレポ）、`shared` や `packages/types` といった共有フォルダを作って参照する方法です。

  * **構成例:**
      * `apps/backend` (サーバー)
      * `apps/frontend` (Webアプリ)
      * `packages/shared` (ここに型定義を置く)
  * **仕組み:** バックエンドとフロントエンドの両方が `packages/shared` をインポートして使います。

#### B. tRPCなどの専用フレームワークを使う（最強の型安全性）

[tRPC](https://trpc.io/) は、Type Sharingを前提に設計されたライブラリです。
APIのエンドポイント定義自体から型を推論し、フロントエンドに自動的に渡します。「型定義ファイルを書く」という作業すら不要になります。

#### C. スキーマ駆動（OpenAPI / GraphQL）

OpenAPI (Swagger) や GraphQL のスキーマ定義ファイルから、TypeScriptの型を自動生成（Codegen）して共有します。言語が異なる場合（例: Go製のサーバーとTS製のフロント）によく使われます。

-----

### 3\. 具体的なコードイメージ（モノレポの例）

「ユーザー情報を取得する」というシンプルな例で見てみましょう。

**① 共有フォルダ (`packages/shared/types.ts`)**
ここで型を定義します。

```typescript
export interface User {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user';
}
```

**② バックエンド (`apps/backend/src/index.ts`)**
共有された型を使ってレスポンスを返します。

```typescript
import { User } from '@my-app/shared'; // 共有パッケージから読み込み

app.get('/api/user', (req, res) => {
  // 型注釈をつけることで、返すデータが間違っていたらエラーになる
  const user: User = {
    id: '1',
    name: 'Gemini',
    email: 'test@example.com',
    role: 'admin',
  };
  res.json(user);
});
```

**③ フロントエンド (`apps/frontend/src/App.tsx`)**
同じ型を使ってデータを受け取ります。

```typescript
import { useState, useEffect } from 'react';
import { User } from '@my-app/shared'; // 全く同じ型を読み込み

export const App = () => {
  const [user, setUser] = useState<User | null>(null);

  useEffect(() => {
    fetch('/api/user')
      .then(res => res.json())
      .then((data: User) => setUser(data));
  }, []);

  if (!user) return <div>Loading...</div>;

  // ここで user.nmae とタイポしたらエディタが警告してくれる
  return <div>Hello, {user.name}</div>;
};
```

-----

### 4\. 発展：Zodを使ったランタイム型共有

最近のトレンドとして、TypeScriptの型（静的）だけでなく、\*\*バリデーションロジック（動的）\*\*も共有するケースが増えています。`Zod` などのライブラリを使います。

  * **共有ファイル:** `const UserSchema = z.object({ ... })` を定義。
  * **バックエンド:** リクエストの検証（バリデーション）に使う。
  * **フロントエンド:** フォームの入力チェックに使う。
  * **型定義:** `type User = z.infer<typeof UserSchema>` で型も自動生成。

-----

### まとめ

**Type Sharing**とは、\*\*「サーバーとクライアントの境界線をTypeScriptで溶かす技術」\*\*と言えます。これにより、フルスタック開発における「API仕様の認識齟齬」によるバグをほぼ撲滅できます。

現在、ご自身のプロジェクトでバックエンドとフロントエンドの両方をTypeScriptで書かれていますか？

**次のステップのご提案:**
もしよろしければ、\*\*「現在想定しているプロジェクト構成（例：Next.js + NestJS、あるいは完全新規など）」\*\*を教えていただければ、その構成に最適なType Sharingの設定方法（フォルダ構成や`tsconfig.json`の設定など）をコード付きで提案できますがいかがでしょうか？
