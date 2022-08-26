---
title: クイックスタート
---

<Intro>

Reactドキュメントへようこそ！ このページでは、Reactを日常的に使う上で必要な概念を紹介します。この内容はReactの概念の8割に及びます。

</Intro>

<YouWillLearn>

- コンポーネントの作成・ネストの方法
- マークアップとスタイリングの方法
- データの表示方法
- リストの描画方法と描画の条件付け方法
- DOMイベントへの応答と画面更新の方法
- コンポーネント間でのデータ共有方法

</YouWillLearn>

## コンポーネントの作成・ネストの方法 {/*components*/}

Reactアプリは複数の*コンポーネント*で出来ています。コンポーネントは、それぞれが独自のロジックと外観を持つUI(ユーザーインターフェース)の一部品です。コンポーネントは、ボタンのような小さなものから、ページ全体のような大きなものまであります。

Reactコンポーネントは、マークアップを返すJavaScriptの関数です:

```js
function MyButton() {
  return (
    <button>I'm a button</button>
  );
}
```

`MyButton`コンポーネントを宣言したので、別のコンポーネントにネストすることが出来ます:

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

`<MyButton />`が大文字で始まっていることに注意してください。これはReactコンポーネントであること示します。Reactコンポーネントの名前は常に大文字で始まり、HTMLタグは小文字で始まる必要があります。

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

`export default`キーワードは、ファイル内のメインコンポーネントを指定します。

 もしあなたがもしあなたがJavaScriptの構文についてよく知らないなら、 [MDN](https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/export) と [javascript.info](https://javascript.info/import-export) に素晴らしい参考資料があります。

## JSXでマークアップを記述する {/*writing-markup-with-jsx*/}

これまでに記述したマークアップの構文は*JSX*と呼ばれています。これはオプションですが、ほとんどのReactプロジェクトではその利便性のためにJSXを使用しています。全ての [ローカル開発におすすめのツール](/learn/installation) はJSXをサポートしています。

JSXはHTMLよりも厳格です。`<br />`タグのようにタグを閉じる必要があります。また、コンポーネントは複数のJSXタグを返すことは出来ません。複数のタグは`<div>...</div>`や空タグ(フラグメントの短縮記法)`<>...</>`のような共通の親タグで囲まなければいけません:

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

JSXに移植するHTMLが多い場合は、 [オンラインコンバーター](https://transform.tools/html-to-jsx)を利用することが出来ます。

## スタイルの追加 {/*adding-styles*/}

Reactでは、CSSのクラスは`className`で指定します。これは[`class`](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/class) 属性と同等に機能します:

```js
<img className="avatar" />
```

そして、それに対するCSSルールを別のCSSファイルに記述します:

```css
/* In your CSS */
.avatar {
  border-radius: 50%;
}
```

Reactでは、CSSファイルの追加方法は規定されていません。最も単純なケースでは、HTMLに[`<link>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/link)タグを追加することになります。ビルドツールやフレームワークを使用している場合は、そのドキュメントを参照して、プロジェクトにCSSファイルを追加する方法を学んでください。

## データの表示 {/*displaying-data*/}

JSXはJavaScriptにマークアップを入れる構文です。中括弧で囲むと、JavaScriptに「エスケープバック（出戻り）」して、コードから変数を埋め込み、それをユーザーに表示することができます。例えば、これは`user.name`を表示します:

```js {3}
return (
  <h1>
    {user.name}
  </h1>
);
```

JSXの属性からJavaScriptにエスケープすることもできますが、引用符の**代わりに**中括弧を使わなければなりません。例えば、`className="avatar"`は CSS のクラスとして`"avatar"`という文字列を渡しますが、`src={user.imageUrl}`は JavaScript の`user.imageUrl`という変数の値を読み取り、その値を`src`属性として渡します:

```js {3,4}
return (
  <img
    className="avatar"
    src={user.imageUrl}
  />
);
```

JSXの中括弧の中には、[文字列の連結](https://javascript.info/operators#string-concatenation-with-binary)のようなより複雑な式を入れることも出来ます:

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

上記の例では、`style={{}}`は特別な構文ではなく、JSXの中括弧`style={ }`の中に通常の`{}`オブジェクトを記述しています。スタイルがJavaScriptの変数に依存している場合、`style`属性を使用することができます。

## 条件付きレンダリング {/*conditional-rendering*/}

Reactでは、条件を記述するための特別な文法はありません。その代わり、通常のJavaScriptのコードを書くときと同じテクニックを使うことになります。例えば [`if`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/if...else) 文を使って条件付きでJSXを組み込むことが出来ます:

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

よりコンパクトなコードを好む場合、 [conditional `?` operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_Operator)を使うことが出来ます。`if`とは異なり、JSXの内部に記述出来ます:

```js
<div>
  {isLoggedIn ? (
    <AdminPanel />
  ) : (
    <LoginForm />
  )}
</div>
```

`else`分岐が必要な場合、より短い[logical `&&` syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_AND#short-circuit_evaluation)を使用することが出来ます:

```js
<div>
  {isLoggedIn && <AdminPanel />}
</div>
```

これらのアプローチは全て、属性を条件付きで指定する場合にも有効です。もしあなたがJavaScriptの構文になれていないなら、常に`if...else`を使うところから始めることをおすすめします。

## リストのレンダリング {/*rendering-lists*/}

コンポーネントのリストを表示するには、 [`for` loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for) や [array `map()` function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) などのJavaScriptの機能に頼ることになります。

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

`<li>`が`key`属性を持っていることに注目してください。リスト内の各項目には、その項目を兄弟間で一意に識別するための文字列または数値を渡す必要があります。通常、キーはデータベースIDのようなデータから取得する必要があります。React は、アイテムの挿入、削除、並べ替えの際に何が起こったかを理解するために、キーを頼りにします:
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

`onClick={handleClick}`の最後に括弧`()`が無いことに注意してください！イベントハンドラ関数を_呼び出し_ては行けません。必要なのは*渡す*ことだけです。ユーザーがボタンをクリックするとReactはイベントハンドラを呼び出します。

## 画面の更新 {/*updating-the-screen*/}

コンポーネントに何らかの情報を「記憶」させ、それを表示させたいことがよくあります。例えば、ボタンがクリックされた回数をカウントしたい場合などです。これを実現するには、コンポーネントに *state* を追加します。

まず、[`useState`](/apis/usestate)をReactからインポートします:

```js {1,4}
import { useState } from 'react';
```

これで、コンポーネント内部で*state変数*を宣言できるようになりました:

```js
function MyButton() {
  const [count, setCount] = useState(0);
```

`useState`からは、現在の状態 (`count`) と、それを更新するための関数 (`setCount`) の2つを取得することができます。これらの関数には任意の名前を付けられますが、慣習として`[something, setSomething]`のように名付けられます。

最初にボタンを表示したときは、`useState()`に`0`を渡したので`count`は`0`になります。状態を変更したい場合は、`setCount()`を呼び出して、新しい値を渡します。このボタンをクリックすると、カウンターが加算されます:

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

Reactはコンポーネント関数を再度呼び出します。今度は`count`が`1`になります。次に`2`となる。という具合です。

同じコンポーネントを複数個レンダリングすると、それぞれが独自の状態を所有します。それぞれのボタンを個別にクリックしてみてください:

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

各ボタンは自分自身の`count`の状態を「記憶」しており、他のボタンには影響を与えないことに注意してください。

## Hookを使用する {/*using-hooks*/}

`use`で始まる関数は *Hooks* と呼ばれます。`useState`は React が提供する組み込みの Hook です。他の組み込みHookは [React API reference](/apis) で確認できます。また、既存のHookを組み合わせて独自のHookを記述することもできます。

Hookは、通常の関数よりも制約を多く持ちます。Hookはコンポーネント(または他のHook)の*トップレベル*でしか呼び出すことができません。もし、条件やループの中で`State`を使いたいなら、新しいコンポーネントに切り出してそのトップレベルに記述してください。

## コンポーネント間のデータ共有 {/*sharing-data-between-components*/}

前の例では、それぞれの`MyButton`が独立した`count`を持っていて、それぞれのボタンがクリックされると、クリックされたボタンの`count`だけが変更されました:

<DiagramGroup>

<Diagram name="sharing_data_child" height={367} width={407} alt="Diagram showing a tree of three components, one parent labeled MyApp and two children labeled MyButton. Both MyButton components contain a count with value zero.">

初期状態では、それぞれの`MyButton`の`count`状態は`0`です

</Diagram>

<Diagram name="sharing_data_child_clicked" height={367} width={407} alt="The same diagram as the previous, with the count of the first child MyButton component highlighted indicating a click with the count value incremented to one. The second MyButton component still contains value zero." >

1つ目の`MyButton`は、`count`を`1`に更新します。

</Diagram>

</DiagramGroup>

しかし多くの場合、コンポーネントはデータを共有し常に一緒に更新する必要があります。

両方の`MyButton`コンポーネントが同じ`count`を表示し一緒に更新するようにするには、個々のボタンの状態を、それらをすべて含む最も近いコンポーネントに「上向きに」移動させる必要があります。

この例では、それは`MyApp`です:

<DiagramGroup>

<Diagram name="sharing_data_parent" height={385} width={410} alt="Diagram showing a tree of three components, one parent labeled MyApp and two children labeled MyButton. MyApp contains a count value of zero which is passed down to both of the MyButton components, which also show value zero." >

初期状態では、`MyApp`の`count`状態は`0`で両方の子コンポーネントに渡されます

</Diagram>

<Diagram name="sharing_data_parent_clicked" height={385} width={410} alt="The same diagram as the previous, with the count of the parent MyApp component highlighted indicating a click with the value incremented to one. The flow to both of the children MyButton components is also highlighted, and the count value in each child is set to one indicating the value was passed down." >

クリックすると、`MyApp`は`count`の状態を`1`に更新し、両方の子コンポーネントにそれを渡します

</Diagram>

</DiagramGroup>

これで、どちらかのボタンをクリックすると、`MyApp`の`count`カウントが変化し、`MyButton`のカウントも両方変化するようになります。これをコードで表現すると次のようになります。

まず、`MyButton`から`MyApp`に *状態を移動* します。

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

そして`MyApp`から各`MyButton`に、状態を共有のクリックハンドラーと共に渡します。これまでに`<img>`の様な組み込みタグで行ったように、JSXの中括弧を使用して`MyButton`へ情報を渡すことが出来ます:

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

このように受け渡す情報のことを_プロパティ_と呼びます。ここで、`MyApp`コンポーネントは`count`ステートと`handleClick`イベントハンドラを所持しており、その両方をプロパティとして各ボタンに渡しています。

最後に、`MyButton`を変更して、親コンポーネントから渡されたプロパティを読み込むようにします:

```js {1,3}
function MyButton({ count, onClick }) {
  return (
    <button onClick={onClick}>
      Clicked {count} times
    </button>
  );
}
```

ボタンをクリックすると、`onClick`ハンドラが起動します。各ボタンの`onClick`プロパティは`MyApp`内の`handleClick`関数に設定されているので、その中のコードが実行されます。このコードは`setCount(count + 1)`を呼び出して、ステート変数`count`をインクリメントします。新しい`count`の値は各ボタンに prop として渡され、すべてのボタンが新しい値を表示するようになります。

これは「state のリフトアップ」と呼ばれます。状態を上に上げることで、コンポーネント間で状態を共有することが出来ました。


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

ここまでの学習で、Reactのコードの書き方の基本はわかったと思います!

[Reactを考える](/learn/thinking-in-react)に向かい、実際にReactでUIを構築する感覚を確認してみましょう。