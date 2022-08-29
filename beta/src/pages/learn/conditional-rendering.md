---
title: Conditional Rendering
---

<Intro>

コンポーネントでは、異なる条件によって異なるものを表示する必要が出てきます。Reactでは、`if`文、`&&`、`? :`演算子などのJavaScript構文を使ってJSXを条件付きでレンダリングすることができます。


</Intro>

<YouWillLearn>

* 条件によって異なるJSXを返す方法
* JSXの一部を条件付きで含めたり除外する方法
* Reactの一般的な条件構文のショートカット

</YouWillLearn>

## 条件付きでJSXを返す {/*conditionally-returning-jsx*/}

例えば、`PackingList`コンポーネントがいくつかの`Item`をレンダリングしているとします：

<Sandpack>

```js
function Item({ name, isPacked }) {
  return <li className="item">{name}</li>;
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item 
          isPacked={true} 
          name="Space suit" 
        />
        <Item 
          isPacked={true} 
          name="Helmet with a golden leaf" 
        />
        <Item 
          isPacked={false} 
          name="Photo of Tam" 
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

`Item` コンポーネントの中には、 `isPacked` プロパティが `false` ではなく `true` に設定されているものがあることに注意してください。パックされたアイテムにチェックマーク(✔)を付けたい場合、`isPacked={true}`を指定します。

これは以下のような[`if`/`else`文](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/if...else)として書くことができます。

```js
if (isPacked) {
  return <li className="item">{name} ✔</li>;
}
return <li className="item">{name}</li>;
```

`isPacked` プロパティが `true` の場合、このコードは **別のJSXツリー** を返します。この変更により、いくつかの項目は最後にチェックマークが付くようになりました：

<Sandpack>

```js
function Item({ name, isPacked }) {
  if (isPacked) {
    return <li className="item">{name} ✔</li>;
  }
  return <li className="item">{name}</li>;
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item 
          isPacked={true} 
          name="Space suit" 
        />
        <Item 
          isPacked={true} 
          name="Helmet with a golden leaf" 
        />
        <Item 
          isPacked={false} 
          name="Photo of Tam" 
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

どちらの場合でも返されるものを編集して、結果がどう変わるか見てみましょう!

JavaScript の `if` と `return` 文で、分岐したロジックをどのように作成しているかに注目してください。Reactでは、制御フロー（条件のようなもの）はJavaScriptで処理されます。

### `null`を使用して条件によって何も返さない{/*conditionally-returning-nothing-with-null*/}。 {/*nullを使用して条件によって何も返さないconditionally-returning-nothing-with-null*/}

状況によっては、何もレンダリングしたくないことがあります。例えば、パックされたアイテムを全く表示したくないとします。コンポーネントは何かを返さなければなりません。この場合、`null`を返すことができます：

```js
if (isPacked) {
  return null;
}
return <li className="item">{name}</li>;
```

`isPacked` が true の場合、コンポーネントは何も返さず、 `null` を返します。そうでない場合は、レンダリングするためのJSXを返します。

<Sandpack>

```js
function Item({ name, isPacked }) {
  if (isPacked) {
    return null;
  }
  return <li className="item">{name}</li>;
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item 
          isPacked={true} 
          name="Space suit" 
        />
        <Item 
          isPacked={true} 
          name="Helmet with a golden leaf" 
        />
        <Item 
          isPacked={false} 
          name="Photo of Tam" 
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

実際には、コンポーネントから `null` を返すことは一般的ではありません。それよりも、親コンポーネントのJSXに条件付きでコンポーネントを含めたり除外したりすることが多いでしょう。ここでは、その方法を説明します。

## JSXを条件付きで含める {/*conditionally-including-jsx*/}


前の例では、コンポーネントから返されるJSXツリーを制御していました。レンダリング出力に重複があることにすでにお気づきかもしれません。

```js
<li className="item">{name} ✔</li>
```

is very similar to

```js
<li className="item">{name}</li>
```

Both of the conditional branches return `<li className="item">...</li>`:

```js
if (isPacked) {
  return <li className="item">{name} ✔</li>;
}
return <li className="item">{name}</li>;
```

この重複は有害ではありませんが、コードの保守を困難にする可能性があります。もし、 `className` を変更したい場合はどうすればよいでしょうか？その場合、コードの2箇所でそれを行わなければなりません。このような場合、条件付きで小さなJSXを含めることで、コードをより[DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)にすることができます。

### 条件(三項)演算子 (`? :`) {/*conditional-ternary-operator--*/}

JavaScript には条件式を書くためのコンパクトな構文があります。[条件演算子](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_Operator) または "三項演算子" です。

以下の書き方の代わりに：

```js
if (isPacked) {
  return <li className="item">{name} ✔</li>;
}
return <li className="item">{name}</li>;
```

この様に書くことができます：

```js
return (
  <li className="item">
    {isPacked ? name + ' ✔' : name}
  </li>
);
```

この式は *"もし `isPacked`が`true`なら,  (`?`) は `name + ' ✔'`をレンダリングし, そうでなければ (`:`) は `name`. をレンダリングする"*) と読むことが出来ます。

<DeepDive title="Are these two examples fully equivalent?">

もしあなたがオブジェクト指向プログラミングの経験を持っているなら、上の二つの例は微妙に違うと思うかもしれません。なぜなら、どちらかが `<li>` の二つの異なる「インスタンス」を作成するかもしれないからです。しかし、JSX要素は内部状態を保持せず、実際のDOMノードでもないため、「インスタンス」ではありません。JSX要素はあくまで設計図のような軽量の記述なのです。ですから、この2つの例は実際には完全に等価です。[Preserving and Resetting State](/learn/preserving-and-resetting-state) は、この仕組みについて詳しく説明しています。

</DeepDive>

さて、完成した項目のテキストを、`<del>` のように別のHTMLタグで囲んで打ち消すとします。さらに改行や括弧を追加して、それぞれのケースでより多くのJSXをネストしやすくすることができます：

<Sandpack>

```js
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {isPacked ? (
        <del>
          {name + ' ✔'}
        </del>
      ) : (
        name
      )}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item 
          isPacked={true} 
          name="Space suit" 
        />
        <Item 
          isPacked={true} 
          name="Helmet with a golden leaf" 
        />
        <Item 
          isPacked={false} 
          name="Photo of Tam" 
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

このスタイルは単純な条件には有効ですが、適度に使ってください。ネストされた条件マークアップが多すぎてコンポーネントがごちゃごちゃになる場合は、子コンポーネントを抽出してすっきりさせることを検討してください。Reactでは、マークアップはコードの一部なので、変数や関数のようなツールを使って複雑な式を整理することができます。

### 論理 AND 演算子 (`&&`) {/*logical-and-operator-*/}

もうひとつよく見かけるショートカットに [JavaScriptの論理AND演算子(`&&`)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_AND#:~:text=The%20logical%20AND%20(%20%26%26%20)%20operator,it%20returns%20a%20Boolean%20value.)があります. Reactコンポーネントの内部では、ある条件がtrueのときにJSXをレンダリングし、そうでないときは何もレンダリングしない、ということがよく出てきます：

```js
return (
  <li className="item">
    {name} {isPacked && '✔'}
  </li>
);
```

これは *"if `isPacked`, then (`&&`) render the checkmark, otherwise, render nothing "*と読むことができます。

実際にやってみると、こんな感じです：

<Sandpack>

```js
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {name} {isPacked && '✔'}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item 
          isPacked={true} 
          name="Space suit" 
        />
        <Item 
          isPacked={true} 
          name="Helmet with a golden leaf" 
        />
        <Item 
          isPacked={false} 
          name="Photo of Tam" 
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

[JavaScript && 式](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_AND)は、左辺（条件）が `true` なら右辺（この場合、チェックマーク）の値を返す。しかし、条件が `false` であれば、式全体が `false` になります。React は `false` を `null` や `undefined` と同じように JSX ツリーの「穴」とみなし、その場所には何もレンダリングしない。


<Gotcha>

**`&&`. の左側に数字を書かないでください。**

この条件を満たすために、JavaScript は左辺を自動的にブーリアンに変換します。しかし、左辺が `0` であれば、式全体がその値 (`0`) を取得し、React は何もしないのではなく、喜んで `0` をレンダリングします。

例えば、よくある間違いは、`messageCount && <p>New messages</p>` のようなコードを書くことです。これは `messageCount` が `0` のときに何もレンダリングしないと思いがちですが、実際には `0` そのものをレンダリングしているのです!

これを修正するには、左側を `messageCount > 0 && <p>New messages</p>` というブール型にします。

</Gotcha>

### 条件付きでJSXを変数に代入する {/*conditionally-assigning-jsx-to-a-variable*/}

ショートカットがプレーンなコードを書くのに邪魔になったら、`if`文と変数を使ってみてください。let`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let) で定義された変数は再代入が可能なので、まずは表示したいデフォルトの内容、名前を用意しましょう。

```js
let itemContent = name;
```

isPacked` が `true` であれば、 `itemContent` に JSX 式を再割り当てするために `if` ステートメントを使用します

```js
if (isPacked) {
  itemContent = name + " ✔";
}
```

[Curly braces open the "window into JavaScript".](/learn/javascript-in-jsx-with-curly-braces#using-curly-braces-a-window-into-the-javascript-world) 中括弧の付いた変数を、返されたJSXツリーに埋め込み、前に計算した式をJSX内部に入れ子にします。

```js
<li className="item">
  {itemContent}
</li>
```

このスタイルは最も冗長ですが、最も柔軟性があります。以下はその実例です：

<Sandpack>

```js
function Item({ name, isPacked }) {
  let itemContent = name;
  if (isPacked) {
    itemContent = name + " ✔";
  }
  return (
    <li className="item">
      {itemContent}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item 
          isPacked={true} 
          name="Space suit" 
        />
        <Item 
          isPacked={true} 
          name="Helmet with a golden leaf" 
        />
        <Item 
          isPacked={false} 
          name="Photo of Tam" 
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

前回同様、テキストだけでなく、任意のJSXに対しても有効です：

<Sandpack>

```js
function Item({ name, isPacked }) {
  let itemContent = name;
  if (isPacked) {
    itemContent = (
      <del>
        {name + " ✔"}
      </del>
    );
  }
  return (
    <li className="item">
      {itemContent}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item 
          isPacked={true} 
          name="Space suit" 
        />
        <Item 
          isPacked={true} 
          name="Helmet with a golden leaf" 
        />
        <Item 
          isPacked={false} 
          name="Photo of Tam" 
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

JavaScriptに慣れていないと、この多様なスタイルに最初は圧倒されるかもしれません。しかし、これらを学ぶことは、Reactコンポーネントだけでなく、あらゆるJavaScriptのコードを読んだり書いたりするのに役立ちます! まずは好きなものを選び、他のものがどう動くのか忘れたら、またこのリファレンスを参照してください。

<Recap>

* Reactでは、JavaScriptで分岐ロジックを制御します。
* ReactではJavaScriptで分岐ロジックを制御することができ、`if`ステートメントで条件付きでJSX式を返すことができます。
* 中括弧を使えば、あるJSXを条件付きで変数に保存し、それを他のJSXの中に入れることができます。
* JSXでは, `{cond ? <A /> : <B />}` は *"if `cond`, render `<A />`, otherwise `<B />`"*を意味します。
* JSXでは、`{cond && <A />}` は *"if `cond`, render `<A />`, otherwise nothing "*を意味します。
* これらのショートカットは一般的なものですが、プレーンな `if` が好きな人は使う必要はありません。

</Recap>



<Challenges>

### 未完成の項目には`? :`のアイコンを表示する {/*show-an-icon-for-incomplete-items-with--*/}

条件演算子 (`cond ? a : b`) を使って、 `isPacked` が `true` でない場合は ❌ をレンダリングすることができます。

<Sandpack>

```js
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {name} {isPacked && '✔'}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item 
          isPacked={true} 
          name="Space suit" 
        />
        <Item 
          isPacked={true} 
          name="Helmet with a golden leaf" 
        />
        <Item 
          isPacked={false} 
          name="Photo of Tam" 
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

<Solution>

<Sandpack>

```js
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {name} {isPacked ? '✔' : '❌'}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item 
          isPacked={true} 
          name="Space suit" 
        />
        <Item 
          isPacked={true} 
          name="Helmet with a golden leaf" 
        />
        <Item 
          isPacked={false} 
          name="Photo of Tam" 
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

</Solution>

### 項目の重要度を`&&`で表示する {/*show-the-item-importance-with-*/}

この例では、それぞれの `Item` が数値の `importance` プロパティを受け取ります。重要度が0でないアイテムに対してのみ、 `&&` 演算子を使用して "_(Importance: X)_" を斜体で表示します。アイテムリストは次のようなものになるはずです。

* 宇宙服 _(重要度:9)_」と表示されます。
* 金箔入りヘルメット
* タムの写真 _(重要度：6)_。

2つのラベルの間にスペースを入れるのを忘れないでください。

<Sandpack>

```js
function Item({ name, importance }) {
  return (
    <li className="item">
      {name}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item 
          importance={9} 
          name="Space suit" 
        />
        <Item 
          importance={0} 
          name="Helmet with a golden leaf" 
        />
        <Item 
          importance={6} 
          name="Photo of Tam" 
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

<Solution>

This should do the trick:

<Sandpack>

```js
function Item({ name, importance }) {
  return (
    <li className="item">
      {name}
      {importance > 0 && ' '}
      {importance > 0 &&
        <i>(Importance: {importance})</i>
      }
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item 
          importance={9} 
          name="Space suit" 
        />
        <Item 
          importance={0} 
          name="Helmet with a golden leaf" 
        />
        <Item 
          importance={6} 
          name="Photo of Tam" 
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

Note that you must write `importance > 0 && ...` rather than `importance && ...` so that if the `importance` is `0`, `0` isn't rendered as the result!

In this solution, two separate conditions are used to insert a space between then name and the importance label. Alternatively, you could use a fragment with a leading space: `importance > 0 && <> <i>...</i></>` or add a space immediately inside the `<i>`:  `importance > 0 && <i> ...</i>`.

</Solution>

### 一連の `? :` を `if` と変数にリファクタリングする。 {/*refactor-a-series-of---to-if-and-variables*/}

この `Drink` コンポーネントは、一連の `? :` コンディションを使用して、 `name` プロパティが `"tea"` か `"coffee"` かによって、異なる情報を表示します。問題は、それぞれの飲み物に関する情報が複数のコンディションにまたがっていることです。このコードをリファクタリングして、3つの `? :` 条件の代わりに、1つの `if` 文を使うようにします。

<Sandpack>

```js
function Drink({ name }) {
  return (
    <section>
      <h1>{name}</h1>
      <dl>
        <dt>Part of plant</dt>
        <dd>{name === 'tea' ? 'leaf' : 'bean'}</dd>
        <dt>Caffeine content</dt>
        <dd>{name === 'tea' ? '15–70 mg/cup' : '80–185 mg/cup'}</dd>
        <dt>Age</dt>
        <dd>{name === 'tea' ? '4,000+ years' : '1,000+ years'}</dd>
      </dl>
    </section>
  );
}

export default function DrinkList() {
  return (
    <div>
      <Drink name="tea" />
      <Drink name="coffee" />
    </div>
  );
}
```

</Sandpack>

一旦、`if`を使用するようにコードをリファクタリングした後、さらに簡略化するアイデアはありますか？

<Solution>

その方法は複数ありますが、ここでは一つの出発点をご紹介します：

<Sandpack>

```js
function Drink({ name }) {
  let part, caffeine, age;
  if (name === 'tea') {
    part = 'leaf';
    caffeine = '15–70 mg/cup';
    age = '4,000+ years';
  } else if (name === 'coffee') {
    part = 'bean';
    caffeine = '80–185 mg/cup';
    age = '1,000+ years';
  }
  return (
    <section>
      <h1>{name}</h1>
      <dl>
        <dt>Part of plant</dt>
        <dd>{part}</dd>
        <dt>Caffeine content</dt>
        <dd>{caffeine}</dd>
        <dt>Age</dt>
        <dd>{age}</dd>
      </dl>
    </section>
  );
}

export default function DrinkList() {
  return (
    <div>
      <Drink name="tea" />
      <Drink name="coffee" />
    </div>
  );
}
```

</Sandpack>

ここでは、各ドリンクの情報が複数の条件に分散することなく、まとめられています。これにより、将来的に飲み物を追加することが容易になります。

もう一つの解決策は、情報をオブジェクトに移動して、条件を完全に取り除くことです：

<Sandpack>

```js
const drinks = {
  tea: {
    part: 'leaf',
    caffeine: '15–70 mg/cup',
    age: '4,000+ years'
  },
  coffee: {
    part: 'bean',
    caffeine: '80–185 mg/cup',
    age: '1,000+ years'
  }
};

function Drink({ name }) {
  const info = drinks[name];
  return (
    <section>
      <h1>{name}</h1>
      <dl>
        <dt>Part of plant</dt>
        <dd>{info.part}</dd>
        <dt>Caffeine content</dt>
        <dd>{info.caffeine}</dd>
        <dt>Age</dt>
        <dd>{info.age}</dd>
      </dl>
    </section>
  );
}

export default function DrinkList() {
  return (
    <div>
      <Drink name="tea" />
      <Drink name="coffee" />
    </div>
  );
}
```

</Sandpack>

</Solution>

</Challenges>