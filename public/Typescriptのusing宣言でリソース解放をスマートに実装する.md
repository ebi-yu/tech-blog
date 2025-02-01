# 初めに  
  
`using`構文について調べたことを備忘録的に記事にします。  
  
https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-2.html  
  
# `using`とは  
  
**TypeScript 5.2で導入された、明示的なリソース管理のための変数宣言です。**  
  
constやletのように変数を宣言する際に使用します。  
  
usingではスコープ終了時にリソースを安全に解放するためのフックメソッド（`[Symbol.dispose]`や`[Symbol.asyncDispose]`）を提供しています。  
  
`[Symbol.dispose]`は同期処理、`[Symbol.asyncDispose]`は非同期処理において使います。  
  
## 主な特徴  
  
- **スコープ終了時に特定の処理を自動実行**  
  - `[Symbol.dispose]`や`[Symbol.asyncDispose]`を使うことで、スコープ終了をフックして特定の処理を行えます。  
- **一回のみ実行**  
  - フックに登録された処理は、スコープ終了時に一度だけ呼び出されます。  
- **エラー処理との連携**  
  - フックに登録された処理内でエラーがスローされた場合、そのエラーはスコープ外で処理されます。  
   
## `using`を使うべきシチュエーション  
  
- **リソースの明示的な解放が必要な場面**  
  - ファイルハンドルの解放  
  - データベース接続の管理  
  - イベントリスナーやタイマー  
  - websockerなどのネットワーク接続  
  
# 使い方と具体的な挙動  
  
実際の挙動と導入手順について記載します。  
  
## 基本的な動作  
  
1. オブジェクトに`[Symbol.dispose]`（同期）または`[Symbol.asyncDispose]`（非同期）を定義する。  
2. `using`構文を用いてオブジェクトを初期化する。  
3. スコープを抜けるタイミングで定義したフックメソッドが自動的に呼び出される。  
  
**公式コード例**  
```ts
// test1.ts
function loggy(id: string): Disposable {
    console.log(`Creating ${id}`);
    return {
        [Symbol.dispose]() {
            console.log(`Disposing ${id}`);
        }
    }
}
function func() {
    using a = loggy("a");
    using b = loggy("b");
    {
        using c = loggy("c");
        using d = loggy("d");
    }
    using e = loggy("e");
    return;
}
func();

// フックメソッド後入れ先出しの順で呼び出される
// Creating a
// Creating b
// Creating c
// Creating d
// Disposing d
// Disposing c
// Creating e
// Disposing e
// Disposing b
// Disposing a
```  
  
https://codesandbox.io/p/devbox/test-using-declaration-jsthks  
  
## 導入手順  
  
- typescriptバージョン5.2以上が必要です  
- **tsconfig.json**の`target`を`es2022`以上にする必要があります。  
  
```json
// tsconfig.json
{
    "compilerOptions": {
        "target": "es2022",
        "lib": ["es2022", "esnext.disposable", "dom"]
    }
}
```  
  
# 使用例  
  
ファイル操作とデータベース接続での例を記載します。  
  
## ファイル操作での利用  
  
#### `using`を使わない場合  
  
`finally`ブロックを使ってクローズ処理を記述します。  
  
```typescript
const fs = require('fs');

const openFile = (path: string) => {
    const file = fs.openSync(path, 'r');
    return {
        file,
        close() {
            fs.closeSync(file);
            console.log(`Closing file: ${path}`);
        }
    };
};

const fileHandle = openFile('example.txt');
try {
    const buffer = Buffer.alloc(1024);
    const bytesRead = fs.readSync(fileHandle.file, buffer, 0, buffer.length, 0);
    console.log("File content:", buffer.toString('utf8', 0, bytesRead));
} finally {
    fileHandle.close();
}
```  
  
#### `using`を使う場合  
  
`[Symbol.dispose]`内にクローズ処理を記述します。  
  
```ts
const fs = require('fs');

const openFile = (path: string) => ({
    file: fs.openSync(path, 'r'),
    [Symbol.dispose]() {
        console.log(`Closing file: ${path}`);
        fs.closeSync(this.file); // ファイルを閉じる
    }
});

(() => {
    using fileHandle = openFile('example.txt');
    const buffer = Buffer.alloc(1024);
    const bytesRead = fs.readSync(fileHandle.file, buffer, 0, buffer.length, 0);
    console.log("File content:", buffer.toString('utf8', 0, bytesRead));
})();
// => File content: (ファイルの中身)
// => Closing file: example.txt
```  
  
## データベース接続での利用  
  
### usingを使わない場合  
  
`finally`ブロックを使ってデータベース接続を破棄します。  
  
```ts
import { DataSource } from 'typeorm';
import { User } from './entity/User';

const createDataSource = async () => {
    const dataSource = new DataSource({
        type: 'postgres',
        host: 'localhost',
        port: 5432,
        username: 'user',
        password: 'password',
        database: 'exampledb',
        entities: [User],
        synchronize: true,
    });
    await dataSource.initialize();
    return {
        dataSource,
        async close() {
            await dataSource.destroy();
            console.log("Database connection closed manually.");
        }
    };
};

(async () => {
    const db = await createDataSource();
    try {
        const userRepository = db.dataSource.getRepository(User);
        const users = await userRepository.find();
        console.log("Users:", users);
    } finally {
        // 明示的にデータベース接続を閉じる
        await db.close();
    }
})();
```  
  
#### `using`を使った場合  
  
`[Symbol.asyncDispose]`内にデータベース接続の破棄処理を記述します。  
  
非同期処理を行う際は`[Symbol.asyncDispose]`と`await using`を使います。  
  
  
```ts
import { DataSource } from 'typeorm';

const createDataSource = async () => {
    const dataSource = new DataSource({
        type: 'postgres',
        host: 'localhost',
        port: 5432,
        username: 'user',
        password: 'password',
        database: 'exampledb',
    });
    await dataSource.initialize();
    return {
        dataSource,
        async [Symbol.asyncDispose]() {
            console.log("Closing database connection...");
            await dataSource.destroy();
        }
    };
};

(async () => {
    await using db = await createDataSource();
    const users = await db.dataSource.query('SELECT * FROM users');
    console.log(users);
})();
// => [ユーザーデータ]
// => Closing database connection...
```  
  
# その他特筆事項  
  
公式ドキュメントを見ていて気になったところについて記述します  
  
## [Symbol.dispose]内でのエラーの扱いについて  
  
「[Symbol.dispose]内のエラー」と「スコープ内のエラー」をどちらも追跡するためError型に`SuppressedError`型の`suppressed`が追加されています。  
  
`e.suppressed`には「スコープ内で最後に投げらえたエラー」、`e.error`には「スコープ終了後、最も最近スローされたエラー」を保持します。  
  
下記コードだと、`e.suppressed`にはErrorA、`e.error`にはErrorBが入ります。  
  
**公式コード**  
  
```ts
class ErrorA extends Error {
    name = "ErrorA";
}
class ErrorB extends Error {
    name = "ErrorB";
}
function throwy(id: string) {
    return {
        [Symbol.dispose]() {
            throw new ErrorA(`Error from ${id}`);
        }
    };
}
function func() {
    using a = throwy("a");
    throw new ErrorB("oops!")
}
try {
    func();
}
catch (e: any) {
    console.log(e.name); // SuppressedError
    console.log(e.message); // An error was suppressed during disposal.
    console.log(e.error.name); // ErrorA
    console.log(e.error.message); // Error from a
    console.log(e.suppressed.name); // ErrorB
    console.log(e.suppressed.message); // oops!
}
```  
  
https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-2.html  
  
## 複数リソースのクリーンアップ処理を行いたいとき  
  
通常、using構文と[Symbol.dispose]を使用すれば、スコープ終了時に自動で1つのリソース解放処理を実行できますが、複数のリソース解放処理をまとめて管理することはできません。  
  
複数のリソースをまとめて管理し、順序正しく解放したい場合には、`DisposableStack`(同期メソッド用)、`AsyncDisposableStack`(非同期メソッド用)の`defer`メソッドを使います。  
  
`defer`メソッドは任意の関数をスタックに追加し、スコープ終了時に後入れ先出しの順序でそれらの処理を実行します。  
  
**同期処理**  
  
```ts
import * as fs from "fs";

async function manageMultipleResources() {
    const file1 = fs.openSync("file1.txt", "w");
    const file2 = fs.openSync("file2.txt", "w");

    using cleanup = new DisposableStack();

    // deferで複数のリソース解放処理を登録
    cleanup.defer(() => {
        console.log("Closing file1...");
        fs.closeSync(file1);
    });
    cleanup.defer(() => {
        console.log("Closing file2...");
        fs.closeSync(file2);
    });

    console.log("Using resources...");
    // ファイル操作...
}

manageMultipleResources();

/*
出力例:
Using resources...
Closing file2...
Closing file1...
*/
```  
  
**非同期処理**  
  
```ts
import * as fsPromises from "fs/promises";

async function manageAsyncResources() {
    const tempFile = "./temp_async_file.txt";
    const fileHandle = await fsPromises.open(tempFile, "w+");

    await using cleanup = new AsyncDisposableStack();

    // 非同期のクリーンアップ処理を登録
    cleanup.defer(async () => {
        console.log("Closing file handle...");
        await fileHandle.close();
    });
    cleanup.defer(async () => {
        console.log("Removing temporary file...");
        await fsPromises.unlink(tempFile);
    });

    console.log("Using the temporary file...");
    // 非同期のファイル操作...
}

await manageAsyncResources();

/*
出力例:
Using the temporary file...
Removing temporary file...
Closing file handle...
*/

```  
  
# まとめ  
  
- **`using`構文**  
  - TypeScript 5.2で導入されたリソース管理のための構文。  
  - スコープ終了時に`[Symbol.dispose]`や`[Symbol.asyncDispose]`を呼び出してリソースを解放する。  
- **`[Symbol.dispose]`の動作**  
  - スコープ終了後に1回だけ実行される。  
  - リソース解放中にエラーが発生すると、そのエラーはスコープ内のエラーと共に追跡される。  
- **`SuppressedError`**  
  - スコープ内のエラーと`[Symbol.dispose]`内のエラーを両方追跡するために導入された`Error`のサブタイプ。  
  - `e.suppressed`: スコープ内で最後にスローされたエラー。  
  - `e.error`: スコープ終了後にスローされた最も最近のエラー。  
- **`DisposableStack`と`AsyncDisposableStack`**  
  - 複数のリソース解放処理を管理するためのスタック型クラス。  
  - `defer`を使って任意の数のクリーンアップ処理を登録可能。  
  - クリーンアップ処理は後入れ先出しの順序で実行される。  
