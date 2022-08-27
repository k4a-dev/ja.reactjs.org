---
title: Reactを考える
---

<Intro>

Reactは、あなたが見るデザイン、作るアプリについての考え方を変えることができます。以前は森を見ていたかもしれませんが、Reactで作業した後は個々の木々を認識することができます。Reactは、デザインシステムとUIのstate(状態)で考えることを容易にしてくれます。このチュートリアルでは、Reactで検索可能な商品データテーブルを構築するための思考プロセスを案内します。

</Intro>

## モックアップから開始する {/*start-with-the-mockup*/}

すでにJSON APIとデザイナーからのモックアップがあると想像してください。

JSON APIは、次のようなデータを返します:

```json
[
  { category: "Fruits", price: "$1", stocked: true, name: "Apple" },
  { category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit" },
  { category: "Fruits", price: "$2", stocked: false, name: "Passionfruit" },
  { category: "Vegetables", price: "$2", stocked: true, name: "Spinach" },
  { category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin" },
  { category: "Vegetables", price: "$1", stocked: true, name: "Peas" }
]
```

モックアップは次のように構成されます:

<img src="/images/docs/s_thinking-in-react_ui.png" width="300" style={{margin: '0 auto'}} />

ReactでUIを実装するには、通常、同じ5つのステップを踏むことになります。

## Step 1: UIをコンポーネント階層に分解する {/*step-1-break-the-ui-into-a-component-hierarchy*/}

まず、モックアップのすべてのコンポーネントとサブコンポーネントの周りにボックスを描き、それらに名前を付けます。デザイナーと一緒に仕事をしている場合、すでに彼らのデザインツールでコンポーネントに名前を付けているかもしれません。デザイナーに確認してみてください。

あなたの経歴に応じて、デザインをさまざまな方法でコンポーネントに分割することを考えることができます。

* **プログラミング**--新しい関数やオブジェクトを作成すべきかどうかを決定するための同じテクニックを使用します。そのようなテクニックの1つが[単一責任原則](https://en.wikipedia.org/wiki/Single_responsibility_principle)です。つまり、コンポーネントは理想的には1つのことだけを行うべきであり、もし大きくなってしまったらより小さなサブコンポーネントに分解されるべきです。
* **CSS**--クラスセレクタを作成するものを考えてみてください(ただし、コンポーネントの粒度はもう少し荒くなります[^1])。
* **デザイン**--デザインのレイヤーをどのように構成するかを考えてみてください。

[^1]:原文に"components are a bit less granular"と記されているため「コンポーネントのほうが粒度が低い」が正しいが、「低い」という言葉は細分化のレベルとしてどちらにも捉えられてしまうため「粗い」という訳を使用。

JSONがうまく構造化されていれば、UIのコンポーネント構造に自然にマッピングされることが多いでしょう。これは、UIとデータモデルが同じ情報アーキテクチャ、つまり同じ形をしていることが多いからです。UIをコンポーネントに分割し、各コンポーネントがデータモデルの1つのピースに対応するようにします。

この画面には5つのコンポーネントがあります:

<FullWidth>

<CodeDiagram flip>

<img src="/images/docs/s_thinking-in-react_ui_outline.png" width="500" style={{margin: '0 auto'}} />

1. `FilterableProductTable` (灰色) は、アプリ全体を包括します。
2. `SearchBar` (青色)は、 ユーザーからの入力を受け取ります。
3. `ProductTable` (紫色) は、ユーザーの入力に従ってリストを表示し、フィルタリングします。
4. `ProductCategoryRow` (緑色)は、 各カテゴリの見出しを表示します。
5. `ProductRow`	(黄色) は、各製品に対応する行を表示します。

</CodeDiagram>

</FullWidth>

`ProductTable` (紫色) を見ると、テーブルのヘッダー（"Name" と "Price" ラベルを含む）が独立したコンポーネントではないことがわかるでしょう。これは好みの問題で、どちらを選んでもかまいません。この例では、`ProductTable`のリスト内に表示されているため、`ProductTable`の一部となっています。しかし、このヘッダーが複雑になってきた場合（例えば、ソートを追加した場合）、これを独自の `ProductTableHeader` コンポーネントにすることは理にかなっています。

モックアップのコンポーネントを定めたので、次はそれらを階層構造に並べます。モックアップ内の他のコンポーネントの中にあるコンポーネントは、 階層の中では子として表示されるはずです:
* `FilterableProductTable`
    * `SearchBar`
    * `ProductTable`
        * `ProductCategoryRow`
        * `ProductRow`

## Step 2: Reactで静的なサイトを構築する {/*step-2-build-a-static-version-in-react*/}

コンポーネント階層ができたので、次はアプリを実装しましょう。最も簡単な方法は、データモデルからUIをレンダリングするバージョンを作成し、インタラクティブ機能を追加しないことです。静的なバージョンを最初に構築し、その後、インタラクティブ機能を別途追加する方が簡単な場合が多いです。静的なバージョンを構築するには、多くのタイピングが必要ですが、インタラクティブ性を追加するには、多くの思考と多くのタイピングは必要ありません。


データモデルをレンダリングする静的バージョンのアプリを構築するには、他のコンポーネントを再利用し、[props(プロパティ)] (/learn/passing-props-to-a-component) を使用してデータを渡す [コンポーネント] (/learn/your-first-component) を構築することになるでしょう。propsとは、親から子へデータを渡す方法です。(もしあなたが[state(状態)](/learn/state-a-components-memory)の概念に精通しているなら、この静的バージョンを構築するためにstateを全く使用しないでください。stateはインタラクティブなもの、つまり時間と共に変化するデータのためにのみ予約されています。これはアプリの静的バージョンなので、必要ないのです)。

「トップダウン」と呼ばれる、階層の上位にあるコンポーネント(`FilterableProductTable`など)から構築する方法と、「ボトムアップ」と呼ばれる、下位のコンポーネント(`ProductRow`など)から作業する方法があります。単純な例では、通常トップダウンの方が簡単ですし、大規模なプロジェクトではボトムアップの方が簡単です。

<Sandpack>

```jsx App.js
function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar() {
  return (
    <form>
      <input type="text" placeholder="Search..." />
      <label>
        <input type="checkbox" />
        {' '}
        Only show products in stock
      </label>
    </form>
  );
}

function FilterableProductTable({ products }) {
  return (
    <div>
      <SearchBar />
      <ProductTable products={products} />
    </div>
  );
}

const PRODUCTS = [
  {category: "Fruits", price: "$1", stocked: true, name: "Apple"},
  {category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit"},
  {category: "Fruits", price: "$2", stocked: false, name: "Passionfruit"},
  {category: "Vegetables", price: "$2", stocked: true, name: "Spinach"},
  {category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin"},
  {category: "Vegetables", price: "$1", stocked: true, name: "Peas"}
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}
```

```css
body {
  padding: 5px
}
label {
  display: block;
  margin-top: 5px;
  margin-bottom: 5px;
}
th {
  padding-top: 10px;
}
td {
  padding: 2px;
  padding-right: 40px;
}
```

</Sandpack>

(このコードが怖く見えるなら、まず [クイックスタート](/learn/) を見てください!)

コンポーネントを構築すると、データモデルをレンダリングする再利用可能なコンポーネントのライブラリができあがります。これは静的なアプリなので、コンポーネントはJSXを返すだけです。階層の一番上にあるコンポーネント(`FilterableProductTable`)は、データモデルをpropsとして受け取ります。これは、トップレベルのコンポーネントからツリーの一番下にあるものへとデータが流れていくため_one-way data flow_と呼ばれます。

<Gotcha>

この時点では、state値を使用する必要はありません。それは次のステップで使用します!

</Gotcha>

## Step 3: UIのstateを最小かつ完全に表現するものを探す {/*step-3-find-the-minimal-but-complete-representation-of-ui-state*/}

UIをインタラクティブにするためには、ユーザが基礎となるデータモデルを変更できるようにする必要があります。このために*state(状態)*を使用します。

stateとは、アプリケーションが記憶しておく必要のある、最小限の変更データのセットだと考えてください。stateを構造化するための最も重要な原則は、[DRY（Don't Repeat Yourself）](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)を維持することです。アプリケーションが必要とするstateの絶対的な最小表現を見つけ出し、それ以外のすべてを必要に応じて計算します。例えば、買い物リストを作成する場合、stateにアイテムを配列として格納できます。もし、リストに含まれる商品の数を表示したい場合は、商品の数を別のstate値として格納するのではなく、配列の長さを読み取ります。

さて、このサンプルアプリケーションのデータの断片をすべて考えてみましょう:

1. 元の商品リスト
2. ユーザーが入力した検索文字列
3. チェックボックスの値
4. フィルタリングされた製品リスト

これらのうち、どれがstateでしょうか？そうでないものを特定してください。

* 時間の経過とともに**変化しない**か？もしそうなら、それはstateではありません。
* 親からprops経由で渡されたものですか？もしそうなら、それはstateではありません。
* コンポーネント内の既存のstateやpropsに基づいて、それを計算することができますか？もしそうなら、それは *間違いなく* stateではありません！

残っているのは、おそらくstateです。

もう一度、1つずつ見ていきましょう:

1. 元の商品リストはpropsとして渡されたものなので、stateではありません。
2. 検索テキストは時間とともに変化し、何からも計算できないので、stateであると思われます。
3. チェックボックスの値は、時間の経過とともに変化し、何からも計算できないので、stateであるように思われます。
4. フィルタリングされた商品リストは、元の商品リストを取り出し、検索テキストとチェックボックスの値に従ってフィルタリングすることで**計算できるのでstateとは言えません**。

つまり、検索テキストとチェックボックスの値だけがstateです! うまく導き出せました!

<DeepDive title="props vs state">。

Reactの「モデル」データには、propsとstateという2つのタイプがあります。この2つは非常に異なっています。

* [**props**は、関数に渡す引数のようなもの](/learn/passing-props-to-a-component)です。親コンポーネントが子コンポーネントにデータを渡したり、見た目をカスタマイズしたりすることができます。例えば、 `Form` は `color` propsを `Button` に渡すことができます。
* [**state** はコンポーネントのメモリのようなもの](/learn/state-a-components-memory) です。これは、コンポーネントがいくつかの情報を記録し、インタラクションに応じてそれを変更することを可能にします。例えば、`Button`は `isHovered` stateを記録しているかもしれません。

propsとstateは異なりますが、一緒に動作します。親コンポーネントはしばしば、ある情報を（変更できるように）stateとして保持し、子コンポーネントにpropsとして*渡す*ことがあります。最初に読んだときに、この違いがまだ曖昧に感じられても大丈夫です。本当に定着させるには、少し練習が必要です。

</DeepDive>

## Step 4: stateが存在すべき場所を特定する {/*step-4-identify-where-your-state-should-live*/}

アプリの最小限のstateデータを特定した後、どのコンポーネントがこのstateの変更に責任を持つか、またはstateを*所有*するかを特定する必要があります。Reactは一方通行のデータフローを使用し、親コンポーネントから子コンポーネントへ、コンポーネント階層を下ってデータを渡します。どのコンポーネントがどのstateを所有すべきかは、すぐにはわからないかもしれません。この概念に慣れていない場合は難しいかもしれませんが、次のステップに従えば解決できます！

アプリケーション内の各stateについて

1. そのstateに基づいて何かをレンダリングする *すべての* コンポーネントを特定する。
2. それらのコンポーネントの最も近い共通の親コンポーネント（階層構造ですべてのコンポーネントの上にあるコンポーネント）を見つける。
3. stateをどこに置くかを決定する。
    1. 多くの場合、stateを共通の親コンポーネントに直接置くことができます。
    2. stateを共通の親の上にあるコンポーネントに入れることもできます。
    3. ステートを所有する意味のあるコンポーネントが見つからない場合は、 ステートを保持するためだけの新しいコンポーネントを作成し、 階層のどこかで共通の親コンポーネントの上に追加します。

前のステップで、このアプリケーションには、検索入力テキストとチェックボックスの値という2つのstateのピースがあることがわかりました。この例では、これらは常に一緒に表示されるので、1つのstateとして考える方が簡単です。

では、このstateに対する戦略を実行してみましょう。

1. **stateを使用するコンポーネントを特定する:** 
    * `ProductTable` は、そのstate（検索テキストとチェックボックスの値）に基づいて、製品リストをフィルタリングする必要があります。
    * `SearchBar` は、そのstate（検索テキストとチェックボックスの値）を表示する必要があります。
2.  **共通の親コンポーネントを探す:** 両コンポーネントが共有する最初の親コンポーネントは `FilterableProductTable` です。
3. **stateをどこに置くか決定する** 。フィルタのテキストとチェックされたstateの値は `FilterableProductTable` に保存されます。

そのため、stateの値は `FilterableProductTable` に格納されます。

[`useState()` フック](/apis/usestate) を使用して、コンポーネントにstateを追加します。フックを使うと、コンポーネントの[レンダーサイクル](/learn/render-and-commit)に "接続する (hook into)"することができます。`FilterableProductTable` のトップに2つのstate変数を追加し、アプリケーションの初期stateを指定します:

```js
function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);  
```

Then, pass `filterText` and `inStockOnly` to `ProductTable` and `SearchBar` as props:

```js
<div>
  <SearchBar 
    filterText={filterText} 
    inStockOnly={inStockOnly} />
  <ProductTable 
    products={products}
    filterText={filterText}
    inStockOnly={inStockOnly} />
</div>
```

アプリケーションがどのように動作するかを確認してみましょう。以下のサンドボックスのコードで、`filterText`の初期値を `useState('')` から `useState('fruit')` に編集してください。検索入力のテキストとテーブルの両方が更新されるのがわかります:
<Sandpack>

```jsx App.js
import { useState } from 'react';

function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar 
        filterText={filterText} 
        inStockOnly={inStockOnly} />
      <ProductTable 
        products={products}
        filterText={filterText}
        inStockOnly={inStockOnly} />
    </div>
  );
}

function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products, filterText, inStockOnly }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (
      product.name.toLowerCase().indexOf(
        filterText.toLowerCase()
      ) === -1
    ) {
      return;
    }
    if (inStockOnly && !product.stocked) {
      return;
    }
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar({ filterText, inStockOnly }) {
  return (
    <form>
      <input 
        type="text" 
        value={filterText} 
        placeholder="Search..."/>
      <label>
        <input 
          type="checkbox" 
          checked={inStockOnly} />
        {' '}
        Only show products in stock
      </label>
    </form>
  );
}

const PRODUCTS = [
  {category: "Fruits", price: "$1", stocked: true, name: "Apple"},
  {category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit"},
  {category: "Fruits", price: "$2", stocked: false, name: "Passionfruit"},
  {category: "Vegetables", price: "$2", stocked: true, name: "Spinach"},
  {category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin"},
  {category: "Vegetables", price: "$1", stocked: true, name: "Peas"}
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}
```

```css
body {
  padding: 5px
}
label {
  display: block;
  margin-top: 5px;
  margin-bottom: 5px;
}
th {
  padding-top: 5px;
}
td {
  padding: 2px;
}
```

</Sandpack>

上記のサンドボックスでは、`ProductTable` と `SearchBar` が `filterText` と `inStockOnly` propsを読み込んでテーブルと入力とチェックボックスをレンダリングしています。例えば、`SearchBar` が入力の値をどのように入力するかは以下の通りです:

```js {1,6}
function SearchBar({ filterText, inStockOnly }) {
  return (
    <form>
      <input 
        type="text" 
        value={filterText} 
        placeholder="Search..."/>
```


Reactがどのようにステートを使用し、それを使ってどのようにアプリを構成するかについて深く掘り下げるには、[Stateの管理](/learn/managing-state)を参照してください。

## Step 5: 逆方向のデータフローを追加する {/*step-5-add-inverse-data-flow*/}

現在、あなたのアプリはpropsとstateが階層を下って流れるように正しくレンダリングされています。しかし、ユーザーの入力に応じてstateを変更するには、データの逆流をサポートする必要があります。階層の深いところにあるフォームコンポーネントは、`FilterableProductTable`のstateを更新する必要があります。

Reactはこのデータフローを明示的にしますが、双方向のデータバインディングよりも少し多くのタイピングを必要とします。上の例でタイプしたり、ボックスにチェックを入れようとすると、Reactが入力を無視するのがわかると思います。これは意図的なものです。`<input value={filterText} />`と記述することで、`input` の `value` propsが常に `FilterableProductTable` から渡された `filterText` stateと等しくなるように設定されています。`filterText` stateが設定されることがないので、入力も変更されません。

ユーザーがフォームの入力を変更するたびに、その変更を反映してstateが更新されるようにしたいと思います。このstateは `FilterableProductTable` が所有しているので、 `setFilterText` と `setInStockOnly` を呼び出すことができます。`SearchBar` に `FilterableProductTable` のstateを更新させるには、これらの関数を `SearchBar` に渡す必要があります。

```js {2,3,10,11}
function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar 
        filterText={filterText} 
        inStockOnly={inStockOnly}
        onFilterTextChange={setFilterText}
        onInStockOnlyChange={setInStockOnly} />
```

`SearchBar` の内部では、`onChange` イベントハンドラを追加して、そこから親のstateを設定することになります。

```js {5}
<input 
  type="text" 
  value={filterText} 
  placeholder="Search..." 
  onChange={(e) => onFilterTextChange(e.target.value)} />
```

Now the application fully works!

<Sandpack>

```jsx App.js
import { useState } from 'react';

function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar 
        filterText={filterText} 
        inStockOnly={inStockOnly} 
        onFilterTextChange={setFilterText} 
        onInStockOnlyChange={setInStockOnly} />
      <ProductTable 
        products={products} 
        filterText={filterText}
        inStockOnly={inStockOnly} />
    </div>
  );
}

function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products, filterText, inStockOnly }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (
      product.name.toLowerCase().indexOf(
        filterText.toLowerCase()
      ) === -1
    ) {
      return;
    }
    if (inStockOnly && !product.stocked) {
      return;
    }
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar({
  filterText,
  inStockOnly,
  onFilterTextChange,
  onInStockOnlyChange
}) {
  return (
    <form>
      <input 
        type="text" 
        value={filterText} placeholder="Search..." 
        onChange={(e) => onFilterTextChange(e.target.value)} />
      <label>
        <input 
          type="checkbox" 
          checked={inStockOnly} 
          onChange={(e) => onInStockOnlyChange(e.target.checked)} />
        {' '}
        Only show products in stock
      </label>
    </form>
  );
}

const PRODUCTS = [
  {category: "Fruits", price: "$1", stocked: true, name: "Apple"},
  {category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit"},
  {category: "Fruits", price: "$2", stocked: false, name: "Passionfruit"},
  {category: "Vegetables", price: "$2", stocked: true, name: "Spinach"},
  {category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin"},
  {category: "Vegetables", price: "$1", stocked: true, name: "Peas"}
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}
```

```css
body {
  padding: 5px
}
label {
  display: block;
  margin-top: 5px;
  margin-bottom: 5px;
}
th {
  padding: 4px;
}
td {
  padding: 2px;
}
```

</Sandpack>

イベントの処理とstateの更新については、[インタラクティブ機能の追加](/learn/adding-interactivity)のセクションですべて学ぶことができます。

## 次に学ぶこと {/*where-to-go-from-here*/}

今回は、Reactでコンポーネントやアプリケーションを構築する際の考え方について、ごく簡単に紹介しました。今すぐ[Reactプロジェクトを開始する](/learn/installation)か、このチュートリアルで使用した[すべての構文について深く掘り下げる](/learn/describing-the-ui)ことができます。