# 概要  
  
- 型定義するときに`type`と`interface`二つの方法があるが、どちらが良いか迷ったので調べた  
- インターフェイスを使うと、ビルド成果物の中の`d.tsファイル`の容量が小さくなるらしい  
  
## Interface（インターフェース）  
  
```typescript
interface User {
    name: string;
}

interface User {
    age: number;
}

// User型は自動的に { name: string; age: number; } にマージされる
```  
  
- **拡張可能（拡張性）**: `interface`は拡張可能です。これは、同じ名前の`interface`を複数回宣言すると、それらが自動的にマージされることを意味します  
- **新しい型の作成**: オブジェクトや関数の型を定義するために使用されます  
- **宣言のマージ**: 同じ名前のインターフェースが存在する場合、それらの宣言は一つのインターフェースにマージされます  
  
  
## Type（型エイリアス）  
  
```typescript
type User = {
    name: string;
    age: number;
};

type Employee = User & {
    company: string;
};
```  
  
- **非拡張可能**: `type`は拡張できません。同じ名前で複数の`type`を宣言することはできません  
- **ユニオン型や交差型などの複雑な型の作成**: `type`はプリミティブ、ユニオン、交差、タプルなど、任意の複雑な型の作成に使用できます  
- **型の再利用**: 既存の型を組み合わせて新しい型を作ることができます  
  
  
## typeとinterfaceの違いまとめ  
  
- **拡張性**: `interface`は拡張可能であり、同じ名前のインターフェースが自動的にマージされますが、`type`は拡張できません  
- **複雑な型の作成**: `type`はより複雑な型の表現（ユニオン型、交差型、タプルなど）に適しています  
- **使用の推奨**: 一般的に、オブジェクトの型を定義する場合は`interface`が推奨されますが、より複雑な型の操作が必要な場合は`type`が適しています  
  
https://typescriptbook.jp/reference/object-oriented/interface/interface-vs-type-alias  
  
# 本題  
  
## Googleさん的にはインターフェイスを推奨しているらしい  
  
> TypeScript は、型式 に名前を付けるための型エイリアスをサポートしています 。これは、プリミティブ、ユニオン、タプル、およびその他のタイプに名前を付けるために使用できます。  
> ただし、オブジェクトの型を宣言する場合は、オブジェクト リテラル式の型エイリアスの代わりにインターフェイスを使用します。  
  
https://google.github.io/styleguide/tsguide.html#interfaces-vs-type-aliases  
  
## インターフェイスを使うと、d.tsファイルの容量が小さくなるらしい  
  
> 昨日、コードを 1 行変更することで TypeScript 宣言を 700KB から 7KB に縮小しました 🔥  
  
https://ncjamieson.com/prefer-interfaces/  
  
## つまりどういうこと  
  
`type`エイリアスを使用して複雑な型を定義すると、TypeScriptのコンパイラはこれをインラインで展開してしまう可能性があります。  
これは、出力される`.d.ts`ファイルのサイズを大きくすることがあります。  
  
```typescript
type Point = {
  x: number;
  y: number;
};

type Circle = {
  center: Point;
  radius: number;
};
```  
  
上記の例で、`Circle`型は`Point`型を使用しています。  
コンパイラがこれをインラインで展開すると、`Point`型の定義が`Circle`型に直接含まれる形になります。  
  
## interfaceの例  
  
`interface`を使用すると、TypeScriptのコンパイラは名前での参照を維持し、`.d.ts`ファイルがコンパクトに保たれる傾向があります。  
  
```typescript
interface Point {
  x: number;
  y: number;
}

interface Circle {
  center: Point;
  radius: number;
}
```  
  
この場合、`Circle`インターフェースは`Point`インターフェースを名前で参照します。  
このため、`.d.ts`ファイルにおいては`Point`の定義が直接`Circle`に組み込まれることはありません。  
  
# まとめ  
  
- 型定義には基本的にinterfaceを使おう  
- 複雑な型（ユニオン型、交差型、タプルなど）が必要なときはtypeを使うときもある  
