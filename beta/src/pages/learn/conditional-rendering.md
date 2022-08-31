---
title: Conditional Rendering
---

<Intro>

コンポーネントでは、異なる条件によって異なるものを表示する必要が出てきます。React では、`if` 文、`&&`、`? :` 演算子などの JavaScript 構文を使って JSX を条件付きでレンダリングすることができます。

</Intro>

<YouWillLearn>

* 条件によって異なる JSX を返す方法
* JSX の一部を条件付きで含めたり除外する方法
* React の一般的な条件構文のショートカット

</YouWillLearn>

## 条件付きで JSX を返す {/*conditionally-returning-jsx*/}

例えば、`PackingList` コンポーネントがいくつかの `Item` をレンダリングしているとします：

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

`Item` コンポーネントの中には、 `isPacked` props が `false` ではなく `true` に設定されているものがあることに注意してください。パックされたアイテムにチェックマーク(✔)を付けたい場合、`isPacked={true}` を指定します。

これは以下のような [`if`/`else` 文](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/if...else)として書くことができます。

```js
if (isPacked) {
  return <li className="item">{name} ✔</li>;
}
return <li className="item">{name}</li>;
```

`isPacked` props が `true` の場合、このコードは **別の JSX ツリー** を返します。この変更により、いくつかの項目は最後にチェックマークが付くようになりました：

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

両方のケースで返されるものを編集して、結果がどう変わるか見てみましょう!

JavaScript の `if` と `return` 文で、分岐したロジックをどのように作成しているかに注目してください。React では、制御フロー（条件）は JavaScript で処理されます。

### `null` を使い、条件付きで何も返さない {/*conditionally-returning-nothing-with-null*/}

状況によっては、何もレンダリングしたくないことがあります。例えば、パックされたアイテムを全く表示したくないとします。コンポーネントは何かを返さなければなりません。この場合、`null` を返すことができます：

```js
if (isPacked) {
  return null;
}
return <li className="item">{name}</li>;
```

`isPacked` が true の場合、コンポーネントは何も返さず、 `null` を返します。そうでない場合は、レンダリングするための JSX を返します。

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

実際には、コンポーネントから `null` を返すことは一般的ではありません。それよりも、親コンポーネントの JSX に条件付きでコンポーネントを含めたり除外したりすることが多いでしょう。これからその方法を説明します。

## JSX を条件付きで含める {/*conditionally-including-jsx*/}

前の例では、コンポーネントから返される JSX ツリーを制御していました。レンダリング出力に重複があることにすでにお気づきかもしれません。

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

この重複は有害ではありませんが、コードの保守を困難にする可能性があります。もし、 `className` を変更したい場合はどうすればよいでしょうか？コードの 2 箇所で変更を行わなければなりません。このような場合、条件付きで小さな JSX を含めることで、コードをより [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) にすることができます。

### 条件（三項）演算子 （`? :`） {/*conditional-ternary-operator--*/}

JavaScript には条件式を書くためのコンパクトな構文があります。[条件演算子](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_Operator) または 「三項演算子」 です。

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

この式は *「もし `isPacked`が`true`なら、 (`?`) は `name + ' ✔'`を描画し、そうでなければ (`:`) は `name` を描画する」* と読むことができます。

<DeepDive title="この2つの例は完全に等価なのか？">

もしあなたがオブジェクト指向プログラミングの経験を持っているなら、上の二つの例は微妙に違うと思うかもしれません。なぜなら、どちらかが `<li>` の二つの異なる「インスタンス」を作成するかもしれないからです。しかし、JSX 要素は内部状態を保持せず、実際の DOM ノードでもないため、「インスタンス」ではありません。JSX 要素はあくまで設計図のような軽量の記述なのです。ですから、この 2 つの例は実際には完全に等価です。[Preserving and Resetting State](/learn/preserving-and-resetting-state)  は、この仕組みについて詳しく説明しています。

</DeepDive>

次は、完成した項目のテキストを、`<del>` のように別の HTML タグで囲んで打ち消してみます。さらに改行や括弧を追加して、それぞれのケースでより多くの JSX をネストしやすくすることができます：

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

このスタイルは単純な条件には有効ですが、適度に使ってください。ネストされた条件マークアップが多すぎてコンポーネントがごちゃごちゃになる場合は、子コンポーネントを抽出してすっきりさせることを検討してください。React では、マークアップはコードの一部なので、変数や関数のようなツールを使って複雑な式を整理することができます。

### 論理 AND 演算子 (`&&`) {/*logical-and-operator-*/}

もうひとつよく見かけるショートカットに [JavaScript の論理 AND 演算子(`&&`)](<https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_AND#:~:text=The%20logical%20AND%20(%20%26%26%20)%20operator,it%20returns%20a%20Boolean%20value.>) があります. React コンポーネントの内部では、ある条件が true のときに JSX をレンダリングし、そうでないときは何もレンダリングしない、ということがよく出てきます：

```js
return (
  <li className="item">
    {name} {isPacked && '✔'}
  </li>
);
```

これは *「もし`isPacked` があれば、(`&&`) はチェックマークを描画し、そうでなければ、何も描画しない 」*と読むことができます。

実際に以下で試してみましょう：

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

[JavaScript && 式](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_AND)は、左辺（条件）が `true` なら右辺（この場合、チェックマーク）の値を返す。しかし、条件が `false` であれば、式全体が `false` になります。React は `false` を `null` や `undefined` と同じように JSX ツリーの「穴」とみなし、その場所には何もレンダリングしません。

<Gotcha>

**`&&` の左側に数字を書かないでください。**

この条件を満たすために、JavaScript は左辺を自動的にブーリアンに変換します。しかし、左辺が `0` であれば、式全体がその値 (`0`) を取得し、React は何もしないのではなく、喜んで `0` をレンダリングします。

例えば、よくある間違いは、`messageCount && <p>New messages</p>` のようなコードを書くことです。これは `messageCount` が `0` のときに何もレンダリングしないと思いがちですが、実際には `0` そのものをレンダリングしているのです!

これを修正するには、左側を `messageCount > 0 && <p>New messages</p>` というブール型にします。

</Gotcha>

### 条件付きで JSX を変数に代入する {/*conditionally-assigning-jsx-to-a-variable*/}

ショートカットがプレーンなコードを書くのに邪魔になったら、`if` 文と変数を使ってみてください。[`let`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let) で定義された変数は再代入が可能なので、まずは表示したいデフォルトの内容、名前を用意しましょう。

```js
let itemContent = name;
```

`if` 文を使用して、`isPacked` が `true` であれば、`itemContent` に JSX 式を再割り当てします。

```js
if (isPacked) {
  itemContent = name + " ✔";
}
```

[波括弧は JavaScript への窓口を開けます。](/learn/javascript-in-jsx-with-curly-braces#using-curly-braces-a-window-into-the-javascript-world) 波括弧の付いた変数を返却する JSX ツリーに埋め込み、前に計算した式を JSX 内部に入れ子にします。

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

前回同様、テキストだけでなく、任意の JSX に対しても有効です：

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

JavaScript に慣れていないと、この多様なスタイルに最初は圧倒されるかもしれません。しかし、これらを学ぶことは、React コンポーネントだけでなく、あらゆる JavaScript のコードを読んだり書いたりするのに役立ちます! まずは好きな記述方法を選び、他の方法がどう動くのか忘れたらまたこのリファレンスを参照してください。

<Recap>

- React では、JavaScript で分岐ロジックを制御します。
- `if`文を使って条件付きで JSX 式を返すことができます。
- 波括弧を使えば、ある JSX を条件付きで変数に保存し、それを他の JSX の中に入れることができます。
- JSX では, `{cond ? <A /> : <B />}` は *「もし `cond` が真なら、 `<A />`を描画し、そうでなければ `<B />` を描画する」*を意味します。
- JSX では、`{cond && <A />}` は *「もし `cond` が真なら、`<A />` を描画し、そうでなければ何もしない」*を意味します。
- これらのショートカットは一般的なものですが、プレーンな `if` が好きな人は使う必要はありません。

</Recap>



<Challenges>

### 未完成の項目には`? :`のアイコンを表示する {/*show-an-icon-for-incomplete-items-with--*/}

条件演算子 (`cond ? a : b`) を使って、 `isPacked` が `true` でない場合は ❌ を描画しましょう。

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

この例では、それぞれの `Item` が数値の `importance` props を受け取ります。重要度が 0 でないアイテムに対してのみ、 `&&` 演算子を使用して 「_(Importance: X)_」 を斜体で表示します。アイテムリストは次のようなものになるはずです。

* Space suit _(Importance: 9)_
* Helmet with a golden leaf
* Photo of Tam _(Importance: 6)_

2 つのラベルの間にスペースを入れるのを忘れないでください。

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

`importance && ...`ではなく、`importance > 0 && ...` と書かなければならないことに注意してください。前者の場合、`importance` が `0` の場合正しく描画されません。

この回答では、条件を2つに分割して、`name`と`importance`ラベルの間にスペースを入れるために使用しています。代わりに、 `importance > 0 && <> <i>...</i></>` の様にフラグメントを使用して先頭にスペースを入れたり、`<i>`:  `importance > 0 && <i> ...</i>` の様にスペースを直接いれることもできます。

</Solution>

### 一連の `? :` を `if` と変数にリファクタリングする。 {/*refactor-a-series-of---to-if-and-variables*/}

この `Drink` コンポーネントは、一連の `? :` コンディションを使用して、 `name` props が `"tea"` か `"coffee"` かによって、異なる情報を表示します。問題は、それぞれの飲み物に関する情報が複数の条件にまたがっていることです。このコードをリファクタリングして、3 つの `? :` 条件の代わりに、1 つの `if` 文を使うようにしましょう。

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

`if`を使用するようにコードをリファクタリングした後、さらに簡略化するアイデアはありますか？

<Solution>

方法は複数ありますが、ここでは一つの出発点を紹介します：

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