---
title: クイックスタート
---

<Intro>

React ドキュメントへようこそ！ このページでは、Reactの概念の中で日常的に使う8割の内容を紹介します。

</Intro>

<YouWillLearn>

- コンポーネントの作成とネストの方法
- マークアップとスタイルの追加方法
- データの表示方法
- リストの描画方法と描画の条件付け方法
- イベントへの応答と画面更新の方法
- コンポーネント間のデータ共有方法

</YouWillLearn>

## コンポーネントの作成とネストの方法 {/*components*/}

React アプリは複数の*コンポーネント*でできています。コンポーネントは、それぞれが独自のロジックと外観を持つ UI(ユーザーインターフェース)の一部品です。コンポーネントは、ボタンのような小さなものから、ページ全体のような大きなものまであります。

React コンポーネントは、マークアップを返す JavaScript の関数です:

```js
function MyButton() {
  return (
    <button>I'm a button</button>
  );
}
```

`MyButton` コンポーネントを宣言したので、別のコンポーネントにネストすることができます:

```js {5}
export default function MyApp() {
  return (
    <div>
      <h1>Welcome to my app</h1>
      <MyButton />
    </div>
  );
}
```

`<MyButton />` が大文字で始まっていることに注意してください。これは React コンポーネントであることを示します。React コンポーネントの名前は常に大文字で始まり、HTML タグは小文字で始まる必要があります。

結果を見てみましょう:

<Sandpack>

```js
function MyButton() {
  return (
    <button>
      I'm a button
    </button>
  );
}

export default function MyApp() {
  return (
    <div>
      <h1>Welcome to my app</h1>
      <MyButton />
    </div>
  );
}
```

</Sandpack>

`export default` キーワードは、ファイル内のメインコンポーネントを指定します。

もしあなたがもしあなたが JavaScript の構文についてよく知らないなら、 [MDN](https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/export) と [javascript.info](https://javascript.info/import-export) に素晴らしい参考資料があります。

## JSX でマークアップを記述する {/*writing-markup-with-jsx*/}

これまでに記述したマークアップの構文は *JSX* と呼ばれています。これはオプションですが、ほとんどの React プロジェクトではその利便性のために JSX を使用しています。全ての [ローカル開発におすすめのツール](/learn/installation) は JSX をサポートしています。

JSX は HTML よりも厳密です。`<br />`タグのようにタグを閉じる必要があります。また、コンポーネントは複数の JSX タグを返すことはできません。複数のタグは`<div>...</div>`や空の`<>...</>`のような共通の親タグで囲まなければいけません:

```js {3,6}
function AboutPage() {
  return (
    <>
      <h1>About</h1>
      <p>Hello there.<br />How do you do?</p>
    </>
  );
}
```

JSX に移植する HTML が多い場合は、 [オンラインコンバーター](https://transform.tools/html-to-jsx)を利用することができます。

## スタイルの追加 {/*adding-styles*/}

React では、CSS のクラスは`className`で指定します。これは[`class`](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/class) 属性と同等に機能します:

```js
<img className="avatar" />
```

そして、それに対する CSS ルールを別の CSS ファイルに記述します:

```css
/* In your CSS */
.avatar {
  border-radius: 50%;
}
```

React では、CSS ファイルの追加方法は規定されていません。最も単純なケースでは、HTML に [`<link>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/link) タグを追加することになります。ビルドツールやフレームワークを使用している場合は、そのドキュメントを参照してプロジェクトに CSS ファイルを追加する方法を学んでください。

## データの表示 {/*displaying-data*/}

JSX は JavaScript にマークアップを入れることができます。波括弧で囲むと、JavaScript に「エスケープバック（出戻り）」して、コードから変数を埋め込み、それをユーザーに表示することができます。例えば、以下は `user.name` を表示します:

```js {3}
return (
  <h1>
    {user.name}
  </h1>
);
```

JSX の属性から JavaScript にエスケープすることもできますが、引用符の**代わりに**波括弧を使わなければなりません。例えば、`className="avatar"` は CSS のクラスとして `"avatar"` という文字列を渡しますが、`src={user.imageUrl}` は JavaScript の`user.imageUrl` という変数の値を読み取り、その値を `src` 属性として渡します:

```js {3,4}
return (
  <img
    className="avatar"
    src={user.imageUrl}
  />
);
```

JSX の波括弧の中には、[文字列の連結](https://javascript.info/operators#string-concatenation-with-binary)のようなより複雑な式を入れることもできます:

<Sandpack>

```js
const user = {
  name: 'Hedy Lamarr',
  imageUrl: 'https://i.imgur.com/yXOvdOSs.jpg',
  imageSize: 90,
};

export default function Profile() {
  return (
    <>
      <h1>{user.name}</h1>
      <img
        className="avatar"
        src={user.imageUrl}
        alt={'Photo of ' + user.name}
        style={{
          width: user.imageSize,
          height: user.imageSize
        }}
      />
    </>
  );
}
```

```css
.avatar {
  border-radius: 50%;
}

.large {
  border: 4px solid gold;
}
```

</Sandpack>

上記の例の中の `style={{}}` は特別な構文ではなく、JSX の波括弧 `style={ }` の中に通常の `{}` オブジェクトを記述しています。スタイルが JavaScript の変数に依存している場合、`style` 属性を使用することができます。

## 条件付きレンダリング {/*conditional-rendering*/}

React では、条件を記述するための特別な文法はありません。その代わり、通常の JavaScript のコードを書くときと同じテクニックを使います。例えば [`if`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/if...else) 文を使って条件付きで JSX を組み込むことができます:

```js
let content;
if (isLoggedIn) {
  content = <AdminPanel />;
} else {
  content = <LoginForm />;
}
return (
  <div>
    {content}
  </div>
);
```

よりコンパクトなコードを好む場合、 [conditional `?` operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_Operator) が使えます。`if` とは異なり、JSX の内部に記述できます:

```js
<div>
  {isLoggedIn ? (
    <AdminPanel />
  ) : (
    <LoginForm />
  )}
</div>
```

`else`分岐が必要な場合、より短い [logical `&&` syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_AND#short-circuit_evaluation)が使えます:

```js
<div>
  {isLoggedIn && <AdminPanel />}
</div>
```

これらのアプローチは全て、属性を条件付きで指定する場合にも有効です。もしあなたが JavaScript の構文をよく知らないのであれば、常に`if...else`を使うところから始めてみましょう。

## リストのレンダリング {/*rendering-lists*/}

コンポーネントのリストを表示するには、 [`for` loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for) や [array `map()` function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) などの JavaScript の機能に頼ることになります。

例えば、商品の配列があるとします:

```js
const products = [
  { title: 'Cabbage', id: 1 },
  { title: 'Garlic', id: 2 },
  { title: 'Apple', id: 3 },
];
```

コンポーネント内部では、`map()`関数を使用して、商品の配列を`<li>`項目の配列に変換します:

```js
const listItems = products.map(product =>
  <li key={product.id}>
    {product.title}
  </li>
);

return (
  <ul>{listItems}</ul>
);
```

`<li>` が `key` 属性を持っていることに注目してください。リスト内の各項目には、その項目を兄弟間で一意に識別するための文字列または数値を渡す必要があります。通常、キーはデータベース ID のようなデータから取得する必要があります。React は、アイテムの挿入、削除、並べ替えの際に何が起こったかを理解するために、キーを頼りにします:

<Sandpack>

```js
const products = [
  { title: 'Cabbage', isFruit: false, id: 1 },
  { title: 'Garlic', isFruit: false, id: 2 },
  { title: 'Apple', isFruit: true, id: 3 },
];

export default function ShoppingList() {
  const listItems = products.map(product =>
    <li
      key={product.id}
      style={{
        color: product.isFruit ? 'magenta' : 'darkgreen'
      }}
    >
      {product.title}
    </li>
  );

  return (
    <ul>{listItems}</ul>
  );
}
```

</Sandpack>

## イベントへの応答 {/*responding-to-events*/}

コンポーネント内部で*イベントハンドラ*関数を宣言することで、イベントに応答することができます:

```js {2-4,7}
function MyButton() {
  function handleClick() {
    alert('You clicked me!');
  }

  return (
    <button onClick={handleClick}>
      Click me
    </button>
  );
}
```

`onClick={handleClick}` の最後に括弧 `()` が無いことに注意してください！イベントハンドラ関数を*呼び出し*ては行けません。必要なのは*渡す*ことだけです。ユーザーがボタンをクリックすると React はイベントハンドラを呼び出します。

## 画面の更新 {/*updating-the-screen*/}

コンポーネントに何らかの情報を「記憶」させ、それを表示させたいことがよくあります。例えば、ボタンがクリックされた回数をカウントしたい場合などです。これを実現するには、コンポーネントに _state_ を追加します。

まず、[`useState`](/apis/usestate)を React からインポートします:

```js {1,4}
import { useState } from 'react';
```

コンポーネント内部で*state 変数*を宣言できるようになりました:

```js
function MyButton() {
  const [count, setCount] = useState(0);
```

`useState`からは、現在の state (`count`) と、それを更新するための関数 (`setCount`) の 2 つを取得することができます。これらの関数には任意の名前を付けられますが、慣習として`[something, setSomething]`のように名付けられます。

最初にボタンを表示したときは、`useState()`に`0`を渡したので`count`は`0`になります。state を変更したい場合は、`setCount()`を呼び出して新しい値を渡します。このボタンをクリックすると、カウンターが加算されます:

```js {5}
function MyButton() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      Clicked {count} times
    </button>
  );
}
```

React はコンポーネント関数を再度呼び出します。今度は`count`が`1`になります。そして次に`2`となる。という具合です。

同じコンポーネントを複数個レンダリングすると、それぞれが独自の state を所有します。それぞれのボタンを個別にクリックしてみてください:

<Sandpack>

```js
import { useState } from 'react';

function MyButton() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      Clicked {count} times
    </button>
  );
}

export default function MyApp() {
  return (
    <div>
      <h1>Counters that update separately</h1>
      <MyButton />
      <MyButton />
    </div>
  );
}
```

```css
button {
  display: block;
  margin-bottom: 5px;
}
```

</Sandpack>

各ボタンは自分自身の `count` の state を「記憶」しており、他のボタンには影響を与えないことに注意してください。

## フックを使用する {/*using-hooks*/}

`use` で始まる関数は *フック(Hooks)* と呼ばれます。`useState` は React が提供する組み込みのフックです。他の組み込みフックは [React API reference](/apis) で確認できます。また、既存のフックを組み合わせて独自のフックを記述することもできます。

フックは、通常の関数よりも制約を多く持ちます。フックはコンポーネント(または他のフック)の*トップレベル*でしか呼び出すことができません。もし、条件やループの中で`State`を使いたいなら、新しいコンポーネントに切り出してそのトップレベルに記述してください。

## コンポーネント間のデータ共有 {/*sharing-data-between-components*/}

前の例では、それぞれの `MyButton` が独立した `count` を持っていて、それぞれのボタンがクリックされるとクリックされたボタンの `count` だけが変更されました:

<DiagramGroup>

<Diagram name="sharing_data_child" height={367} width={407} alt="MyApp という親と MyButton という子からなる3つのコンポーネントのツリーを示す図です。MyButton コンポーネントは両方ともカウント値 0 を含んでいます。">

初期状態では、それぞれの`MyButton`の`count` state は`0`です

</Diagram>

<Diagram name="sharing_data_child_clicked" height={367} width={407} alt="先程と同じ図で、最初の子 MyButton コンポーネントのカウントがハイライトされ、クリックされてカウント値が 1 に増加したことを示しています。2 番目の MyButton コンポーネントは、まだ値0を含んでいます。" >

1 つ目の`MyButton`は、`count`を`1`に更新します

</Diagram>

</DiagramGroup>

しかし多くの場合、コンポーネントはデータを共有し常に一緒に更新する必要があります。

両方の `MyButton` コンポーネントが同じ `count` を表示し一緒に更新するようにするには、個々のボタンの state を、それらをすべて含む最も近いコンポーネントに「上向きに」移動させる必要があります。

この例では、それは `MyApp` です:

<DiagramGroup>

<Diagram name="sharing_data_parent" height={385} width={410} alt="MyApp という親と MyButton という子からなる3つのコンポーネントのツリーを示す図です。MyApp はカウント値 0 を含み、その値は MyButton コンポーネントの両方に渡され、同じく値 0 を表示します。" >

初期状態では、`MyApp`の`count` state は`0`で両方の子コンポーネントに渡されます

</Diagram>

<Diagram name="sharing_data_parent_clicked" height={385} width={410} alt="先程と同じ図で、親である MyApp コンポーネントのカウントがハイライトされており、クリックされると値が 1 に増加したことを示しています。子である MyButton コンポーネントへのフローもハイライトされ、それぞれの子のカウント値が 1 に設定され、値が受け渡されたことを示しています。" >

クリックすると、`MyApp`は`count` state を`1`に更新し、両方の子コンポーネントにそれを渡します

</Diagram>

</DiagramGroup>

これで、どちらかのボタンをクリックすると`MyApp` の `count` が変化し、`MyButton` のカウントも両方変化するようになります。これをコードで表現すると次のようになります。

まず、`MyButton`から`MyApp`に *state を移動* します：

```js {2,6-10}
function MyButton() {
  // ... we're moving code from here ...
}

export default function MyApp() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <div>
      <h1>Counters that update separately</h1>
      <MyButton />
      <MyButton />
    </div>
  );
}
```

そして `MyApp` から各 `MyButton` に state を共有のクリックハンドラーと共に渡します。これまでに `<img>` の様な組み込みタグで行ったように、JSX の波括弧を使用して `MyButton` へ情報を渡すことができます:

```js {11-12}
export default function MyApp() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <div>
      <h1>Counters that update together</h1>
      <MyButton count={count} onClick={handleClick} />
      <MyButton count={count} onClick={handleClick} />
    </div>
  );
}
```

このように受け渡す情報のことを *props（プロパティ）* と呼びます。ここで、`MyApp` コンポーネントは `count` state と`handleClick`イベントハンドラを所持しており、その両方を props として各ボタンに渡しています。

最後に、`MyButton`を変更して、親コンポーネントから渡された props を読み込むようにします:

```js {1,3}
function MyButton({ count, onClick }) {
  return (
    <button onClick={onClick}>
      Clicked {count} times
    </button>
  );
}
```

ボタンをクリックすると、`onClick` ハンドラが起動します。各ボタンの `onClick` props は `MyApp` 内の `handleClick` 関数に設定されているので、その中のコードが実行されます。このコードは `setCount(count + 1)` を呼び出して、ステート変数 `count` を増加します。新しい`count`の値は各ボタンに props として渡され、すべてのボタンが新しい値を表示するようになります。

これは「state のリフトアップ」と呼ばれます。state を上に上げることで、コンポーネント間で state を共有することができました。


<Sandpack>

```js
import { useState } from 'react';

function MyButton({ count, onClick }) {
  return (
    <button onClick={onClick}>
      Clicked {count} times
    </button>
  );
}

export default function MyApp() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <div>
      <h1>Counters that update together</h1>
      <MyButton count={count} onClick={handleClick} />
      <MyButton count={count} onClick={handleClick} />
    </div>
  );
}
```

```css
button {
  display: block;
  margin-bottom: 5px;
}
```

</Sandpack>

## 次のステップ {/*next-steps*/}

ここまでの学習で、React のコードの書き方の基本はわかったと思います!

[React の考え方](/learn/thinking-in-react)に向かい、実際に React で UI を構築する感覚を確認してみましょう。