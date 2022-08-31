---
title: UIの記述
---

<Intro>

React は、ユーザーインターフェース（UI）をレンダリングするための JavaScript ライブラリです。UI は、ボタン、テキスト、画像などの小さな単位から構築されます。React を使用すると、それらを再利用可能でネスト可能な*コンポーネント*として組み合わせることができます。この章では、React コンポーネントの作成、カスタマイズ、および条件付きレンダリングについて学びます。

</Intro>

<YouWillLearn isChapter={true}>

* [初めての React コンポーネントの作成方法](/learn/your-first-component)
* [マルチコンポーネントファイルの作成方法とタイミング](/learn/importing-and-exporting-components)
* [マークアップをJSXで JavaScript に追加する方法](/learn/writing-markup-with-jsx)
* [JSX で波括弧を使用して、コンポーネントから JavaScript の機能にアクセスする方法](/learn/javascript-in-jsx-with-curly-braces)
* [props でコンポーネントを構成する方法](/learn/passing-props-to-a-component)
* [コンポーネントを条件付きでレンダリングする方法](/learn/conditional-rendering)
* [複数のコンポーネントを一度にレンダリングする方法](/learn/rendering-lists)
* [コンポーネントを純粋に保つことで複雑なバグを回避する方法](/learn/keeping-components-pure)

</YouWillLearn>

## 初めてのコンポーネント {/*your-first-component*/}

React アプリケーションは、*コンポーネント*と呼ばれる UI の分離された部品から構築されます。React コンポーネントは、マークアップに散りばめることができる JavaScript 関数です。コンポーネントはボタンのような小さなものから、ページ全体のような大きなものまであります。以下の例では、`Gallery` コンポーネントが 3 つの `Profile` コンポーネントをレンダリングしています：

<Sandpack>

```js
function Profile() {
  return (
    <img
      src="https://i.imgur.com/MK3eW3As.jpg"
      alt="Katherine Johnson"
    />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>Amazing scientists</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

```css
img { margin: 0 10px 10px 0; height: 90px; }
```

</Sandpack>

<LearnMore path="/learn/your-first-component">

React コンポーネントの宣言と使用方法については、**[初めてのコンポーネント](/learn/your-first-component)** を参照してください。

</LearnMore>

## コンポーネントのインポートとエクスポート {/*importing-and-exporting-components*/}

1 つのファイルで多くのコンポーネントを宣言することができますが、大きなファイルを操作するのは難しくなります。これを解決するために、コンポーネントをそれぞれのファイルに*エクスポート*し、そのコンポーネントを別のファイルから*インポート*することができます：

<Sandpack>

```js App.js hidden
import Gallery from './Gallery.js';

export default function App() {
  return (
    <Gallery />
  );
}
```

```js Gallery.js active
import Profile from './Profile.js';

export default function Gallery() {
  return (
    <section>
      <h1>Amazing scientists</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

```js Profile.js
export default function Profile() {
  return (
    <img
      src="https://i.imgur.com/QIrZWGIs.jpg"
      alt="Alan L. Hart"
    />
  );
}
```

```css
img { margin: 0 10px 10px 0; }
```

</Sandpack>

<LearnMore path="/learn/importing-and-exporting-components">

コンポーネントを独自のファイルに分割する方法については、**[コンポーネントのインポートとエクスポート](/learn/importing-and-exporting-components)** を参照してください。

</LearnMore>

## JSX によるマークアップ方法 {/*writing-markup-with-jsx*/}

各 React コンポーネントは JavaScript 関数で、React がブラウザにレンダリングするマークアップを含むことができます。React コンポーネントは、マークアップを表現するために JSX と呼ばれる構文拡張を使用します。JSX は HTML とよく似ていますが、少し厳密で、動的な情報を表示することができます。

既存の HTML マークアップを React コンポーネントに貼り付けた場合、必ずしもうまくいくとは限りません：

<Sandpack>

```js
export default function TodoList() {
  return (
    // This doesn't quite work!
    <h1>Hedy Lamarr's Todos</h1>
    <img
      src="https://i.imgur.com/yXOvdOSs.jpg"
      alt="Hedy Lamarr"
      class="photo"
    >
    <ul>
      <li>Invent new traffic lights
      <li>Rehearse a movie scene
      <li>Improve spectrum technology
    </ul>
  );
}
```

```css
img { height: 90px; }
```

</Sandpack>

このような既存の HTML がある場合は、[コンバータ](https://transform.tools/html-to-jsx)を使用して修正することができます：

<Sandpack>

```js
export default function TodoList() {
  return (
    <>
      <h1>Hedy Lamarr's Todos</h1>
      <img
        src="https://i.imgur.com/yXOvdOSs.jpg"
        alt="Hedy Lamarr"
        className="photo"
      />
      <ul>
        <li>Invent new traffic lights</li>
        <li>Rehearse a movie scene</li>
        <li>Improve spectrum technology</li>
      </ul>
    </>
  );
}
```

```css
img { height: 90px; }
```

</Sandpack>

<LearnMore path="/learn/writing-markup-with-jsx">

**[JSX でマークアップを記述する](/learn/writing-markup-with-jsx)** を読んで、有効な JSX を書く方法を学んでください。

</LearnMore>

## JSX に波括弧で JavaScript を含める {/*javascript-in-jsx-with-curly-braces*/}

JSX では、JavaScript ファイルの中に HTML のようなマークアップを書くことができ、レンダリングロジックとコンテンツを同じ場所に保つことができます。時には、そのマークアップの中に、ちょっとした JavaScript のロジックを追加したり、動的な props を参照したりしたい場合があります。このような場合、JSX の中で波括弧を使用すると、JavaScript への「窓口を開く」ことができます：

<Sandpack>

```js
const person = {
  name: 'Gregorio Y. Zara',
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
};

export default function TodoList() {
  return (
    <div style={person.theme}>
      <h1>{person.name}'s Todos</h1>
      <img
        className="avatar"
        src="https://i.imgur.com/7vQD0fPs.jpg"
        alt="Gregorio Y. Zara"
      />
      <ul>
        <li>Improve the videophone</li>
        <li>Prepare aeronautics lectures</li>
        <li>Work on the alcohol-fuelled engine</li>
      </ul>
    </div>
  );
}
```

```css
body { padding: 0; margin: 0 }
body > div > div { padding: 20px; }
.avatar { border-radius: 50%; height: 90px; }
```

</Sandpack>

<LearnMore path="/learn/javascript-in-jsx-with-curly-braces">

JSX から JavaScript のデータにアクセスする方法については、**[JSX に波括弧で JavaScript を含める](/learn/javascript-in-jsx-with-curly-braces) **をお読みください。

</LearnMore>

## props をコンポーネントに渡す {/*passing-props-to-a-component*/}

React のコンポーネントは、互いに通信するために *props* を使用します。すべての親コンポーネントは、props を与えることで子コンポーネントに何らかの情報を渡すことができます。props は HTML の属性を思い起こさせるかもしれませんが、オブジェクト、配列、関数、そして JSX など、あらゆる JavaScript の値を渡すことが可能です

<Sandpack>

```js
import { getImageUrl } from './utils.js'

export default function Profile() {
  return (
    <Card>
      <Avatar
        size={100}
        person={{
          name: 'Katsuko Saruhashi',
          imageId: 'YfeOqp2'
        }}
      />
    </Card>
  );
}

function Avatar({ person, size }) {
  return (
    <img
      className="avatar"
      src={getImageUrl(person)}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}

function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

```

```js utils.js
export function getImageUrl(person, size = 's') {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    size +
    '.jpg'
  );
}
```

```css
.card {
  width: fit-content;
  margin: 5px;
  padding: 5px;
  font-size: 20px;
  text-align: center;
  border: 1px solid #aaa;
  border-radius: 20px;
  background: #fff;
}
.avatar {
  margin: 20px;
  border-radius: 50%;
}
```

</Sandpack>

<LearnMore path="/learn/passing-props-to-a-component">

props の渡し方と読み方については、**[コンポーネントに props を渡す](/learn/passing-props-to-a-component)** を読んでください。

</LearnMore>

## 条件付きレンダリング {/*conditional-rendering*/}

コンポーネントは、しばしば異なる条件によって異なるものを表示する必要があります。React では、`if` 文、`&&`、`? :` 演算子などの JavaScript 構文を使って、JSX を条件付きでレンダリングすることができます。

この例では、JavaScript の `&&` 演算子を使用して、チェックマークを条件付きでレンダリングしています：

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

<LearnMore path="/learn/conditional-rendering">

**[条件付きレンダリング](/learn/Conditional-rendering)** を読んで、コンテンツを条件付きでレンダリングするさまざまな方法を学んでください。

</LearnMore>

## リストの描画 {/*rendering-lists*/}

データの集まりから、複数の似たようなコンポーネントを表示したいことはよくあります。JavaScript の `filter()` と `map()` を React で使用すると、データの配列をフィルタリングしてコンポーネントの配列に変換することができます。

配列の各項目には、`key`を指定する必要があります。通常は、データベースからの ID を `key` として使用します。キーを指定することで、リストが変更された場合でも、React は各アイテムの位置を把握することができます。

<Sandpack>

```js App.js
import { people } from './data.js';
import { getImageUrl } from './utils.js';

export default function List() {
  const listItems = people.map(person =>
    <li key={person.id}>
      <img
        src={getImageUrl(person)}
        alt={person.name}
      />
      <p>
        <b>{person.name}:</b>
        {' ' + person.profession + ' '}
        known for {person.accomplishment}
      </p>
    </li>
  );
  return (
    <article>
      <h1>Scientists</h1>
      <ul>{listItems}</ul>
    </article>
  );
}
```

```js data.js
export const people = [{
  id: 0,
  name: 'Creola Katherine Johnson',
  profession: 'mathematician',
  accomplishment: 'spaceflight calculations',
  imageId: 'MK3eW3A'
}, {
  id: 1,
  name: 'Mario José Molina-Pasquel Henríquez',
  profession: 'chemist',
  accomplishment: 'discovery of Arctic ozone hole',
  imageId: 'mynHUSa'
}, {
  id: 2,
  name: 'Mohammad Abdus Salam',
  profession: 'physicist',
  accomplishment: 'electromagnetism theory',
  imageId: 'bE7W1ji'
}, {
  id: 3,
  name: 'Percy Lavon Julian',
  profession: 'chemist',
  accomplishment: 'pioneering cortisone drugs, steroids and birth control pills',
  imageId: 'IOjWm71'
}, {
  id: 4,
  name: 'Subrahmanyan Chandrasekhar',
  profession: 'astrophysicist',
  accomplishment: 'white dwarf star mass calculations',
  imageId: 'lrWQx8l'
}];
```

```js utils.js
export function getImageUrl(person) {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    's.jpg'
  );
}
```

```css
ul { list-style-type: none; padding: 0px 10px; }
li {
  margin-bottom: 10px;
  display: grid;
  grid-template-columns: 1fr 1fr;
  align-items: center;
}
img { width: 100px; height: 100px; border-radius: 50%; }
h1 { font-size: 22px; }
h2 { font-size: 20px; }
```

</Sandpack>

<LearnMore path="/learn/rendering-lists">

コンポーネントのリストをレンダリングする方法、およびキーを選択する方法については、**[Rendering Lists](/learn/rendering-lists)** を参照してください。

</LearnMore>

## コンポーネントを純粋に保つ {/*keeping-components-pure*/}

JavaScript の関数の中には *純粋* なものがあります：

* **自分自身の仕事だけ行う。** 呼び出される前に存在したオブジェクトや変数は変更されません。。
* **同じ入力に同じ出力。**同じ入力があれば、純粋な関数は常に同じ結果を返します。

コンポーネントを純関数としてのみ記述することで、コードベースの拡大に伴う不可解なバグや予測不可能な挙動を回避することができます。以下は、不純なコンポーネントの例です：

<Sandpack>

```js
let guest = 0;

function Cup() {
  // Bad: changing a preexisting variable!
  guest = guest + 1;
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup />
      <Cup />
      <Cup />
    </>
  )
}
```

</Sandpack>

既存の変数を変更するのではなく、props を渡すことでこのコンポーネントを純粋なものにすることができます：

<Sandpack>

```js
function Cup({ guest }) {
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup guest={1} />
      <Cup guest={2} />
      <Cup guest={3} />
    </>
  );
}
```

</Sandpack>

<LearnMore path="/learn/keeping-components-pure">

コンポーネントを純粋で予測可能な関数として記述する方法を学ぶには、 **[Keeping Components Pure](/learn/keeping-components-pure)** を読んでください。

</LearnMore>

## 次は何？ {/*whats-next*/}

[初めてのコンポーネント](/learn/your-first-component) に移動して、この章を 1 ページずつ読み始めましょう！

また、すでにこれらのトピックに精通している場合は、[インタラクティブ機能の追加](/learn/adding-interactivity)を読んでみてください。