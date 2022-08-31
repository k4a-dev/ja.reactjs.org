---
title: 'Effect による同期'
---

<Intro>

外部システムとの同期が必要なコンポーネントも存在します。React の state に基づいて React 以外のコンポーネントを制御する、サーバー接続を設定する、コンポーネントが画面に表示されたときに分析ログを送信する、などが例に挙げられます。*Effect* を使うと、レンダリング後にコードを実行してコンポーネントを React 以外のシステムと同期させることができます。

</Intro>

<YouWillLearn>

- Effect とは何か
- イベントと Effect の違い
- コンポーネント内で Effect を宣言する方法
- Effect の不要な再実行を防ぐ方法
- 開発環境で Effect が 2 回実行される理由とその修正方法

</YouWillLearn>

## エフェクトとは何か、イベントと何が違うのか？ {/*what-are-effects-and-how-are-they-different-from-events*/}

Effects に入る前に、React コンポーネント内の 2 種類のロジックについて知っておく必要があります。

- **コードのレンダリング**（[UI の記述](/learn/describing-the-ui)で紹介）は、コンポーネントの最上位レベルに存在します。props と state を受け取り、それらを変換し、画面に表示したい JSX を返します。[コードのレンダリングは純粋でなければなりません。](/learn/keeping-components-pure)数式のように結果を _計算_ するだけで、他のことは何もしてはいけません。

- **イベントハンドラ**（[インタラクティビティの追加](/learn/adding-interactivity)で紹介）は、コンポーネント内にネストされた関数で、単に計算するのではなく何かを*行う*ものです。イベントハンドラは、入力フィールドを更新したり、商品を購入するために HTTP POST リクエストを送信したり、ユーザーを他の画面に移動させたりします。イベントハンドラには、プログラムの状態を変更する [「副作用」](<https://en.wikipedia.org/wiki/Side_effect_(computer_science)>) があり、（ボタンのクリックや文字入力などの）特定のユーザーアクションによって引き起こされます。

これだけでは十分でない場合もあります。例えば、 `ChatRoom` コンポーネントが画面に表示されているときは常にチャットサーバに接続しなければならないとします。サーバーへの接続は純粋な計算ではないので（副作用）、レンダリング中に発生することはありません。しかし、クリックのような特定のイベントによって `ChatRoom` が表示されるわけではありません。

**_Effect_ では、特定のイベントではなく、レンダリングそのものに起因する副作用を指定することができます**。チャットでメッセージを送ることは、ユーザーが特定のボタンをクリックすることによって直接引き起こされるため、*イベント*です。しかし、サーバー接続の設定は、どのインタラクションによってコンポーネントが表示されたかに関係なく発生する必要があるため、*Effect* と呼ばれます。Effect は、画面が更新された後の[レンダリング処理](/learn/render-and-commit)の終了時に実行されます。これは、React コンポーネントを何らかの外部システム（ネットワークやサードパーティライブラリなど）と同期させるのによいタイミングです。

<Note>

本文中では、大文字で書かれた「Effect」は上記の React 固有の定義、すなわちレンダリングによって引き起こされる副作用を指します。より広いプログラミングの概念に言及する場合は、「副作用(side effect)」と記述します。

</Note>


## Effect は必要ないかもしれない {/*you-might-not-need-an-effect*/}

**焦って Effect をコンポーネントに追加する必要はありません。**Effect は通常、React のコードから「抜け出して」、何らかの*外部システム*と同期するために使用されることを覚えておいてください。これには、ブラウザ API、サードパーティウィジェット、ネットワークなどが含まれます。あなたの Effect が、他の state に基づいて他の state を調整するだけであれば、[Effect は必要ないかもしれません。](/learn/you-might-not-need-an-effect)

## Effect の書き方 {/*how-to-write-an-effect*/}

Effect を書くには、以下の 3 つのステップを踏みます：

1. **Effect を宣言する。** デフォルトでは、レンダリングのたびに Effect が実行されます。
2. **Effect の依存関係を指定する。** ほとんどの Effect は、レンダリングのたびに実行するのではなく、必要なときにのみ再実行する必要があります。例えば、フェードインアニメーションは、コンポーネントが表示されたときのみトリガーされます。チャットルームへの接続と切断は、コンポーネントが表示されたり消えたりしたとき、またはチャットルームが変更されたときにのみ発生するようにすべきです。このような制御を行うに、*依存関係*を指定する方法を学びます。
3. **必要な場合はクリーンアップを行う。** 場合によっては Effect に対して、停止、取り消し、クリーンアップの方法を用意する必要があります。例えば、「connect」は「disconnect」を、「subscribe」は「unsubscribe」を、そして「fetch」は「cancel」または「ignore」が必要です。これらを行うために、*クリーンアップ関数*を返す方法を学びます。

それでは、それぞれのステップを詳しく見ていきましょう。

### ステップ 1：Effect の宣言 {/*step-1-declare-an-effect*/}

コンポーネントで Effect を宣言するには、React の[`useEffect` フック](/api/useeffect)をインポートします：

```js
import { useEffect } from 'react';
```

フックをコンポーネントのトップレベルで呼び出し、Effect の中に何らかのコードを入れます：

```js {2-4}
function MyComponent() {
  useEffect(() => {
    // Code here will run after *every* render
  });
  return <div />;
}
```

コンポーネントがレンダリングするたびに React は画面を更新し、その後 `useEffect` 内のコードを実行します。言い換えると、**`useEffect` は、レンダリングが画面に反映されるまで、コードの実行を「遅らせる」**のです。

それでは、Effect を使って外部システムと同期する方法を見てみましょう。`<VideoPlayer>` という React コンポーネントがあるとします。このコンポーネントに `isPlaying` props を渡すことで、再生中か一時停止中かを制御できると良さそうです：

```js
<VideoPlayer isPlaying={isPlaying} />;
```

カスタムの `VideoPlayer` コンポーネントはブラウザに組み込まれた [`<video>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/video)タグを描画します：

```js
function VideoPlayer({ src, isPlaying }) {
  // TODO: do something with isPlaying
  return <video src={src} />;
}
```

しかし、ブラウザの `<video>` タグは `isPlaying` プロパティを持ちません。これを制御する唯一の方法は、DOM 要素の [`play()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/play) と [`pause()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/pause) メソッドを手動で呼び出すことです。**ビデオが現在再生されているべきかどうかを示す `isPlaying` props の値を、 `play()` や `pause()` といった命令型の呼び出しと同期させる必要があります。**

まず、`<video>` という DOM ノードへの [ref](/learn/manipulating-the-dom-with-refs) を取得する必要があります。

レンダリング中に `play()` や `pause()` を呼び出そうと思うかもしれませんが、それは正しくありません：

<Sandpack>

```js
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  if (isPlaying) {
    ref.current.play();  // Calling these while rendering isn't allowed.
  } else {
    ref.current.pause(); // Also, this crashes.
  }

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  return (
    <>
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? 'Pause' : 'Play'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

```css
button { display: block; margin-bottom: 20px; }
video { width: 250px; }
```

</Sandpack>

このコードが正しくない理由は、レンダリング中に DOM ノードに対して何かを行おうとするからです。React では、[レンダリングは JSX の純粋な計算](/learn/keeping-components-pure) であるべきで、DOM を修正するような副作用を含んではいけません。

さらに、`VideoPlayer` が初めて呼び出されたとき、その DOM はまだ存在しません！なぜなら、React は JSX を返した後でないとどの DOM を作成すればよいのか分からないからです。

ここでの解決策は、 **副作用を `useEffect` でラップして、レンダリング計算から除外する** ことです：

```js {6,12}
import { useEffect, useRef } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}
```

DOM の更新を Effect で包むことで、React に先に画面を更新させることができます。その後、Effect が実行されます。

`VideoPlayer` コンポーネントが（初回または再度）レンダリングされると、次の事象が発生します。まず、React は画面を更新し、`<video>` タグが正しい props で DOM にあることを確認します。次に、React は Effect を実行します。最後に、Effect は `isPlaying` props の値に応じて `play()` または `pause()` を呼び出します。

再生や一時停止を何度も行い、ビデオプレーヤーが `isPlaying` の値と同期していることを確認してください：

<Sandpack>

```js
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  return (
    <>
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? 'Pause' : 'Play'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

```css
button { display: block; margin-bottom: 20px; }
video { width: 250px; }
```

</Sandpack>

この例では、React の state に同期させた「外部システム」は、ブラウザのメディア API でした。同様のアプローチで、レガシーな非 React コード（jQuery プラグインなど）を宣言型の React コンポーネントでラップすることができます。

ビデオプレーヤーの制御は、実際にはもっと複雑であることに注意してください。`play()` の呼び出しに失敗したり、ユーザーがブラウザの組み込みコントロールを使って再生や一時停止を行ったり、などが考えられます。この例は非常に単純化されており、不完全なものです。

<Gotcha>

デフォルトでは、エフェクトは _毎_ レンダリング後に実行されます。そのため、次のようなコードは**無限ループを発生**させます：

```js
const [count, setCount] = useState(0);
useEffect(() => {
  setCount(count + 1);
});
```

エフェクトはレンダリングの *結果* として実行されます。state の更新はレンダリングを*引き起こします*。エフェクトの中ですぐに state を設定することは、自分のコンセントに自分のプラグを差し込むようなものです。Effect が実行され state が設定されると、再レンダリングが行われ、Effect が実行され、再び state が設定されると、また再レンダリングが行われ、その後は想像のとおりです。

Effect は通常、コンポーネントを*外部*のシステムとの同期にしようします。もし、外部システムがなく、ある state を他の state に基づいて調整したいだけであれば、[エフェクトは必要ないかもしれません。](/learn/you-might-need-an-effect)

</Gotcha>

### ステップ 2：Effect の依存関係を指定する {/*step-2-specify-the-effect-dependencies*/}

デフォルトでは、Effect は *毎* レンダリング後に実行されます。多くの場合、これは **望ましい動作ではありません：** 

- 動作が遅い場合。外部システムとの同期は常に即座に行われるとは限らないので、必要でない限り同期をスキップしたいでしょう。例えば、キーストロークのたびにチャットサーバーに再接続するのは好ましくありません。
- 間違っている場合。例えば、キー入力のたびにコンポーネントのフェードイン・アニメーションを起動させるのは好ましくありません。アニメーションは、コンポーネントが初めて表示されたときに一度だけ再生されるべきです。

この問題を説明するために、前の例に対して `console.log` をいくつか呼び出し、親コンポーネントの state を更新するテキスト入力を行ってみましょう。タイピングをすると、Effect が再実行されることに注意してください：

<Sandpack>

```js
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      console.log('Calling video.play()');
      ref.current.play();
    } else {
      console.log('Calling video.pause()');
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? 'Pause' : 'Play'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

```css
input, button { display: block; margin-bottom: 20px; }
video { width: 250px; }
```

</Sandpack>

`UseEffect` 呼び出しの第二引数に*依存*配列を指定することで、React に **不必要な Effect の再実行を省く** ように指示することができます。まず、上記の例の 14 行目に空の `[]` 配列を追加してください：

```js {3}
  useEffect(() => {
    // ...
  }, []);
```

`React Hook useEffect has a missing dependency: 'isPlaying'` というエラーが表示されるはずです：

<Sandpack>

```js
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      console.log('Calling video.play()');
      ref.current.play();
    } else {
      console.log('Calling video.pause()');
      ref.current.pause();
    }
  }, []); // This causes an error

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? 'Pause' : 'Play'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

```css
input, button { display: block; margin-bottom: 20px; }
video { width: 250px; }
```

</Sandpack>

Effect 内のコードは、条件判断を `isPlaying` props に *依存* しているにもかかわらず、この依存関係が明示的に宣言されていないことが問題です。この問題を解決するには、`isPlaying` を依存配列に追加します：

```js {2,7}
  useEffect(() => {
    if (isPlaying) { // It's used here...
      // ...
    } else {
      // ...
    }
  }, [isPlaying]); // ...so it must be declared here!
```

これですべての依存関係が宣言されたので、エラーは発生しません。依存配列に `[isPlaying]` の指定は、 React に対して `isPlaying` が前回のレンダリングのときと同じであれば Effect の再実行をスキップするように指示します。この変更により、input にタイプしても Effect は再実行されず、Play/Pause を押したときのみ再実行されるようになりました：

<Sandpack>

```js
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      console.log('Calling video.play()');
      ref.current.play();
    } else {
      console.log('Calling video.pause()');
      ref.current.pause();
    }
  }, [isPlaying]);

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? 'Pause' : 'Play'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

```css
input, button { display: block; margin-bottom: 20px; }
video { width: 250px; }
```

</Sandpack>

依存配列には、複数の依存関係を含めることができます。React は、指定した依存関係の*すべて*が前回のレンダリング時とまったく同じ値である場合にのみ、Effect の再実行をスキップします。React は依存関係の値を [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) という比較方法を用いて比較します。詳しくは [`useEffect` の API reference](/apis/useeffect#reference) を参照してください。

**依存関係を「選択」できないことに注意してください。** 指定した依存関係が、Effect 内のコードに基づいて React が期待するものと一致しない場合、lint エラーが発生します。これは、コードの多くのバグを発見するのに役立ちます。もし、Effect がある値を使用しているが変更されたときには Effect を再実行したくない場合は、その依存関係を「必要としない」ように*Effect のコード自体を編集*する必要があります。これについては、[Specifying the Effect Dependencies](/learn/specifying-effect-dependencies) で詳しく説明されています。

<Gotcha>

依存関係配列がない場合と、依存関係配列が空 `[]` である場合では、動作が大きく異なります：

```js {3,7,11}
useEffect(() => {
  // This runs after every render
});

useEffect(() => {
  // This runs only on mount (when the component appears)
}, []);

useEffect(() => {
  // This runs on mount *and also* if either a or b have changed since the last render
}, [a, b]);
```

「マウント」の意味については、次のステップで詳しく見ていきましょう。

</Gotcha>

<DeepDive title="依存配列からrefが省略されているのはなぜか？">

この Effect は `ref` と `isPlaying` の両方を使用していますが、依存関係として宣言されているのは `isPlaying` だけです：

```js {9}
function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);
  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  }, [isPlaying]);
```

これは、`ref`オブジェクトが*不変の同一性*を持っているためです。React は、すべてのレンダリングで同じ `useRef` を呼び出すと、[常に同じオブジェクトを得る](/apis/useref#returns)ことを保証しています。オブジェクトが変更されることは決して無いため、ref によって Effect が再実行されることはありません。したがって、これを含めるかどうかは重要ではありません。含めても問題ありません：

```js {9}
function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);
  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  }, [isPlaying, ref]);
```

`useState` が返す [更新関数](/apis/usestate#setstate) も不変の同一性を持っているので、依存関係から省かれたものをよく見かけます。もし、linter がエラーにならずに依存関係を省略できるのであれば、それは安全な方法です。

常に不変である依存関係を省略できるのは、リンターがそのオブジェクトが安定していることを「確認」できるときだけです。例えば、`ref` が親コンポーネントから渡された場合、依存配列でそれを指定する必要があります。親コンポーネントが常に同じ ref を渡すのか、それとも条件付きで複数の ref のうちの 1 つを渡すのかが分からないので、これは良いことです。この場合、Effect はどの ref が渡されるかに依存することになります。

</DeepDive>

### ステップ 3：必要な場合にクリーンアップを追加する {/*step-3-add-cleanup-if-needed*/}

別の例を考えてみましょう。あなたは `ChatRoom` コンポーネントを書いていて、表示されたときにチャットサーバーに接続する必要があるとします。`connect()` と `disconnect()` メソッドを持つオブジェクトを返す `createConnection()` API が与えられています。ユーザーに表示されている間、コンポーネントの接続を維持するにはどうしたらよいでしょうか？

まず、Effect ロジックを書きます：

```js
useEffect(() => {
  const connection = createConnection();
  connection.connect();
});
```

再レンダリングのたびにチャットに接続するのは遅延が発生するため、依存配列を追加します：

```js {4}
useEffect(() => {
  const connection = createConnection();
  connection.connect();
}, []);
```

**Effect 内のコードは props や state を使用しないので、依存配列は `[]` (空) となります。これは、コンポーネントが「マウント」されたとき、つまり初めて画面に表示されたときにのみ、このコードを実行するように React に指示します。**

では、このコードを実行してみましょう：

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
  }, []);
  return <h1>Welcome to the chat!</h1>;
}
```

```js chat.js
export function createConnection() {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('Connecting...');
    },
    disconnect() {
      console.log('Disconnected.');
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
```

</Sandpack>

この Effect はマウント時にしか実行されないので、コンソールに `"Connecting..."` が一度だけ表示されると思うかもしれません。**しかし、コンソールを確認すると、`"Connecting..."`と 2 回出力されます。なぜ、このようなことが起こるのでしょうか？**

`ChatRoom` コンポーネントが、多くの異なる画面を持つ大きなアプリの一部であると想像してください。ユーザは `ChatRoom` ページから旅を始めます。コンポーネントがマウントされ、`connection.connect()`が呼び出されます。次に、ユーザが別の画面、例えば、Settings ページに移動したとします。このとき、`ChatRoom` コンポーネントはアンマウントされます。最後に、ユーザが Back をクリックすると `ChatRoom` が再びマウントされます。ここで 2 つ目の接続をセットアップすることになりますが、最初の接続は破棄されていません！ユーザーがアプリをナビゲートすると、接続が積み重なることになります。

このようなバグは、大規模な手動テストを行わない限り、簡単に見逃してしまうものです。このようなバグを素早く発見するために、React の開発環境では、すべてのコンポーネントを最初にマウントした直後に一度だけ再マウントします。**`"Connecting..."` のログを 2 回見ることで、「コンポーネントがアンマウントされたときに接続を閉じない」という本当の問題に気づくことができます。**

この問題を解決するには、Effect から*クリーンアップ関数*を返します：

```js {4-6}
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []);
```

React は Effect が再実行される前に毎回クリーンアップ関数を呼び出し、最後にコンポーネントがアンマウント（削除）されるときにもう一度呼び出します。クリーンアップ関数を実装するとどうなるか見てみましょう：

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, []);
  return <h1>Welcome to the chat!</h1>;
}
```

```js chat.js
export function createConnection() {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('Connecting...');
    },
    disconnect() {
      console.log('Disconnected.');
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
```

</Sandpack>

これで、開発環境でコンソールログが 3 つ表示されるようになりました。

1. `"Connecting..."`
2. `"Disconnected."`
3. `"Connecting..."`

**これは開発環境の正しい動作です。** コンポーネントを再マウントすることで、React は離脱と復帰のナビゲーションがコードを壊さないことを確認します。切断して再び接続するという動作は、まさに望ましいことです! クリーンアップをうまく実装すれば、Effect を一度実行するのと、一度実行してクリーンアップし、再度実行するのとで、ユーザから見える違いはないはずです。接続/切断のペアが余分にあるのは、開発中に React があなたのコードのバグを探っているためです。これは正常なことであり、消そうとするべきではありません。

**本番環境では、`"Connecting..."` が一度だけ表示されます。** コンポーネントの再マウントは、クリーンアップが必要な Effect を見つけるために開発中にのみ発生します。[Strict Mode](/apis/strictmode) をオフにすると開発時の動作を省略することができますが、オンにしておくことをお勧めします。この仕組みによって、上記のような多くのバグを発見することができます。

## 開発環境に Effect が 2 回発生した場合の対処法は？ {/*how-to-handle-the-effect-firing-twice-in-development*/}

React は開発環境に意図的にコンポーネントを再マウントして、ここまでに示したようなバグを見つけやすくしています。**正しい質問は「Effect を 1 度だけ実行する方法」ではなく、「再マウント後に Effect が動作するように修正する方法」です。**

通常、その答えは、クリーンアップ関数を実装することです。 クリーンアップ関数は、Effect が行っていたことを止めるか、元に戻すべきです。経験則では、本番環境のように Effect が一度だけ実行される場合と、開発環境のように _Effect → クリーンアップ → Effect_ と一連で実行される場合をユーザが区別することはできません。

あなたが書く Effect のほとんどは、以下の共通パターンのいずれかに当てはまるはずです。

### 非 React ウィジェットの制御 {/*controlling-non-react-widgets*/}

React に書かれていない UI ウィジェットを追加する必要がある場合があります。例えば、ページに地図コンポーネントを追加するとします。`setZoomLevel()` メソッドがあり、React コード内の state 変数 `zoomLevel` と同期してズームレベルを維持したいとします。Effect は以下のようになります：

```js
useEffect(() => {
  const map = mapRef.current;
  map.setZoomLevel(zoomLevel);
}, [zoomLevel]);
```

この場合、クリーンアップの必要がないことに注意してください。開発時には、React は Effect を 2 回呼び出しますが、同じ値で 2 回 `setZoomLevel` を呼び出しても何も起こらないので、問題にはなりません。若干遅くなるかもしれませんが、再マウントは開発専用で本番では発生しないので、問題にはなりません。

API によっては、2 回続けて呼び出すことができないものもあります。例えば、組み込みの [`<dialog>`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement) 要素の [`showModal`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/showModal) メソッドは、2 回呼ぶと throw されます。クリーンアップ関数を実装して、ダイアログを閉じるようにします：

```js {4}
useEffect(() => {
  const dialog = dialogRef.current;
  dialog.showModal();
  return () => dialog.close();
}, []);
```

開発環境の Effect では、`showModal()` を呼び出した後、すぐに `close()` を行い、再び `showModal()` を呼び出すことになります。ユーザーから見える動作としては、本番環境で`showModal()` を一度呼び出すのと同じです。

### イベントを購読する {/*subscribing-to-events*/}

Effect が何かを購読している場合、クリーンアップ関数が購読を解除する必要があります：

```js {6}
useEffect(() => {
  function handleScroll(e) {
    console.log(e.clientX, e.clientY);
  }
  window.addEventListener('scroll', handleScroll);
  return () => window.removeEventListener('scroll', handleScroll);
}, []);
```

開発環境では、Effect は `addEventListener()` を呼び出し、すぐに `removeEventListener()` を呼び出し、そして同じハンドラで再び `addEventListener()` を呼び出すことになります。つまり、一度にアクティブなサブスクリプションは 1 つだけとなります。ユーザーから見える動作としては、本番環境で `addEventListener()` を一度呼び出すのと同じです。

### アニメーションのトリガー {/*triggering-animations*/}

Effect が何かをアニメーションさせる場合、クリーンアップ関数はアニメーションを初期値に戻す必要があります：

```js {4-6}
useEffect(() => {
  const node = ref.current;
  node.style.opacity = 1; // Trigger the animation
  return () => {
    node.style.opacity = 0; // Reset to the initial value
  };
}, []);
```

開発段階では、opacity は `1` に設定され、次に `0` に設定され、そして再び `1` に設定されます。ユーザーから見える動作としては、直接 `1` に設定するのと同じになるはずです。トゥイーンをサポートするサードパーティのアニメーションライブラリを使用している場合、クリーンアップ関数はトゥイーンのタイムラインを初期状態にリセットする必要があります。

### データの取得 {/*fetching-data*/}

Effect が何かを取得する場合、クリーンアップ関数は、[取得を中止する](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) か、その結果を無視する必要があります：

```js {2,6,13-15}
useEffect(() => {
  let ignore = false;

  async function startFetching() {
    const json = await fetchTodos(userId);
    if (!ignore) {
      setTodos(json);
    }
  }

  startFetching();

  return () => {
    ignore = true;
  };
}, [userId]);
```

ネットワークリクエストを「取り消し」することはできませんが、クリーンアップ関数は、*もう関係ない*取得がアプリケーションに影響を与え続けないことを保証しなければいけません。例えば、 `userId` が `'Alice'` から `'Bob'` に変わった場合、 `'Alice'` のレスポンスが `'Bob'` の後に来ても無視されるようにクリーンアップを行います。

**開発環境では、Network タブに 2 つの取得が表示されます。** しかし何も問題はありません。上記のアプローチでは、最初の Effect はすぐにクリーンアップされるので、 `ignore` 変数のコピーには `true` がセットされます。したがって、余分なリクエストがあっても、 `if (!ignore)` チェックにより state には影響を与えません。

**本番環境では、リクエストは 1 つだけです。** もし開発環境に 2 つ目のリクエストが気になるようでしたら、リクエストを重複排除し、コンポーネント間でレスポンスをキャッシュするソリューションを使用するのがベストな方法です：

```js
function TodoList() {
  const todos = useSomeDataLibrary(`/api/user/${userId}/todos`);
  // ...
```

これは、開発体験を向上させるだけでなく、アプリケーションをより高速に感じさせることができます。例えば、Back ボタンを押したユーザーは、データがキャッシュされるため、再度ロードされるのを待つ必要がありません。このようなキャッシュを自分で構築するか、Effects の手動取得に代わる既存の多くの選択肢の一つを使用することができます。

<DeepDive title="Effect でデータを取得するための良い方法は？">

Effect 内で `fetch` コールを書くことは、特に完全なクライアントサイドのアプリケーションでは、[データを取得する一般的な方法](https://www.robinwieruch.de/react-hooks-fetch-data/) です。しかし、これは非常に手動的なアプローチであり、大きな欠点があります：

- **Effect はサーバーでは動作しません。**これは、サーバーでレンダリングされた最初の HTML には、データのない読み込み状態だけが含まれることを意味します。クライアントコンピュータは、すべての JavaScript をダウンロードしてアプリをレンダリングし、今度はデータをロードする必要があることを発見しなければなりません。これは効率的ではありません。
- **Effect で直接取得することは、「ネットワークウォーターフォール」が簡単に発生させます。**親コンポーネントをレンダリングし、データを取得し、子コンポーネントをレンダリングし、子コンポーネントがデータの取得を開始します。ネットワークがあまり高速でない場合、これはすべてのデータを並行して取得するよりもかなり遅くなります。
- **Effect で直接取得することは、通常、データのプリロードやキャッシュを行わないことを意味します。**例えば、コンポーネントがアンマウントした後、再度マウントすると、再度データを取得する必要があります。
- **人間工学的に優れているとは言えません。**[レースコンディション](https://maxrozen.com/race-conditions-fetching-data-react-with-useeffect)のようなバグに悩まされない方法で`fetch`コールを書くには、かなり多くの定型的なコードが必要です。

このデメリットのリストは、React に特有のものではありません。どのようなライブラリでもマウントでデータを取得する際に適用されます。ルーティングと同様に、データ取得はうまくやるのは簡単なことではないので、以下のアプローチを推奨します：

- **[フレームワーク](/learn/start-a-new-react-project#building-with-a-full-featured-framework)を使用している場合、その組み込みデータ取得機能**を使用します。モダンな React フレームワークは、効率的で上記の落とし穴に苦しまない統合データ取得機構を備えています。
- **そうでない場合は、クライアントサイドキャッシュの使用または構築を検討**してください。有名なオープンソースソリューションには、[React Query](https://react-query.tanstack.com/), [useSWR](https://swr.vercel.app/), および [React Router 6.4+](https://beta.reactrouter.com/en/dev/getting-started/data) があります。Effect を利用しつつ、リクエストの重複排除、レスポンスのキャッシュ、ネットワークのウォーターフォールの回避（データのプリロードやデータ要件のルートへの引き上げ）のためのロジックを追加することができます。

もし、これらの方法が適切でない場合は、Effects で直接データを取得してもよいでしょう。

</DeepDive>

### アナリティクスを送信する {/*sending-analytics*/}

ページ訪問時にアナリティクスイベントを送信する以下のコードを考えてみましょう：

```js
useEffect(() => {
  logVisit(url); // Sends a POST request
}, [url]);
```

開発環境では、すべての URL に対して `logVisit` が 二度呼び出されます。以前の例と同様に、**このコードはそのままにしておくことをお勧めします。** 一度だけ実行しても、二度実行しても、**ユーザーから見えるような動作の差**はありません。実用的な観点からは、 `logVisit` は開発時には何もすべきではありません。なぜなら、開発マシンからのログが本番環境のメトリクスに影響を与えないようにしたいからです。コンポーネントはファイルを保存するたびに再マウントされるので、開発中はとにかく余分な訪問を送ることになります。


**本番環境では、重複する訪問ログはありません。**

送信している分析イベントをデバッグするには、アプリをステージング環境（本番モードの動作）にデプロイするか、一時的に [Strict Mode](/api/strictmode)を無効にして開発環境専用の再マウントチェックをオプトアウトすることができます。また、Effect の代わりにルート変更イベントハンドラからアナリティクスを送信することもできます。より正確な分析をしたければ、どのコンポーネントがビューポートにありどれだけの時間表示されたままかを追跡するのに、[intersection observers](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API) が役立ちます。

### 非 Effect：アプリケーションの初期化 {/*not-an-effect-initializing-the-application*/}

アプリケーションの起動時に一度だけ実行されるべきロジックがある場合もあります。そのようなロジックはコンポーネントの外側に置くことができます：

```js {2-3}
if (typeof window !== 'undefined') { // Check if we're running in the browser.
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

こうすることで、このロジックはブラウザがページを読み込んだ後に一度だけ実行されることが保証されます。

### 非 Effect: 製品の購入 {/*not-an-effect-buying-a-product*/}

クリーンアップ関数を記述しても、Effect が 2 回実行されることでユーザーから見える結果を防げない場合もあります。例えば、商品を購入するような POST リクエストを送る Effect があるとします：

```js {2-3}
useEffect(() => {
  // 🔴 Wrong: This Effect fires twice in development, exposing a problem in the code.
  fetch('/api/buy', { method: 'POST' });
}, []);
```

商品を 2 回買いたいとは思わないでしょう。しかし、これは Effect にこのロジックを入れるべきではない理由でもあります。もし、ユーザーが別のページに移動して、「戻る」を押したらどうでしょうか？また Effect が実行されてしまいます。ユーザーがページを*訪れたとき*に商品を購入するのではなく、ユーザーが購入ボタンを*クリック*したときに商品を購入したいのです。

購入はレンダリングによるものではなく、特定のインタラクションによるものです。インタラクション（クリック）は一度だけ発生するため、一度だけ実行されます。**Effect を削除して、`/api/buy`リクエストを Buy ボタンのイベントハンドラーに移動します：**

```js {2-3}
  function handleClick() {
    // ✅ Buying is an event because it is caused by a particular interaction.
    fetch('/api/buy', { method: 'POST' });
  }
```

**ここまでの情報は、再マウントがアプリケーションのロジックを壊す場合、通常は既存のバグを発見したということを示しています。** ユーザーの観点では、「ページを訪問すること」は「ページを訪問してリンクをクリックし戻るを押すこと」と異なるべきではありません。React は、開発中にコンポーネントを一度再マウントすることで、コンポーネントがこの原則を破らないことを確認します。

## 全てをまとめる {/*putting-it-all-together*/}

以下の playground は、Effect が実際にどのように動作するのかを「感じる」のに役立ちます。

この例では、[`setTimeout`](https://developer.mozilla.org/en-US/docs/Web/API/setTimeout) を使って、入力されたテキストを含むコンソールログが、Effect の実行後 3 秒間に表示されるようにスケジュールしています。クリーンアップ関数は、保留中のタイムアウトをキャンセルします。まず、"Mount the component"を押してください：

<Sandpack>

```js
import { useState, useEffect } from 'react';

function Playground() {
  const [text, setText] = useState('a');

  useEffect(() => {
    function onTimeout() {
      console.log('⏰ ' + text);
    }

    console.log('🔵 Schedule "' + text + '" log');
    const timeoutId = setTimeout(onTimeout, 3000);

    return () => {
      console.log('🟡 Cancel "' + text + '" log');
      clearTimeout(timeoutId);
    };
  }, [text]);

  return (
    <>
      <label>
        What to log:{' '}
        <input
          value={text}
          onChange={e => setText(e.target.value)}
        />
      </label>
      <h1>{text}</h1>
    </>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>
        {show ? 'Unmount' : 'Mount'} the component
      </button>
      {show && <hr />}
      {show && <Playground />}
    </>
  );
}
```

</Sandpack>

まず、`Schedule "a" log`, `Cancel "a" log`, そして再び `Schedule "a" log` の 3 つのログを見ることができます。3 秒後に`a`というログも表示されます。このページで学んだように、余分なスケジュールとキャンセルのペアは、 **React が開発中に一度コンポーネントを再マウントして、クリーンアップがうまく実装されているかどうかを確認するため**に存在します。

今度は入力を編集して `abc` とします。もしあなたが十分な速さでそれを行えば、`Schedule "ab" log` の直後に `Cancel "ab" log` と `Schedule "abc" log` が表示されるはずです。**React は常に次のレンダリングの Effect の前に、前のレンダリングの Effect をクリーンアップします。**何度か入力を編集し、コンソールを見て、Effect がどのようにクリーンアップされるかを感じてみてください。

入力欄に何か入力し、すぐに 「Unmount the component」を押してください。**アンマウントすることで、最後のレンダリングの Effect をクリーンアップしていることに注目してください。**この例では、最後のタイムアウトが発生する前にアンマウントすることで、その Effect がクリーンアップされます。

最後に、上記のコンポーネントを編集し、**クリーンアップ関数をコメントアウト**して、タイムアウトがキャンセルされないようにします。`abcde` を高速でタイプしてみてください。3 秒後に何が起こると思いますか？タイムアウト中に `console.log(text)` が最新の `text` を表示し、 `abcde` のログを 5 つ生成するでしょうか? あなたの勘を確かめるために試してみてください!

3 秒後には、5 つの `abcde` のログではなく、一連のログ (`a`, `ab`, `abc`, `abcd`,`abcde`) が表示されるはずです。**各 Effect は対応するレンダリングの `text` 値を「キャプチャ」します**。 `text = 'ab'` のレンダリングによる Effect は、常に `'ab'` を見ます。言い換えると、各レンダリングの Effect は、互いに分離されています。この仕組みに興味がある方は、[クロージャ](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures) を読んでみてください。

<DeepDive title="各レンダリングは独自のEffectを持つ">

`useEffect` は、レンダリング出力に動作の一部を「付属」させることだと考えることができます。以下の Effect を考えてみましょう：

```js
export default function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return <h1>Welcome to {roomId}!</h1>;
}
```

ユーザーがアプリ内を移動する際に何が起こるか、具体的に見ていきましょう。

#### 初期レンダリング {/*initial-render*/}

ユーザーは `<ChatRoom roomId="general" />` を訪問します。ここで、`roomId` を `'general'` に[置き換えて](/learn/state-as-a-snapshot#rendering-takes-a-snapshot-in-time)みましょう：

```js
  // JSX for the first render (roomId = "general")
  return <h1>Welcome to general!</h1>;
```

**Effect *も*レンダリング出力の一部です。** 最初のレンダリングの Effect は、次のようになります：

```js
  // Effect for the first render (roomId = "general")
  () => {
    const connection = createConnection('general');
    connection.connect();
    return () => connection.disconnect();
  },
  // Dependencies for the first render (roomId = "general")
  ['general']
```

React はこの Effect を実行し、`'general'` チャットルームに接続します。

#### 同じ依存関係で再レンダリングする {/*re-render-with-same-dependencies*/}

`<ChatRoom roomId="general" />` が再レンダリングされたとしましょう。JSX の出力は同じです：

```js
  // JSX for the second render (roomId = "general")
  return <h1>Welcome to general!</h1>;
```

React はレンダリング出力が変化していないことを確認するので、DOM を更新しません。

2 回目のレンダリングによる Effect はこのようになります：

```js
  // Effect for the second render (roomId = "general")
  () => {
    const connection = createConnection('general');
    connection.connect();
    return () => connection.disconnect();
  },
  // Dependencies for the second render (roomId = "general")
  ['general']
```

React は、2 番目のレンダリングからの `['general']` と最初のレンダリングからの `['general']` を比較します。**依存関係はすべて同じなので、React は 2 つ目のレンダリングからの Effect を *無視* します。**

#### 異なる依存関係で再レンダリングする {/*re-render-with-different-dependencies*/}

次に、ユーザーは `<ChatRoom roomId="travel" />` にアクセスします。このとき、コンポーネントは異なる JSX を返します。

```js
  // JSX for the third render (roomId = "travel")
  return <h1>Welcome to travel!</h1>;
```

React は DOM を更新して、`"Welcome to general"` を `"Welcome to travel"` に変更します。

3 回目のレンダリングによる Effect はこのようになります：

```js
  // Effect for the third render (roomId = "travel")
  () => {
    const connection = createConnection('travel');
    connection.connect();
    return () => connection.disconnect();
  },
  // Dependencies for the third render (roomId = "travel")
  ['travel']
```

React は、3 番目のレンダリングの `['travel']` と 2 番目のレンダリングの `['general']` を比較します。1 つの依存関係が異なります。 `Object.is('travel', 'general')` は `false` です。Effect をスキップすることはできません。

**2 番目のレンダリングの Effect はスキップされたため、React は 1 番目のレンダリングの Effect をクリーンアップする必要があります**。最初のレンダリングまでスクロールすると、そのクリーンアップで `createConnection('general')` で作成した接続に対して `disconnect()` を呼び出していることがわかります。これにより、アプリは `'general'` チャットルームから切断されます。

その後、React は 3 番目のレンダリングの Effect を実行します。これは、`'travel'`チャットルームに接続します。

#### アンマウント {/*unmount*/}

最後に、ユーザーがナビゲートして、`ChatRoom` コンポーネントがアンマウントされたとします。React は最後の Effect のクリーンアップ関数を実行します。最後の Effect は、3 番目のレンダリングで使用されたものです。3 番目のレンダリングのクリーンアップでは、`createConnection('travel')` の接続が絶たれます。そのため、アプリは `'travel'` ルームから切断されます。

#### 開発環境のみの動作について {/*development-only-behaviors*/}

[Strict Mode](/apis/strictmode) がオンのとき、React はマウント後にすべてのコンポーネントを一度再マウントします (state と DOM は保存されます)。これにより、[クリーンアップが必要な Effect を見つけやすく](#step-3-add-cleanup-if-needed)、レースコンディションなどのバグを早期に顕在化させることができます。さらに、React は開発中にファイルを保存するたびに Effect を再マウントします。これらの動作はどちらも開発環境でのみ行われます。

</DeepDive>

<Recap>

- イベントとは異なり、Effect は特定のインタラクションではなくレンダリングそのものによって引き起こされます。
- Effect を使用すると、コンポーネントを外部システム（サードパーティ API、ネットワークなど）と同期させることができます。
- デフォルトでは、Effect は（最初のレンダリングを含む）すべてのレンダリングの後に実行されます。
- React は、すべての依存関係が最後のレンダリング時と同じ値である場合に、Effect をスキップします。
- 依存関係を「選択」することはできません。Effect 内部のコードによって決定されます。
- 空の依存関係配列 (`[]`) は、コンポーネントが「実装」される、つまり画面に追加されることに相当します。
- Strict Mode がオンの場合、React はコンポーネントを（開発時のみ！）2 回マウントし、Effect をストレステストします。
- もし、再マウントによって Effect が壊れてしまったら、クリーンアップ関数を実装する必要があります。
- React は Effect が次回実行される前と、アンマウントの際にクリーンアップ関数を呼び出します。

</Recap>

<Challenges>

### マウント時にフィールドをフォーカスする {/*focus-a-field-on-mount*/}

この例では、フォームに `<MyInput />` コンポーネントをレンダリングしています。

入力の [`focus()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus) メソッドを使用して、 `MyInput` が画面に表示されたときに自動的にフォーカスを当てたいです。すでにコメントアウトされた実装がありますが、なかなかうまくいきません。なぜうまくいかないのかを考えて修正してください。(もしあなたが `autoFocus` 属性に慣れているのなら、それは存在しないものとしてください: 同じ機能を一から再実装します)

<Sandpack>

```js MyInput.js active
import { useEffect, useRef } from 'react';

export default function MyInput({ value, onChange }) {
  const ref = useRef(null);

  // TODO: This doesn't quite work. Fix it.
  // ref.current.focus()    

  return (
    <input
      ref={ref}
      value={value}
      onChange={onChange}
    />
  );
}
```

```js App.js hidden
import { useState } from 'react';
import MyInput from './MyInput.js';

export default function Form() {
  const [show, setShow] = useState(false);
  const [name, setName] = useState('Taylor');
  const [upper, setUpper] = useState(false);
  return (
    <>
      <button onClick={() => setShow(s => !s)}>{show ? 'Hide' : 'Show'} form</button>
      <br />
      <hr />
      {show && (
        <>
          <label>
            Enter your name:
            <MyInput
              value={name}
              onChange={e => setName(e.target.value)}
            />
          </label>
          <label>
            <input
              type="checkbox"
              checked={upper}
              onChange={e => setUpper(e.target.checked)}
            />
            Make it uppercase
          </label>
          <p>Hello, <b>{upper ? name.toUpperCase() : name}</b></p>
        </>
      )}
    </>
  );
}
```

```css
label {
  display: block;
  margin-top: 20px;
  margin-bottom: 20px;
}

body {
  min-height: 150px;
}
```

</Sandpack>

動作確認として、「Show form」を押して、入力にフォーカスが当たる（ハイライトされ、カーソルが中に入る）ことを確認します。「Hide form」を押し、再度 「Show form」を押してください。入力が再びハイライトされることを確認してください。

`MyInput` はレンダリングのたびにフォーカスされるのではなく、マウント時にのみフォーカスされるはずです。動作が正しいことを確認するために、「Show form」 を押して、「Make it uppercase」 チェックボックスを繰り返し押してみてください。チェックボックスをクリックしても、その上の入力にはフォーカスが当たらないはずです。

<Solution>

レンダリング中に `ref.current.focus()` を呼び出すことは、_副作用_ であるため、間違っています。副作用は、イベントハンドラ内に配置するか、`useEffect` を使用して宣言する必要があります。`useEffect`を使用する場合、副作用は特定のインタラクションによってではなく、コンポーネントが現れることによって引き起こされます。

間違いを修正するには、`ref.current.focus()` の呼び出しを Effect 宣言にラップします。そして、この Effect がレンダリングのたびに実行されるのではなく、マウント時にのみ実行されるように空の `[]` 依存関係を追加してください。

<Sandpack>

```js MyInput.js active
import { useEffect, useRef } from 'react';

export default function MyInput({ value, onChange }) {
  const ref = useRef(null);

  useEffect(() => {
    ref.current.focus();
  }, []);

  return (
    <input
      ref={ref}
      value={value}
      onChange={onChange}
    />
  );
}
```

```js App.js hidden
import { useState } from 'react';
import MyInput from './MyInput.js';

export default function Form() {
  const [show, setShow] = useState(false);
  const [name, setName] = useState('Taylor');
  const [upper, setUpper] = useState(false);
  return (
    <>
      <button onClick={() => setShow(s => !s)}>{show ? 'Hide' : 'Show'} form</button>
      <br />
      <hr />
      {show && (
        <>
          <label>
            Enter your name:
            <MyInput
              value={name}
              onChange={e => setName(e.target.value)}
            />
          </label>
          <label>
            <input
              type="checkbox"
              checked={upper}
              onChange={e => setUpper(e.target.checked)}
            />
            Make it uppercase
          </label>
          <p>Hello, <b>{upper ? name.toUpperCase() : name}</b></p>
        </>
      )}
    </>
  );
}
```

```css
label {
  display: block;
  margin-top: 20px;
  margin-bottom: 20px;
}

body {
  min-height: 150px;
}
```

</Sandpack>

</Solution>

### 条件付きでフィールドをフォーカスする {/*focus-a-field-conditionally*/}

このフォームには、2 つの `<MyInput />` コンポーネントが表示されます。

「Show form 」を押して、2 番目のフィールドに自動的にフォーカスが当たっていることに注目してください。これは、両方の `<MyInput />` コンポーネントが内部のフィールドにフォーカスを当てようとするためです。2 つの入力フィールドに対して連続して `focus()` を呼び出すと、常に最後の 1 つが 「勝ち」 になります。

最初のフィールドにフォーカスを当てたいとしましょう。最初の `MyInput` コンポーネントは、`true` にセットされたブール値の `shouldFocus` props を受け取ります。`MyInput` が受け取った `shouldFocus` props が `true` である場合にのみ `focus()` が呼び出されるようにロジックを変更します。

<Sandpack>

```js MyInput.js active
import { useEffect, useRef } from 'react';

export default function MyInput({ shouldFocus, value, onChange }) {
  const ref = useRef(null);

  // TODO: call focus() only if shouldFocus is true.
  useEffect(() => {
    ref.current.focus();
  }, []);

  return (
    <input
      ref={ref}
      value={value}
      onChange={onChange}
    />
  );
}
```

```js App.js hidden
import { useState } from 'react';
import MyInput from './MyInput.js';

export default function Form() {
  const [show, setShow] = useState(false);
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  const [upper, setUpper] = useState(false);
  const name = firstName + ' ' + lastName;
  return (
    <>
      <button onClick={() => setShow(s => !s)}>{show ? 'Hide' : 'Show'} form</button>
      <br />
      <hr />
      {show && (
        <>
          <label>
            Enter your first name:
            <MyInput
              value={firstName}
              onChange={e => setFirstName(e.target.value)}
              shouldFocus={true}
            />
          </label>
          <label>
            Enter your last name:
            <MyInput
              value={lastName}
              onChange={e => setLastName(e.target.value)}
              shouldFocus={false}
            />
          </label>
          <p>Hello, <b>{upper ? name.toUpperCase() : name}</b></p>
        </>
      )}
    </>
  );
}
```

```css
label {
  display: block;
  margin-top: 20px;
  margin-bottom: 20px;
}

body {
  min-height: 150px;
}
```

</Sandpack>

解決策を確認するために、「Show form」と「Hide form」を繰り返し押してください。フォームが表示されると、*最初の*入力にのみフォーカスが当たるはずです。これは、親コンポーネントが最初の入力を `shouldFocus={true}` で、2 番目の入力を `shouldFocus={false}` でレンダリングしているからです。また、両方の入力は正しく動作し、両方に入力できることも確認してください。

<Hint>

Effect を条件付きで宣言することはできませんが、Effect に条件付きロジックを含めることは可能です。

</Hint>

<Solution>

条件付きロジックを Effect の中に記述します。`ShouldFocus` は、Effect の内部で使用するため、依存関係として指定する必要があります(これは、ある入力の `shouldFocus` が `false` から `true` に変わった場合、マウント後にフォーカスを当てることを意味します)。

<Sandpack>

```js MyInput.js active
import { useEffect, useRef } from 'react';

export default function MyInput({ shouldFocus, value, onChange }) {
  const ref = useRef(null);

  useEffect(() => {
    if (shouldFocus) {
      ref.current.focus();
    }
  }, [shouldFocus]);

  return (
    <input
      ref={ref}
      value={value}
      onChange={onChange}
    />
  );
}
```

```js App.js hidden
import { useState } from 'react';
import MyInput from './MyInput.js';

export default function Form() {
  const [show, setShow] = useState(false);
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  const [upper, setUpper] = useState(false);
  const name = firstName + ' ' + lastName;
  return (
    <>
      <button onClick={() => setShow(s => !s)}>{show ? 'Hide' : 'Show'} form</button>
      <br />
      <hr />
      {show && (
        <>
          <label>
            Enter your first name:
            <MyInput
              value={firstName}
              onChange={e => setFirstName(e.target.value)}
              shouldFocus={true}
            />
          </label>
          <label>
            Enter your last name:
            <MyInput
              value={lastName}
              onChange={e => setLastName(e.target.value)}
              shouldFocus={false}
            />
          </label>
          <p>Hello, <b>{upper ? name.toUpperCase() : name}</b></p>
        </>
      )}
    </>
  );
}
```

```css
label {
  display: block;
  margin-top: 20px;
  margin-bottom: 20px;
}

body {
  min-height: 150px;
}
```

</Sandpack>

</Solution>

### 2 回発火するインターバルを修正する {/*fix-an-interval-that-fires-twice*/}

この `Counter` コンポーネントは、1 秒ごとに増加するカウンターを表示します。マウント時に、[`setInterval`](https://developer.mozilla.org/en-US/docs/Web/API/setInterval)を呼び出します。これにより、`onTick`が 1 秒ごとに実行されます。`onTick` 関数はカウンターを増加させます。

しかし、1 秒に 1 回増加するのではなく、2 回増加します。なぜでしょうか？バグの原因を見つけて修正してください。

<Hint>

`setInterval` はインターバル ID を返すことを覚えておきましょう。インターバル ID を [`clearInterval`](https://developer.mozilla.org/en-US/docs/Web/API/clearInterval) に渡すと、インターバルを停止させることができます。

</Hint>

<Sandpack>

```js Counter.js active
import { useState, useEffect } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    function onTick() {
      setCount(c => c + 1);
    }

    setInterval(onTick, 1000);
  }, []);

  return <h1>{count}</h1>;
}
```

```js App.js hidden
import { useState } from 'react';
import Counter from './Counter.js';

export default function Form() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(s => !s)}>{show ? 'Hide' : 'Show'} counter</button>
      <br />
      <hr />
      {show && <Counter />}
    </>
  );
}
```

```css
label {
  display: block;
  margin-top: 20px;
  margin-bottom: 20px;
}

body {
  min-height: 150px;
}
```

</Sandpack>

<Solution>

[Strict Mode](/apis/strictmode)が ON の場合（このサイトのサンドボックスのように）、React は開発環境では各コンポーネントを 1 回ずつ再マウントします。そのため、インターバルが 2 回設定され、1 秒ごとにカウンターが 2 回増加されます。

しかし、React の動作はバグの*原因*ではありません（バグはすでにコードに存在しています）。React の動作がバグを目立たせているのです。この Effect はプロセスを開始しますが、クリーンアップする方法を提供しないことが本当の原因です。

このコードを修正するには、 `setInterval` が返すインターバル ID を保存し、 [`clearInterval`](https://developer.mozilla.org/en-US/docs/Web/API/clearInterval) でクリーンアップ関数を実装してください：

<Sandpack>

```js Counter.js active
import { useState, useEffect } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    function onTick() {
      setCount(c => c + 1);
    }

    const intervalId = setInterval(onTick, 1000);
    return () => clearInterval(intervalId);
  }, []);

  return <h1>{count}</h1>;
}
```

```js App.js hidden
import { useState } from 'react';
import Counter from './Counter.js';

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(s => !s)}>{show ? 'Hide' : 'Show'} counter</button>
      <br />
      <hr />
      {show && <Counter />}
    </>
  );
}
```

```css
label {
  display: block;
  margin-top: 20px;
  margin-bottom: 20px;
}

body {
  min-height: 150px;
}
```

</Sandpack>

開発環境では、React はクリーンアップがうまく実装されているかどうかを確認するために、コンポーネントを一度マウントし直します。そのため、 `setInterval` の呼び出しがあり、その直後に `clearInterval` を呼び出し、再び `setInterval` を呼び出すことになります。本番環境では、 `setInterval` の呼び出しは一度だけです。どちらの場合も、ユーザーから見える動作は同じです（カウンターは 1 秒に 1 回増加します）。

</Solution>

### Effect 内部での取得を修正する {/*fix-fetching-inside-an-effect*/}

このコンポーネントは、選択された人物の経歴を表示します。マウント時や `person` が変更されるたびに非同期関数 `fetchBio(person)` を呼び出して経歴をロードします。この非同期関数は、最終的に文字列に解決される [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) を返します。取得が完了したら、 `setBio` を呼び出して、セレクトボックスの下にその文字列を表示します。

<Sandpack>

```js App.js
import { useState, useEffect } from 'react';
import { fetchBio } from './api.js';

export default function Page() {
  const [person, setPerson] = useState('Alice');
  const [bio, setBio] = useState(null);

  useEffect(() => {
    setBio(null);
    fetchBio(person).then(result => {
      setBio(result);
    });
  }, [person]);

  return (
    <>
      <select value={person} onChange={e => {
        setPerson(e.target.value);
      }}>
        <option value="Alice">Alice</option>
        <option value="Bob">Bob</option>
        <option value="Taylor">Taylor</option>
      </select>
      <hr />
      <p><i>{bio ?? 'Loading...'}</i></p>
    </>
  );
}
```

```js api.js hidden
export async function fetchBio(person) {
  const delay = person === 'Bob' ? 2000 : 200;
  return new Promise(resolve => {
    setTimeout(() => {
      resolve('This is ' + person + '’s bio.');
    }, delay);
  })
}

```

</Sandpack>

このコードにはバグがあります。まず「アリス」を選択します。次に「ボブ」を選択し、その直後に「テイラー」を選択してください。これを十分な速さで行うと、そのバグに気がつくでしょう。Taylor が選択されているのに、下の段落に 「This is Bob's bio. 」と書いてあるのです。

なぜこのようなことが起こるのでしょうか？この Effect の中のバグを修正しましょう。

<Hint>

Effect が非同期に何かを取得する場合、通常、クリーンアップが必要です。

</Hint>

<Solution>

このバグを引き起こすには、以下の順序で動作が発生する必要があります：

- `'Bob'` を選択すると、`fetchBio('Bob')` が発火される。
- `'Taylor'` を選択すると、`fetchBio('Taylor')` が発火される。
- **`'Taylor'` の取得は `'Bob'` の取得の前に完了する**。
- `'Taylor'` のレンダリングの Effect で `setBio('This is Taylor's bio')` が呼び出される。
- `'Bob'` の取得が完了する。
- `'Bob'` のレンダリングの Effect で `setBio('This is Bob's bio')`が呼び出される。

このため、Taylor が選択されているにもかかわらず、Bob の bio が表示されるのです。このようなバグは[レースコンディション](https://en.wikipedia.org/wiki/Race_condition)と呼ばれます。2 つの非同期処理が互いに「レース」しており、予期しない順序で到着する可能性があるからです。

このレースコンディションを修正するために、クリーンアップ関数を追加してください：

<Sandpack>

```js App.js
import { useState, useEffect } from 'react';
import { fetchBio } from './api.js';

export default function Page() {
  const [person, setPerson] = useState('Alice');
  const [bio, setBio] = useState(null);
  useEffect(() => {
    let ignore = false;
    setBio(null);
    fetchBio(person).then(result => {
      if (!ignore) {
        setBio(result);
      }
    });
    return () => {
      ignore = true;
    }
  }, [person]);

  return (
    <>
      <select value={person} onChange={e => {
        setPerson(e.target.value);
      }}>
        <option value="Alice">Alice</option>
        <option value="Bob">Bob</option>
        <option value="Taylor">Taylor</option>
      </select>
      <hr />
      <p><i>{bio ?? 'Loading...'}</i></p>
    </>
  );
}
```

```js api.js hidden
export async function fetchBio(person) {
  const delay = person === 'Bob' ? 2000 : 200;
  return new Promise(resolve => {
    setTimeout(() => {
      resolve('This is ' + person + '’s bio.');
    }, delay);
  })
}

```

</Sandpack>

各レンダリングの Effect は、独自の `ignore` 変数を持ちます。初期状態では、`ignore` 変数は `false` に設定されています。しかし、Effect が（別の人を選択されるなどで）クリーンアップされると、その `ignore` 変数は `true` になります。つまり、リクエストがどの順番で完了するかは問題ではありません。最後の人の Effect だけは `ignore` が `false` に設定されているので、 `setBio(result)` を呼び出すことになります。過去の Effect はクリーンアップされているので、`if (!ignore)` チェックにより `setBio` が呼び出されません。

- `'Bob'` を選択すると `fetchBio('Bob')` が発火される。
- `'Taylor'`を選択すると `fetchBio('Taylor')` が発火され、**前(Bob) の Effect をクリーンアップする。**
- `'Taylor'`の取得は、`Bob'`の取得の前に完了する。
- `'Taylor'` のレンダリングの Effect は、 `setBio('This is Taylor' s bio')` を呼び出す。
- `'Bob'`の取得が完了する。
- `'Bob'` レンダリングの Effect は、**`ignore` フラグが `true` に設定されているので何も行わない。**

古くなった API 呼び出しの結果を無視するだけでなく、[`AbortController`](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) を使用して、不要になったリクエストをキャンセルすることも可能です。しかし、これだけではレースコンディションを防ぐには不十分です。取得の後にさらに非同期なステップが連鎖する可能性があるので、 `ignore` のような明示的なフラグを使用することが、この種の問題を解決する最も確実な方法となります。

</Solution>

</Challenges>

