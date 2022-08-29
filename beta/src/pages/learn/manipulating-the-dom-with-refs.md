---
title: 'Refを使ったDOMの操作'
---

<Intro>

React はレンダリング出力に合わせて [DOM](https://developer.mozilla.org/docs/Web/API/Document_Object_Model/Introduction) を自動的に更新するので、コンポーネントがそれを操作する必要はあまりありません。しかし、時にはReactが管理するDOM要素にアクセスする必要があるかもしれません。例えば、ノードにフォーカスを当てたり、スクロールしたり、サイズや位置を測定したする場合です。Reactにはそのようなことをするための組み込みの方法はありませんので、DOMノードへの*ref*が必要になります。

</Intro>

<YouWillLearn>

- React が管理する DOM ノードに `ref` 属性でアクセスする方法
- JSXの `ref` 属性と `useRef` フックとの関連性
- 他のコンポーネントの DOM ノードにアクセスする方法
- React が管理する DOM を変更しても安全なのはどのような場合か

</YouWillLearn>

## ノードの ref を取得する {/*getting-a-ref-to-the-node*/}

React が管理する DOM ノードにアクセスするには、まず、 `useRef` フック をインポートします：

```js
import { useRef } from 'react';
```

次に、それを使ってコンポーネント内でrefを宣言します：

```js
const myRef = useRef(null);
```

最後に、DOM ノードに `ref` 属性として渡します：

```js
<div ref={myRef}>
```

`useRef` フック は `current` という一つのプロパティを持つオブジェクトを返します。初期状態では、 `myRef.current` は `null` になります。React がこの `<div>` の DOM ノードを作成すると、React はこのノードへの参照を `myRef.current` に格納します。この DOM ノードに [イベントハンドラ] (/learn/responding-to-events) からアクセスし、このノードで定義された組み込みの [browser API](https://developer.mozilla.org/docs/Web/API/Element) を使用することが可能になります。

```js
// You can use any browser APIs, for example:
myRef.current.scrollIntoView();
```

### 例：テキスト入力にフォーカスを当てる {/*example-focusing-a-text-input*/}

この例では、ボタンをクリックすると入力にフォーカスが当たります：

<Sandpack>

```js
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```

</Sandpack>

これを実現するためには：

1. `useRef`フックで `inputRef` を宣言します。
2. `<input ref={inputRef}>`として渡します。これはReactに、**この `<input>` のDOMノードを `inputRef.current` に入れる**ように伝えます。
3. `handleClick` 関数で、 `inputRef.current` から入力 DOM ノードを読み込んで、 `inputRef.current.focus()` で [`focus()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus) を呼び出します。
4. `onClick`で、`<button>` に `handleClick` イベントハンドラを渡します。

DOM操作がrefの最も一般的な使用例ですが、`useRef` フックはタイマーIDのようなReactの外部で他のものを保存するために使用することができます。refについては[Refを使った値の参照](/learn/referencing-values-with-refs)に詳しく記述してあります。

### 例：要素にスクロールする {/*example-scrolling-to-an-element*/}

1つのコンポーネントの中に、複数のrefを持つことができます。以下の例では、3つの画像のカルーセルがあります。各ボタンは、対応する DOM ノードからブラウザの [`scrollIntoView()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollIntoView) メソッドを呼び出し画像が画面中央に来るようスクロールします：

<Sandpack>

```js
import { useRef } from 'react';

export default function CatFriends() {
  const firstCatRef = useRef(null);
  const secondCatRef = useRef(null);
  const thirdCatRef = useRef(null);

  function handleScrollToFirstCat() {
    firstCatRef.current.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  function handleScrollToSecondCat() {
    secondCatRef.current.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  function handleScrollToThirdCat() {
    thirdCatRef.current.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  return (
    <>
      <nav>
        <button onClick={handleScrollToFirstCat}>
          Tom
        </button>
        <button onClick={handleScrollToSecondCat}>
          Maru
        </button>
        <button onClick={handleScrollToThirdCat}>
          Jellylorum
        </button>
      </nav>
      <div>
        <ul>
          <li>
            <img
              src="https://placekitten.com/g/200/200"
              alt="Tom"
              ref={firstCatRef}
            />
          </li>
          <li>
            <img
              src="https://placekitten.com/g/300/200"
              alt="Maru"
              ref={secondCatRef}
            />
          </li>
          <li>
            <img
              src="https://placekitten.com/g/250/200"
              alt="Jellylorum"
              ref={thirdCatRef}
            />
          </li>
        </ul>
      </div>
    </>
  );
}
```

```css
div {
  width: 100%;
  overflow: hidden;
}

nav {
  text-align: center;
}

button {
  margin: .25rem;
}

ul,
li {
  list-style: none;
  white-space: nowrap;
}

li {
  display: inline;
  padding: 0.5rem;
}
```

</Sandpack>

<DeepDive title="How to manage a list of refs using a ref callback">

上記の例では、あらかじめ定義された数のrefが存在します。しかし、時にはリストの各項目への参照が必要な場合があり、その数が分からないことがあります。このような場合、**うまくいかないでしょう**：

```js
<ul>
  {items.map((item) => {
    // Doesn't work!
    const ref = useRef(null);
    return <li ref={ref} />;
  })}
</ul>
```

なぜなら、 **フックはコンポーネントのトップレベルでのみ呼び出せる* らです。 ループや条件、`map()` の中では `useRef` を呼び出すことができません。

これを回避する一つの方法として、親要素への単一の ref を取得し、そこから個々の子ノードを「見つける」ために [`querySelectorAll`](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelectorAll) のような DOM 操作のメソッドを使用することが考えられます。しかし、これは脆く、DOM構造が変わると壊れてしまう可能性があります。

もう一つの解決策は、**`ref` 属性に関数を渡す**ことです。これは「ref callback」と呼ばれます。React は、ref を設定する時には DOM ノードで、ref をクリアする時には `null` で、ref コールバックを呼び出します。これにより、独自の配列や[Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)を保持し、インデックスや何らかのIDを使用して参照にアクセスすることができます。

以下の例では、この方法を用いて、長いリストの中の任意のノードにスクロールする方法を示しています：

<Sandpack>

```js
import { useRef } from 'react';

export default function CatFriends() {
  const itemsRef = useRef(null);

  function scrollToId(itemId) {
    const map = getMap();
    const node = map.get(itemId);
    node.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  function getMap() {
    if (!itemsRef.current) {
      // Initialize the Map on first usage.
      itemsRef.current = new Map();
    }
    return itemsRef.current;
  }

  return (
    <>
      <nav>
        <button onClick={() => scrollToId(0)}>
          Tom
        </button>
        <button onClick={() => scrollToId(5)}>
          Maru
        </button>
        <button onClick={() => scrollToId(9)}>
          Jellylorum
        </button>
      </nav>
      <div>
        <ul>
          {catList.map(cat => (
            <li
              key={cat.id}
              ref={(node) => {
                const map = getMap();
                if (node) {
                  map.set(cat.id, node);
                } else {
                  map.delete(cat.id);
                }
              }}
            >
              <img
                src={cat.imageUrl}
                alt={'Cat #' + cat.id}
              />
            </li>
          ))}
        </ul>
      </div>
    </>
  );
}

const catList = [];
for (let i = 0; i < 10; i++) {
  catList.push({
    id: i,
    imageUrl: 'https://placekitten.com/250/200?image=' + i
  });
}

```

```css
div {
  width: 100%;
  overflow: hidden;
}

nav {
  text-align: center;
}

button {
  margin: .25rem;
}

ul,
li {
  list-style: none;
  white-space: nowrap;
}

li {
  display: inline;
  padding: 0.5rem;
}
```

</Sandpack>

この例では、`itemsRef`は単一のDOMノードを保持しているわけではありません。その代わり、アイテム ID から DOM ノードへの [Map](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Map) を保持しています。([Refs can hold any values!](/learn/referencing-values-with-refs)) 各リストアイテムの `ref` コールバックは、Mapを更新するように配慮されています：

```js
<li
  key={cat.id}
  ref={node => {
    const map = getMap();
    if (node) {
      // Add to the Map
      map.set(cat.id, node);
    } else {
      // Remove from the Map
      map.delete(cat.id);
    }
  }}
>
```

これにより、後でマップから個々の DOM ノードを読み取ることができます。

</DeepDive>

## 他コンポーネントのDOMノードへのアクセス {/*accessing-another-components-dom-nodes*/}

ブラウザの要素を出力する組み込みコンポーネントに `<input />` のような Ref を置くと、React はその Ref の `current` プロパティを対応する DOM ノード（ブラウザ上の実際の `<input />` など）にセットします。

しかし、 `<MyInput />` のような **自分自身の** コンポーネントにrefを設定しようとすると、デフォルトでは `null` を取得することになります。ボタンをクリックしても入力にフォーカスされないことに注意してください：

<Sandpack>

```js
import { useRef } from 'react';

function MyInput(props) {
  return <input {...props} />;
}

export default function MyForm() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```

</Sandpack>

ボタンをクリックすると、コンソールにエラーメッセージが表示されます。

<ConsoleBlock level="error">

Warning: Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()?

</ConsoleBlock>

これは、Reactのデフォルトでは、コンポーネントが他のコンポーネントのDOMノードにアクセスできないために起こります。自分の子であってもダメです! これは意図的なものです。refは逃げ道であり、控えめに使うべきものです。他のコンポーネントのDOMノードを手動で操作すると、コードがより脆弱になります。

その代わり、DOM ノードを公開したいコンポーネントは、その挙動を **opt in** しなければなりません。コンポーネントは、その子コンポーネントのひとつにrefを「転送」するように指定することができます。以下は、 `MyInput` が `forwardRef` API を使用する方法です：

```js
const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});
```

以下のように動作します：

1. `<MyInput ref={inputRef}>`はReactに対応するDOMノードを`inputRef.current`に入れるように指示します。しかし、`MyInput` コンポーネントがこれを受け入れるかどうかは、`MyInput` コンポーネント次第です（デフォルトでは受け入れません）。
2. `MyInput` コンポーネントは `forwardRef` を使用して宣言されています。これは、 `props` の後に宣言された `ref` の第二引数として、上記の `inputRef` を受け取るようにするものです。
3. `MyInput` 自身は、受け取った `ref` を内部の `<input>` に渡します。

これで、ボタンをクリックして入力にフォーカスを当てることができるようになりました：

<Sandpack>

```js
import { forwardRef, useRef } from 'react';

const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```

</Sandpack>

デザインシステムでは、ボタンや入力などの低レベルのコンポーネントは、そのDOMノードへのrefを転送するのが一般的なパターンです。一方、フォーム、リスト、ページセクションなどの高レベルコンポーネントは、DOM構造への偶発的な依存を避けるために、通常DOMノードを公開しません。

<DeepDive title="命令型ハンドルでAPIのサブセットを公開する">

上記の例では、 `MyInput` はオリジナルの DOM input 要素を公開しています。これにより、親コンポーネントが `focus()` を呼び出すことができます。しかし、これによって親コンポーネントは他のこと、例えば CSS スタイルを変更することもできます。一部のケースでは、公開する機能を制限したいかもしれません。これは `useImperativeHandle` を用いて行うことができます：

<Sandpack>

```js
import {
  forwardRef, 
  useRef, 
  useImperativeHandle
} from 'react';

const MyInput = forwardRef((props, ref) => {
  const realInputRef = useRef(null);
  useImperativeHandle(ref, () => ({
    // Only expose focus and nothing else
    focus() {
      realInputRef.current.focus();
    },
  }));
  return <input {...props} ref={realInputRef} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```

</Sandpack>

この例では、 `MyInput` 内の `realInputRef` は実際の入力 DOM ノードを保持しています。しかし、`useImperativeHandle`は、親コンポーネントのrefの値として、独自の特別なオブジェクトを提供するようにReactに指示します。そのため、`Form` コンポーネント内の `inputRef.current` は `focus` メソッドのみを持つことになります。この場合、ref が扱うのは DOM ノードではなく、 `useImperativeHandle` 呼び出し内で作成したカスタムオブジェクトになります。

</DeepDive>

## reactはいつrefをアタッチするか {/*when-react-attaches-the-refs*/}

Reactでは、すべての更新は[2つのフェーズ](/learn/render and-commit#step-3-react-commits-changes-to-the-dom) に分かれています。

* **レンダー**時、Reactはコンポーネントを呼び出して、画面に表示されるべきものを見つける。
* **コミット**時、ReactはDOMに変更を加える。

一般的に、レンダリング中にrefにアクセスすることは[したくない](/learn/referencing-values-with-refs#best-practices-for-refs)と言われています。これは、DOM ノードを保持する ref にも当てはまります。最初のレンダリングでは、DOM ノードはまだ作成されていないので、 `ref.current` は `null` になります。また、更新のレンダリング中はDOM ノードはまだ更新されていません。つまり、読むにはまだ早すぎるのです。

React はコミット時に `ref.current` をセットします。DOM を更新する前に、React は影響を受ける `ref.current` 値を `null` に設定します。DOM を更新した後、React は直ちにそれらを対応する DOM ノードに設定します。

**通常、refにはイベントハンドラからアクセス**します。 refで何かをしたいがそれを実行する特定のイベントがない場合、effectが必要になることがあります。effectについては、次のページで説明します。

<DeepDive title="flushSyncによってstateを同期的に更新するよう強制する">

新しいTodoを追加して、リストの最後の子まで画面をスクロールさせる、次のようなコードを考えてみましょう。何らかの理由で、常に最後に追加された ToDo の *直前* の ToDo にスクロールしていることに注意してください：

<Sandpack>

```js
import { useState, useRef } from 'react';

export default function TodoList() {
  const listRef = useRef(null);
  const [text, setText] = useState('');
  const [todos, setTodos] = useState(
    initialTodos
  );

  function handleAdd() {
    const newTodo = { id: nextId++, text: text };
    setText('');
    setTodos([ ...todos, newTodo]);
    listRef.current.lastChild.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest'
    });
  }

  return (
    <>
      <button onClick={handleAdd}>
        Add
      </button>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <ul ref={listRef}>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}

let nextId = 0;
let initialTodos = [];
for (let i = 0; i < 20; i++) {
  initialTodos.push({
    id: nextId++,
    text: 'Todo #' + (i + 1)
  });
}
```

</Sandpack>

問題はこの2行にあります：

```js
setTodos([ ...todos, newTodo]);
listRef.current.lastChild.scrollIntoView();
```

Reactでは、[stateの更新をキューに入れる](/learn/queueing-a-series-of-state-updates)ことができます。通常、これは望ましい動作です。しかし、ここでは`setTodos` がすぐに DOM を更新しないため問題が発生します。リストを最後の要素までスクロールした時点では、TODO はまだ追加されていません。そのため、スクロールが常に「1項目分遅れて」行われるのです。

この問題を解決するに、Reactに強制的にDOMを同期的に更新（"フラッシュ"）させることができます。これを行うには、 `react-dom` から `flushSync` をインポートし、**stateの更新を `flushSync` 呼び出しにラップ**します。

```js
flushSync(() => {
  setTodos([ ...todos, newTodo]);
});
listRef.current.lastChild.scrollIntoView();
```

これは、`flushSync`でラップされたコードが実行された直後に、ReactにDOMを同期的に更新するように指示します。その結果、最後のTodoにスクロールしようとしたときには、すでにDOMに表示されていることになります：

<Sandpack>

```js
import { useState, useRef } from 'react';
import { flushSync } from 'react-dom';

export default function TodoList() {
  const listRef = useRef(null);
  const [text, setText] = useState('');
  const [todos, setTodos] = useState(
    initialTodos
  );

  function handleAdd() {
    const newTodo = { id: nextId++, text: text };
    flushSync(() => {
      setText('');
      setTodos([ ...todos, newTodo]);      
    });
    listRef.current.lastChild.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest'
    });
  }

  return (
    <>
      <button onClick={handleAdd}>
        Add
      </button>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <ul ref={listRef}>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}

let nextId = 0;
let initialTodos = [];
for (let i = 0; i < 20; i++) {
  initialTodos.push({
    id: nextId++,
    text: 'Todo #' + (i + 1)
  });
}
```

</Sandpack>

</DeepDive>

## refを使った DOM 操作のベストプラクティス {/*best-practices-for-dom-manipulation-with-refs*/}

Ref は逃げ道です。「Reactの外に出る」必要がある場合にのみ使用する必要があります。よくある例としては、フォーカスやスクロール位置の管理、Reactが公開していないブラウザAPIの呼び出しなどがあります。

フォーカスやスクロールのような非破壊的な動作にこだわるのであれば、特に問題は発生しないはずです。しかし、手動でDOMを**変更**しようとすると、Reactが行っている変更と競合する危険性があります。

この問題を説明するために、この例ではウェルカムメッセージと2つのボタンを使っています。最初のボタンは、通常 React で行うように [条件付きレンダリング](/learn/conditional-rendering) と [state](/learn/state-a-components-memory) を使用してその表示を切り替えています。2つ目のボタンは、[`remove()` DOM API](https://developer.mozilla.org/en-US/docs/Web/API/Element/remove) を使って、Reactの制御外のDOMから強制的に削除しています。

「Toggle with setState」を何度か押してみてください。メッセージが消えたり、また現れたりするはずです。そして「Remove from the DOM」を押してください。これで強制的に削除されます。最後に「Toggle with setState」を押してください：

<Sandpack>

```js
import {useState, useRef} from 'react';

export default function Counter() {
  const [show, setShow] = useState(true);
  const ref = useRef(null);

  return (
    <div>
      <button
        onClick={() => {
          setShow(!show);
        }}>
        Toggle with setState
      </button>
      <button
        onClick={() => {
          ref.current.remove();
        }}>
        Remove from the DOM
      </button>
      {show && <p ref={ref}>Hello world</p>}
    </div>
  );
}
```

```css
p,
button {
  display: block;
  margin: 10px;
}
```

</Sandpack>

DOM 要素を手動で削除した後、`setState` を使って再び表示しようとするとクラッシュします。これは DOM を変更したためで、React はそれを正しく管理し続ける方法を知らないのです。

Reactによって管理されているDOMノードを変更しないようにしましょう。Reactによって管理されている要素を変更したり、子を追加したり、子から削除したりすると、一貫性のない表示結果や上記のようなクラッシュを引き起こす可能性があります。

ただし、全くできないわけではありません。注意が必要です。**Reactが更新する _理由のない_ DOMの一部分であれば安全に変更することができます。** 例えば、ある `<div>` が JSX で常に空であれば、React はその子要素リストを触る理由がありません。したがって、そのDOMに手動で要素を追加したり削除したりすることは安全です。

<Recap>

- Refは一般的な概念ですが、多くの場合、DOM要素を保持するために使用します。
- React に DOM ノードを `myRef.current` に入れるように指示するには、`<div ref={myRef}>` を渡します。
- 通常、フォーカスやスクロール、DOM要素の計測など、非破壊的な操作のためにrefを使用します。
- コンポーネントは、デフォルトではその DOM ノードを公開しません。`forwardRef` を使用し、第二引数の `ref` を特定のノードに渡すことで、DOM ノードの公開を選択することができます。
- React が管理する DOM ノードを変更しないようにしましょう。
- React が管理する DOM ノードを変更する場合は、React が更新する理由のない部分を変更します。

</Recap>



<Challenges>

### 動画を再生・一時停止する {/*play-and-pause-the-video*/}

この例では、ボタンがstate変数をトグルして、再生と一時停止の状態を切り替えています。しかし、実際にビデオを再生したり一時停止したりするには、状態を切り替えるだけでは不十分です。また、`<video>` の DOM 要素上で [`play()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/play) と [`pause()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/pause) を呼び出す必要があります。refを追加して、ボタンを動作させてみましょう。

<Sandpack>

```js
import { useState, useRef } from 'react';

export default function VideoPlayer() {
  const [isPlaying, setIsPlaying] = useState(false);

  function handleClick() {
    const nextIsPlaying = !isPlaying;
    setIsPlaying(nextIsPlaying);
  }

  return (
    <>
      <button onClick={handleClick}>
        {isPlaying ? 'Pause' : 'Play'}
      </button>
      <video width="250">
        <source
          src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
          type="video/mp4"
        />
      </video>
    </>
  )
}
```

```css
button { display: block; margin-bottom: 20px; }
```

</Sandpack>

さらに発展的な内容として、ユーザーがビデオを右クリックしてブラウザの内蔵メディアコントロールを使って再生しても、「再生」ボタンとビデオが再生されているかどうかを同期させておくことが挙げられます。そのためには、ビデオの `onPlay` と `onPause` に耳を傾けるのがよいでしょう。

<Solution>

ref を宣言し、`<video>` 要素に配置します。そして、次のstateに応じてイベントハンドラで `ref.current.play()` と `ref.current.pause()` を呼び出します。

<Sandpack>

```js
import { useState, useRef } from 'react';

export default function VideoPlayer() {
  const [isPlaying, setIsPlaying] = useState(false);
  const ref = useRef(null);

  function handleClick() {
    const nextIsPlaying = !isPlaying;
    setIsPlaying(nextIsPlaying);

    if (nextIsPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  }

  return (
    <>
      <button onClick={handleClick}>
        {isPlaying ? 'Pause' : 'Play'}
      </button>
      <video
        width="250"
        ref={ref}
        onPlay={() => setIsPlaying(true)}
        onPause={() => setIsPlaying(false)}
      >
        <source
          src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
          type="video/mp4"
        />
      </video>
    </>
  )
}
```

```css
button { display: block; margin-bottom: 20px; }
```

</Sandpack>

組み込みのブラウザコントロールを処理するために、 `<video>` 要素に `onPlay` と `onPause` ハンドラを追加して、そこから `setIsPlaying` を呼び出すことができます。この方法では、ユーザーがブラウザコントロールを使用して動画を再生した場合、それに応じてstateが調整されます。

</Solution>

### 検索フィールドをフォーカスする {/*focus-the-search-field*/}

「検索」ボタンをクリックすると、フィールドにフォーカスが当たるようにします。

<Sandpack>

```js
export default function Page() {
  return (
    <>
      <nav>
        <button>Search</button>
      </nav>
      <input
        placeholder="Looking for something?"
      />
    </>
  );
}
```

```css
button { display: block; margin-bottom: 10px; }
```

</Sandpack>

<Solution>

input に ref を追加し、DOM ノード上で `focus()` を呼び出してフォーカスを当てます：

<Sandpack>

```js
import { useRef } from 'react';

export default function Page() {
  const inputRef = useRef(null);
  return (
    <>
      <nav>
        <button onClick={() => {
          inputRef.current.focus();
        }}>
          Search
        </button>
      </nav>
      <input
        ref={inputRef}
        placeholder="Looking for something?"
      />
    </>
  );
}
```

```css
button { display: block; margin-bottom: 10px; }
```

</Sandpack>

</Solution>

### 画像カルーセルのスクロール {/*scrolling-an-image-carousel*/}

この画像カルーセルには、アクティブな画像を切り替える「次へ」ボタンがあります。クリックするとギャラリーがアクティブなイメージまで水平にスクロールするようにしましょう。アクティブな画像のDOMノードで[`scrollIntoView()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollIntoView)を呼び出したいはずです：

```js
node.scrollIntoView({
  behavior: 'smooth',
  block: 'nearest',
  inline: 'center'
});
```

<Hint>

この演習では、すべての画像への参照を持つ必要はありません。現在アクティブな画像への参照、またはリスト自体への参照を持つだけで十分です。`flushSync` を使用して、スクロールする前に DOM が更新されるようにします。

</Hint>

<Sandpack>

```js
import { useState } from 'react';

export default function CatFriends() {
  const [index, setIndex] = useState(0);
  return (
    <>
      <nav>
        <button onClick={() => {
          if (index < catList.length - 1) {
            setIndex(index + 1);
          } else {
            setIndex(0);
          }
        }}>
          Next
        </button>
      </nav>
      <div>
        <ul>
          {catList.map((cat, i) => (
            <li key={cat.id}>
              <img
                className={
                  index === i ?
                    'active' :
                    ''
                }
                src={cat.imageUrl}
                alt={'Cat #' + cat.id}
              />
            </li>
          ))}
        </ul>
      </div>
    </>
  );
}

const catList = [];
for (let i = 0; i < 10; i++) {
  catList.push({
    id: i,
    imageUrl: 'https://placekitten.com/250/200?image=' + i
  });
}

```

```css
div {
  width: 100%;
  overflow: hidden;
}

nav {
  text-align: center;
}

button {
  margin: .25rem;
}

ul,
li {
  list-style: none;
  white-space: nowrap;
}

li {
  display: inline;
  padding: 0.5rem;
}

img {
  padding: 10px;
  margin: -10px;
  transition: background 0.2s linear;
}

.active {
  background: rgba(0, 100, 150, 0.4);
}
```

</Sandpack>

<Solution>

 `selectedRef`を宣言することで、現在の画像に対してのみ条件付きで `selectedRef` を渡すことができます：

```js
<li ref={index === i ? selectedRef : null}>
```

`index === i` の場合、つまり画像が選択されている場合、 `<li>` は `selectedRef` を受け取ります。react は `selectedRef.current` が常に正しい DOM ノードを指していることを確認します。

`flushSync` の呼び出しは、React がスクロールする前に DOM を更新するために必要です。そうしないと、 `selectedRef.current` は常に前に選択されたアイテムを指すことになります。

<Sandpack>

```js
import { useRef, useState } from 'react';
import { flushSync } from 'react-dom';

export default function CatFriends() {
  const selectedRef = useRef(null);
  const [index, setIndex] = useState(0);

  return (
    <>
      <nav>
        <button onClick={() => {
          flushSync(() => {
            if (index < catList.length - 1) {
              setIndex(index + 1);
            } else {
              setIndex(0);
            }
          });
          selectedRef.current.scrollIntoView({
            behavior: 'smooth',
            block: 'nearest',
            inline: 'center'
          });            
        }}>
          Next
        </button>
      </nav>
      <div>
        <ul>
          {catList.map((cat, i) => (
            <li
              key={cat.id}
              ref={index === i ?
                selectedRef :
                null
              }
            >
              <img
                className={
                  index === i ?
                    'active'
                    : ''
                }
                src={cat.imageUrl}
                alt={'Cat #' + cat.id}
              />
            </li>
          ))}
        </ul>
      </div>
    </>
  );
}

const catList = [];
for (let i = 0; i < 10; i++) {
  catList.push({
    id: i,
    imageUrl: 'https://placekitten.com/250/200?image=' + i
  });
}

```

```css
div {
  width: 100%;
  overflow: hidden;
}

nav {
  text-align: center;
}

button {
  margin: .25rem;
}

ul,
li {
  list-style: none;
  white-space: nowrap;
}

li {
  display: inline;
  padding: 0.5rem;
}

img {
  padding: 10px;
  margin: -10px;
  transition: background 0.2s linear;
}

.active {
  background: rgba(0, 100, 150, 0.4);
}
```

</Sandpack>

</Solution>

### 分離したコンポーネントで検索フィールドをフォーカスする {/*focus-the-search-field-with-separate-components*/}. {/*分離したコンポーネントで検索フィールドをフォーカスする-focus-the-search-field-with-separate-components*/}

「検索」ボタンをクリックすると、そのフィールドにフォーカスが当たるようにしましょう。なお、各コンポーネントは別々のファイルに定義されているので、そこから移動させてはいけません。どのように各コンポーネントを接続すればよいでしょうか？

<Hint>

`SearchInput` のような独自のコンポーネントから DOM ノードを公開するためには、 `forwardRef` が必要です。

</Hint>

<Sandpack>

```js App.js
import SearchButton from './SearchButton.js';
import SearchInput from './SearchInput.js';

export default function Page() {
  return (
    <>
      <nav>
        <SearchButton />
      </nav>
      <SearchInput />
    </>
  );
}
```

```js SearchButton.js
export default function SearchButton() {
  return (
    <button>
      Search
    </button>
  );
}
```

```js SearchInput.js
export default function SearchInput() {
  return (
    <input
      placeholder="Looking for something?"
    />
  );
}
```

```css
button { display: block; margin-bottom: 10px; }
```

</Sandpack>

<Solution>

`SearchButton` に `onClick` プロパティを追加し、`SearchButton` からブラウザの `<button>` に渡す必要があります。また、`<SearchInput>` に ref を渡すと、実際の `<input>` に転送されて入力されます。最後に、クリックハンドラで `focus` を呼び出し、その ref に格納されている DOM ノードに対して呼び出します。

<Sandpack>

```js App.js
import { useRef } from 'react';
import SearchButton from './SearchButton.js';
import SearchInput from './SearchInput.js';

export default function Page() {
  const inputRef = useRef(null);
  return (
    <>
      <nav>
        <SearchButton onClick={() => {
          inputRef.current.focus();
        }} />
      </nav>
      <SearchInput ref={inputRef} />
    </>
  );
}
```

```js SearchButton.js
export default function SearchButton({ onClick }) {
  return (
    <button onClick={onClick}>
      Search
    </button>
  );
}
```

```js SearchInput.js
import { forwardRef } from 'react';

export default forwardRef(
  function SearchInput(props, ref) {
    return (
      <input
        ref={ref}
        placeholder="Looking for something?"
      />
    );
  }
);
```

```css
button { display: block; margin-bottom: 10px; }
```

</Sandpack>

</Solution>

</Challenges>