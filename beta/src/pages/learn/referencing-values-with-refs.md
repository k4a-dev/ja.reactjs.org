---
title: 'Refを使った値の参照'
---

<Intro>

コンポーネントに何らかの情報を「記憶」させたいが、その情報が[新しいレンダリングのトリガー](/learn/render-and-commit)とならないようにしたい場合、*ref* を使用することができます。

</Intro>

<YouWillLearn>

- コンポーネントに ref を追加する方法
- ref の値を更新する方法
- ref と state の違い
- ref を安全に使用する方法

</YouWillLearn>

## コンポーネントに ref を追加する {/*adding-a-ref-to-your-component*/}

React の `useRef` フックをインポートすることで、コンポーネントに ref を追加することができます：

```js
import { useRef } from 'react';
```

コンポーネント内部で `useRef` フックを呼び出し、参照したい初期値を引数として渡します。例えば、ここでは値 `0` を参照しています：

```js
const ref = useRef(0);
```

`useRef`は以下のようなオブジェクトを返します：

```js
{ 
  current: 0 // The value you passed to useRef
}
```

<Illustration src="/images/docs/illustrations/i_ref.png" alt="An arrow with 'current' written on it stuffed into a pocket with 'ref' written on it." />

ref の現在の値には、`ref.current` プロパティを通してアクセスすることができます。この値は意図的に mutable（可変）であり、読み書きの両方が可能です。これは、React が追跡しない、コンポーネントの秘密のポケットのようなものです。(React の一方通行のデータフローからの「避難ハッチ」になるものです。詳細は後述します！)

以下の例では、ボタンがクリックされるたびに `ref.current` を増加します：

<Sandpack>

```js
import { useRef } from 'react';

export default function Counter() {
  let ref = useRef(0);

  function handleClick() {
    ref.current = ref.current + 1;
    alert('You clicked ' + ref.current + ' times!');
  }

  return (
    <button onClick={handleClick}>
      Click me!
    </button>
  );
}
```

</Sandpack>

今回の ref は数値を参照しますが、[state](/learn/state-a-components-memory) のように、文字列、オブジェクト、あるいは関数など、何でも指定することが可能です。state とは異なり、ref は `current` プロパティを持つプレーンな JavaScript オブジェクトなので、読み書きが可能です。

**増加されるたびにコンポーネントが再レンダリング**はされないことに注意してください。ref は state と同様に再レンダリングされても React によって情報が保持されます。しかし、state を設定はコンポーネントの再レンダリングを引き起こしますが、ref の変更では発生しません。

## 例: ストップウォッチの作成 {/*example-building-a-stopwatch*/}

一つのコンポーネントの中で、ref と state を組み合わせることができます。例えば、ユーザーがボタンを押すことで開始・停止できるストップウォッチを作ってみましょう。ユーザーが「Start」を押してからの経過時間を表示するために、Start ボタンが押された時刻と現在の時刻を記録しておく必要があります。**この情報はレンダリングに使用されるため、state で保持する**ことになります：

```js
const [startTime, setStartTime] = useState(null);
const [now, setNow] = useState(null);
```

ユーザーが「Start」を押したら、[`setInterval`](https://developer.mozilla.org/docs/Web/API/setInterval)を使って 100 ミリ秒ごとに時間を更新するようにします：

<Sandpack>

```js
import { useState } from 'react';

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);

  function handleStart() {
    // Start counting.
    setStartTime(Date.now());
    setNow(Date.now());

    setInterval(() => {
      // Update the current time every 10ms.
      setNow(Date.now());
    }, 10);
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>Time passed: {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>
        Start
      </button>
    </>
  );
}
```

</Sandpack>

「Stop」ボタンが押されたとき、既存のインターバルをキャンセルして state 変数 `now` の更新を停止する必要があります。これは[`clearInterval`](https://developer.mozilla.org/en-US/docs/Web/API/clearInterval)を呼び出すことで可能ですが、ユーザーが Start を押したときに`setInterval`呼び出しによって返されたインターバル ID を与える必要があります。このインターバル ID はどこかに保存しておく必要があります。**インターバル ID はレンダリングに使用されないので、ref に保存**しておくとよいでしょう：

<Sandpack>

```js
import { useState, useRef } from 'react';

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);
  const intervalRef = useRef(null);

  function handleStart() {
    setStartTime(Date.now());
    setNow(Date.now());

    clearInterval(intervalRef.current);
    intervalRef.current = setInterval(() => {
      setNow(Date.now());
    }, 10);
  }

  function handleStop() {
    clearInterval(intervalRef.current);
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>Time passed: {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>
        Start
      </button>
      <button onClick={handleStop}>
        Stop
      </button>
    </>
  );
}
```

</Sandpack>

ある情報がレンダリングに使用される場合、その情報を state で保持します。イベントハンドラが必要とするだけで、情報を変更しても再レンダリングが必要ない場合は、ref を使用する方が効率的かもしれません。

## ref と state の違い {/*differences-between-refs-and-state*/}

おそらく、ref は state よりも**厳しくない**と感じたはずです。例えば、ref を使用すれば state 更新関数を常に使わなくても値を変更することができます。しかし、ほとんどの場合state を使いたくなるはずです。ref は、あまり必要としない「避難ハッチ」なのです。以下は state と ref の比較です：

| ref                                                                         | state                                                                                                                        |
| --------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `useRef(initialValue)` は `{ current: initialValue }` を返す。              | `useState(initialValue)` は state 変数の現在値と state 更新関数 ( `[value, setValue]`) を返す                                |
| 変更しても再レンダリングを引き起こさない。                                  | 変更したら再レンダリングを引き起こす。                                                                                       |
| 「可変」 - レンダリングプロセスの外側で `current` の値の変更や更新が可能。 | 「不変」 - 再レンダリングのキューに入れるために、state 更新関数を使用する必要がある。                                     |
| レンダリング中に `current` の値の読み書きをしてはいけない。                 | state はいつでも読み取り可能。ただし、各レンダリングは変更不可な state の [snapshot](/learn/state-as-a-snapshot) を保持する。|

以下は、state で実装されたカウンターボタンです：

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      You clicked {count} times
    </button>
  );
}
```

</Sandpack>

`count` の値は表示されるため、state 値を使用することは理にかなっています。カウンターの値を `setCount()` で設定すると、React はコンポーネントを再レンダリングし、画面は新しいカウントを反映したものに更新されます。

もしこれを ref で実装しようとすると、React はコンポーネントを再レンダリングしないので、カウントが変化するのを見ることはできません！このボタンをクリックしても **テキストが更新されない** ことを確認してください：

<Sandpack>

```js
import { useRef } from 'react';

export default function Counter() {
  let countRef = useRef(0);

  function handleClick() {
    // This doesn't re-render the component!
    countRef.current = countRef.current + 1;
  }

  return (
    <button onClick={handleClick}>
      You clicked {countRef.current} times
    </button>
  );
}
```

</Sandpack>

これが、レンダリング中に `ref.current` を読むと信頼性のないコードになる理由です。もしレンダリング中の読み取りが必要なら、代わりに state を使ってください。

<DeepDive title="useRefは内部でどのように動作するか？">

`useState` と `useRef` の両方が React で提供されていますが、原理的には `useRef` は `useState` の _上に_ 実装することができます。React の内部では `useRef` はこのように実装されていると想像してください：
```js
// Inside of React
function useRef(initialValue) {
  const [ref, unused] = useState({ current: initialValue });
  return ref;
}
```

最初のレンダリングで、`useRef`は `{ current: initialValue }` を返します。このオブジェクトは React によって保存されるので、次のレンダリングでは同じオブジェクトが返されます。この例では、state 更新関数が使用されていないことに注意してください。なぜなら、`useRef`は常に同じオブジェクトを返す必要があるからです！これは不要です。

これは非常に一般的であるため、React は`useRef` のビルトインバージョンを提供しています。しかし、ref は更新関数を持たない通常の state 変数と考えることができます。オブジェクト指向プログラミングに慣れている人なら ref はインスタンスフィールドを連想させるかもしれませんが、 `this.something` ではなく `somethingRef.current` と記述します。

</DeepDive>

## ref を使うとき {/*when-to-use-refs*/}

一般的に、コンポーネントが React の外に出て、外部の API と通信する必要がある場合に ref を使用します。多くの場合、コンポーネントの外観に影響を与えないブラウザ API を使用します。ここでは、これらのまれな状況のいくつかを紹介します：

- [タイムアウト ID](https://developer.mozilla.org/docs/Web/API/setTimeout)の保存
- [DOM 要素](https://developer.mozilla.org/docs/Web/API/Element)の保存と操作 - これは[次のページ](/learn/manipulating-the-dom-with-refs)で説明します。
- JSX の計算に必要でない他のオブジェクトを格納

コンポーネントが何らかの値を保存する必要があるが、レンダリングロジックに影響を与えない場合は、ref を選択します。

## ref のベストプラクティス {/*best-practices-for-refs*/}

以下の原則に従うことで、コンポーネントをより予測しやすくすることができます。

- **refを避難ハッチとして扱う。**外部システムやブラウザの API と連携する場合、ref は有用です。もし、アプリケーションのロジックやデータフローの多くを ref に依存しているのであれば、そのアプローチを見直した方がよいでしょう。
- **レンダリング中に `ref.current` を読み書きしない。**レンダリング中に何らかの情報が必要な場合は、代わりに [state](/learn/state-a-components-memory) を使用してください。React は `ref.current` がいつ変更されたかを知らないので、レンダリング中にそれを読むだけでもコンポーネントの挙動を予測することが難しくなります(唯一の例外は `if (!ref.current) ref.current = new Thing()` のようなコードで、最初のレンダリング時に一度だけ ref を設定します)。

React の state の制限は、ref には適用されません。例えば、state は[レンダリングごとのスナップショット](/learn/state-as-a-snapshot)と[同期的に更新しない](/learn/queueing-a-series-of-state-updates)という性質を持ちます。しかし、ref の現在値を更新し、変更は直ちに行われます：

```js
ref.current = 5;
console.log(ref.current); // 5
```

これは**ref 自体が通常の JavaScript オブジェクト**であり、そのままの振る舞いを持つからです。

また、ref を扱う際に [Avoiding mutation](/learn/updating-objects-in-state) を気にする必要はないでしょう。更新したオブジェクトがレンダリングに使用されない限り、React はあなたが ref やそのコンテンツに対して何をしようと気にしません。

## Ref と DOM {/*refs-and-the-dom*/}

Ref は任意の値を指定することができます。しかし、ref の最も一般的な使用例は、DOM 要素にアクセスすることです。例えば、プログラム的に入力にフォーカスを当てたい場合などに便利です。JSX の `ref` 属性に `<div ref={myRef}>` のように ref を渡すと、React は対応する DOM 要素を `myRef.current` に配置します。これについては、[Refを使ったDOMの操作](/learn/manipulating-the-dom-with-refs)で詳しく説明しています。

<Recap>

- Ref は、レンダリングに使用しない値を保持するための避難ハッチです。頻繁に必要とするものではありません。
- Ref はプレーンな JavaScript オブジェクトで、`current`という一つのプロパティを持ち、読み書きの両方が可能です。
- `useRef` フックを呼び出すことで、React に ref を渡すように要求することができます。
- state と同様に、ref はコンポーネントの再レンダリングの間、情報を保持することができます。
- state と異なり、ref の `current` 値を設定しても、再レンダリングは行われません。
- レンダリング中に `ref.current` を読み書きしないでください。これはコンポーネントを予測しづらくします。

</Recap>



<Challenges>

### 壊れたチャット入力を修正する {/*fix-a-broken-chat-input*/}

メッセージを入力し、「Send」をクリックします。「Sent!」というアラートが表示されるまで、3 秒の遅延があることに気づくでしょう。この間、「Undo」ボタンが表示されます。この「Undo」ボタンは、「Sent!」メッセージが表示されるのを止めるためのものです。`handleSend`のときに保存されたタイムアウト ID に対して、[`clearTimeout`](https://developer.mozilla.org/en-US/docs/Web/API/clearTimeout)を呼び出すことでそれを実現します。しかし、「Undo」 をクリックしても 「Sent!」メッセージは表示されます。なぜうまくいかないのか、その原因を探って解決してください。

<Hint>

レンダリングのたびにコンポーネントが一から実行され、変数も初期化されるため、`let timeoutID` のような通常の変数は再レンダリング間で「生き残る」ことはできません。タイムアウト ID はどこか別の場所に保存するべきでしょうか？

</Hint>

<Sandpack>

```js
import { useState } from 'react';

export default function Chat() {
  const [text, setText] = useState('');
  const [isSending, setIsSending] = useState(false);
  let timeoutID = null;

  function handleSend() {
    setIsSending(true);
    timeoutID = setTimeout(() => {
      alert('Sent!');
      setIsSending(false);
    }, 3000);
  }

  function handleUndo() {
    setIsSending(false);
    clearTimeout(timeoutID);
  }

  return (
    <>
      <input
        disabled={isSending}
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button
        disabled={isSending}
        onClick={handleSend}>
        {isSending ? 'Sending...' : 'Send'}
      </button>
      {isSending &&
        <button onClick={handleUndo}>
          Undo
        </button>
      }
    </>
  );
}
```

</Sandpack>

<Solution>

コンポーネントが再レンダリングするときは（state を設定するときなど）いつでも、すべてのローカル変数が一から初期化されます。このため、タイムアウト ID をローカル変数 `timeoutID` に保存しても、別のイベントハンドラでそれを「見る」ことはできないのです。代わりに、React がレンダリング時に保持する ref に保存します。

<Sandpack>

```js
import { useState, useRef } from 'react';

export default function Chat() {
  const [text, setText] = useState('');
  const [isSending, setIsSending] = useState(false);
  const timeoutRef = useRef(null);

  function handleSend() {
    setIsSending(true);
    timeoutRef.current = setTimeout(() => {
      alert('Sent!');
      setIsSending(false);
    }, 3000);
  }

  function handleUndo() {
    setIsSending(false);
    clearTimeout(timeoutRef.current);
  }

  return (
    <>
      <input
        disabled={isSending}
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button
        disabled={isSending}
        onClick={handleSend}>
        {isSending ? 'Sending...' : 'Send'}
      </button>
      {isSending &&
        <button onClick={handleUndo}>
          Undo
        </button>
      }
    </>
  );
}
```

</Sandpack>

</Solution>

### コンポーネントの再レンダリングに失敗する不具合を修正 {/*fix-a-component-failing-to-re-render*/}

このボタンは、「On」と「Off」を切り替えて表示することになっています。しかし、常に 「Off」を表示しています。このコードは何が悪いのでしょうか？修正してみましょう：

<Sandpack>

```js
import { useRef } from 'react';

export default function Toggle() {
  const isOnRef = useRef(false);

  return (
    <button onClick={() => {
      isOnRef.current = !isOnRef.current;
    }}>
      {isOnRef.current ? 'On' : 'Off'}
    </button>
  );
}
```

</Sandpack>

<Solution>

この例では，`{isOnRef.current ? 'On' : 'Off'}` のようにレンダリング出力の計算に ref の現在値を使用しています。これは、この情報が ref にあるべきでなく、代わりに state に置かれるべきであったことを示すサインです。修正するには、ref を削除し、代わりに state を使用します：

<Sandpack>

```js
import { useState } from 'react';

export default function Toggle() {
  const [isOn, setIsOn] = useState(false);

  return (
    <button onClick={() => {
      setIsOn(!isOn);
    }}>
      {isOn ? 'On' : 'Off'}
    </button>
  );
}
```

</Sandpack>

</Solution>

### debounce の修正 {/*fix-debouncing*/}

この例では、すべてのボタンのクリックハンドラが [「debounce」](https://redd.one/blog/debounce-vs-throttle) されています。これが何を意味するかは、ボタンの一つを押してみるとわかります。1 秒後にメッセージが表示されることに注意してください。メッセージを待っている間にボタンを押すと、タイマーはリセットされます。なので、同じボタンを何度も速くクリックし続けると、クリックをやめてから 1 秒後までメッセージは表示されません。debounce は、ユーザーが「何かをやめる」まで、何らかのアクションを遅らせてくれます。

この例は動作はしますが、意図した動作とは違います。ボタンが独立していないのです。問題を確認するために、ボタンの一つをクリックし、すぐに別のボタンをクリックしてみてください。遅延の後、両方のボタンのメッセージが表示されることを期待します。しかし、最後のボタンのメッセージだけが表示されます。最初のボタンのメッセージは失われてしまいました。

なぜボタンがお互いに干渉しあっているのでしょうか？問題を発見し、解決してください。

<Hint>

最後のタイムアウト ID 変数は、すべての `DebouncedButton` コンポーネント間で共有されます。これが、あるボタンをクリックすると他のボタンのタイムアウトがリセットされる理由です。各ボタンに別々のタイムアウト ID を格納することは可能でしょうか？

</Hint>

<Sandpack>

```js
import { useState } from 'react';

let timeoutID;

function DebouncedButton({ onClick, children }) {
  return (
    <button onClick={() => {
      clearTimeout(timeoutID);
      timeoutID = setTimeout(() => {
        onClick();
      }, 1000);
    }}>
      {children}
    </button>
  );
}

export default function Dashboard() {
  return (
    <>
      <DebouncedButton
        onClick={() => alert('Spaceship launched!')}
      >
        Launch the spaceship
      </DebouncedButton>
      <DebouncedButton
        onClick={() => alert('Soup boiled!')}
      >
        Boil the soup
      </DebouncedButton>
      <DebouncedButton
        onClick={() => alert('Lullaby sung!')}
      >
        Sing a lullaby
      </DebouncedButton>
    </>
  )
}
```

```css
button { display: block; margin: 10px; }
```

</Sandpack>

<Solution>

`timeoutID` のような変数は、すべてのコンポーネントで共有されます。このため、2 つ目のボタンをクリックすると、1 つ目のボタンの保留中のタイムアウトがリセットされます。これを解決するためにタイムアウトを ref で保持してみます。各ボタンはそれ自身の ref を取得するので、互いに衝突することはありません。2 つのボタンを速くクリックすると、両方のメッセージが表示されることに注目してください。

<Sandpack>

```js
import { useState, useRef } from 'react';

function DebouncedButton({ onClick, children }) {
  const timeoutRef = useRef(null);
  return (
    <button onClick={() => {
      clearTimeout(timeoutRef.current);
      timeoutRef.current = setTimeout(() => {
        onClick();
      }, 1000);
    }}>
      {children}
    </button>
  );
}

export default function Dashboard() {
  return (
    <>
      <DebouncedButton
        onClick={() => alert('Spaceship launched!')}
      >
        Launch the spaceship
      </DebouncedButton>
      <DebouncedButton
        onClick={() => alert('Soup boiled!')}
      >
        Boil the soup
      </DebouncedButton>
      <DebouncedButton
        onClick={() => alert('Lullaby sung!')}
      >
        Sing a lullaby
      </DebouncedButton>
    </>
  )
}
```

```css
button { display: block; margin: 10px; }
```

</Sandpack>

</Solution>

### 最新の state を読む {/*read-the-latest-state*/}

この例では、「送信」を押した後、メッセージが表示されるまでに少し時間がかかります。「hello」と入力し、「Send」を押した後、もう一度素早く入力を編集してみましょう。編集したにもかかわらず、アラートには「hello」（ボタンをクリックした[当時](/learn/state-as-a-snapshot#state-over-time)の state 値）が表示されたままになっています。

通常、この動作はアプリに適しています。しかし、非同期コードで最新バージョンの state を読み取りたい場合もあるでしょう。クリックしたときの入力テキストではなく、*現在の*入力テキストをアラートに表示させる方法を考えてみましょう。
<Sandpack>

```js
import { useState, useRef } from 'react';

export default function Chat() {
  const [text, setText] = useState('');

  function handleSend() {
    setTimeout(() => {
      alert('Sending: ' + text);
    }, 3000);
  }

  return (
    <>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button
        onClick={handleSend}>
        Send
      </button>
    </>
  );
}
```

</Sandpack>

<Solution>

State は[スナップショットのように](/learn/state-as-a-snapshot)動作するので、タイムアウトなどの非同期処理から最新の state を読み取ることはできません。しかし、最新の入力テキストを ref に保持することはできます。ref は不変なので、いつでも `current` プロパティを読み出すことができます。現在のテキストはレンダリングにも使用されるので、この例では（レンダリングのための）state 変数と（タイムアウトでそれを読むための）ref の _両方_ が必要です。現在の ref の値は手動で更新する必要があります。

<Sandpack>

```js
import { useState, useRef } from 'react';

export default function Chat() {
  const [text, setText] = useState('');
  const textRef = useRef(text);

  function handleChange(e) {
    setText(e.target.value);
    textRef.current = e.target.value;
  }

  function handleSend() {
    setTimeout(() => {
      alert('Sending: ' + textRef.current);
    }, 3000);
  }

  return (
    <>
      <input
        value={text}
        onChange={handleChange}
      />
      <button
        onClick={handleSend}>
        Send
      </button>
    </>
  );
}
```

</Sandpack>

</Solution>

</Challenges>
