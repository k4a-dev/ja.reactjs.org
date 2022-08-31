---
title: 'Effectは必要ないかもしれない'
---

<Intro>

Effect は、React のパラダイムからの避難ハッチです。React の外に出て、コンポーネントを非 React ウィジェット、ネットワーク、ブラウザ DOM などの外部システムと同期させることができます。外部システムが関与しない場合（例えば、props や state が変化したときにコンポーネントの state を更新したい場合）、Effect は必要ありません。不要な Effect を削除することで、コードが分かりやすくなり、実行速度が向上し、エラーが少なくなります。

</Intro>

<YouWillLearn>

* コンポーネントから不要な Effect を取り除く理由と方法
* Effect を使わずにコストの高い計算をキャッシュする方法
* Effect を使わずにコンポーネントの state をリセットし調整する方法
* イベントハンドラ間でロジックを共有する方法
* どのロジックをイベントハンドラに移すべきか
* 親コンポーネントに変更を通知する方法

</YouWillLearn>

## 不要な Effect を取り除く方法 {/*how-to-remove-unnecessary-effects*/}

Effect が不要となるケースは主に以下の 2 つです：

- **レンダリングのためのデータ変換に Effect は必要ありません。** 例えば、リストを表示する前にフィルタをかけたい場合、リストが変化したときに state 変数を更新する Effect を書きたくなるかもしれません。しかし、これは非効率的です。コンポーネントの state を更新するとき、React はまずコンポーネントの関数を呼び出して、画面に表示されるべき内容を計算します。次に、React はこれらの変更を DOM に[「commit」](/learn/render-and-commit)し、画面を更新するのです。その後、React は Effect を実行します。もし、Effect もすぐに state を更新してしまうと、プロセス全体を一からやり直すことになります。不要なレンダーパスを避けるには、コンポーネントの最上位ですべてのデータを変換します。このコードは、props や state が変更されるたびに、自動的に再実行されます。
- **ユーザーイベントの処理に Effect は必要ありません。** 例えば、`/api/buy` の POST リクエストを送信し、ユーザーが製品を購入したときに通知を表示したいとします。購入ボタンのクリックイベントハンドラでは、何が起こったかを正確に知ることができます。Effect が実行される頃には、ユーザが何をしたのか（例えば、どのボタンがクリックされたのか）、わかりません。通常、ユーザーイベントは対応するイベントハンドラで処理します。

しかし、外部システムと[同期](/learn/synchronizing-with-effects#what-are-effects-and-how-are-they-different-from-events)するときには Effect が必要です。例えば、jQuery のウィジェットを React の state と同期させる Effect を書くことができます。また、検索結果を現在の検索クエリに同期させるといったように、 Effect でデータを取得することもできます。最近の[フレームワーク](/learn/start-a-new-react-project#building-with-a-full-featured-framework)では、コンポーネントに直接 Effect を記述するより効率的な組み込みデータ取得機能を提供していることを覚えておいてください。

正しい感覚を得るために、いくつかの一般的な具体例を見てみましょう！

### props や state に基づく state の更新 {/*updating-state-based-on-props-or-state*/}

`firstName` と `lastName` という 2 つの state 変数を持つコンポーネントがあるとします。それらを連結して、`fullName`を計算したいです。さらに、 `firstName` や `lastName` が変更されるたびに、 `fullName` が更新されるようにしたいです。最初の直感は、`fullName`の state 変数を追加して、Effect でそれを更新することかもしれません：

```js {5-9}
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');

  // 🔴 Avoid: redundant state and unnecessary Effect
  const [fullName, setFullName] = useState('');
  useEffect(() => {
    setFullName(firstName + ' ' + lastName);
  }, [firstName, lastName]);
  // ...
}
```

これは必要以上に複雑です。また、非効率的でもあります。`fullName`の値が古いまま全てのレンダリングを行い、その後すぐに更新された値で再レンダリングします。state 変数と Effect の両方を削除してください：

```js {4-5}
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  // ✅ Good: calculated during rendering
  const fullName = firstName + ' ' + lastName;
  // ...
}
```

**すでにある props や state から値が計算できる場合、 [state を使用しないでください。](/learn/choosing-the-state-structure#avoid-redundant-state) その代わり、レンダリング中に値を計算します。** これにより、（余分な更新「cascading updates」を回避して）コードが速くなり、（いくつかのコードを削除できるので）シンプルになり、（異なる state 変数が互いに同期しないことによって引き起こされるバグを回避して）エラーが起こりにくくなります。このアプローチが馴染まない場合は、[Thinking in React](/learn/thinking-in-react#step-3-find-the-minimal-but-complete-representation-of-ui-state) に、state に入れるべきものについてのガイダンスがあります。

### コストの高い計算をキャッシュする {/*caching-expensive-calculations*/}

このコンポーネントは、props で受け取った `todos` を `filter` props に従ってフィルタリングすることで、 `visibleTodos` を計算します。この結果を state 変数に格納し、Effect で更新したくなるかもしれません：

```js {4-8}
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');

  // 🔴 Avoid: redundant state and unnecessary Effect
  const [visibleTodos, setVisibleTodos] = useState([]);
  useEffect(() => {
    setVisibleTodos(getFilteredTodos(todos, filter));
  }, [todos, filter]);

  // ...
}
```

先の例と同様、これは不要かつ非効率的です。まず、state と Effect を削除します：

```js {3-4}
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // ✅ This is fine if getFilteredTodos() is not slow.
  const visibleTodos = getFilteredTodos(todos, filter);
  // ...
}
```

多くの場合、このコードで大丈夫です! しかし、もしかしたら `getFilteredTodos()` が遅いかもしれないですし、たくさんの `todos` があるかもしれません。そのような場合、`newTodo` のような無関係な state 変数が変化したときに `getFilteredTodos()` を再計算したくありません。

[`useMemo`](/apis/usememo) フックでラップすることで、コストの高い計算をキャッシュ（または [「メモ化」](https://en.wikipedia.org/wiki/Memoization)）することができます：

```js {5-8}
import { useMemo, useState } from 'react';

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  const visibleTodos = useMemo(() => {
    // ✅ Does not re-run unless todos or filter change
    return getFilteredTodos(todos, filter);
  }, [todos, filter]);
  // ...
}
```

一行で書く場合は：

```js {5-6}
import { useMemo, useState } from 'react';

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // ✅ Does not re-run getFilteredTodos() unless todos or filter change
  const visibleTodos = useMemo(() => getFilteredTodos(todos, filter), [todos, filter]);
  // ...
}
```

**これは `todos` か `filter` のどちらかが変更されない限り、内部関数の再実行はしないことを React に伝えます。** React は最初のレンダリング時に `getFilteredTodos()` の戻り値を記憶しています。次のレンダリングでは、`todos` または `filter` が変わっていないかどうかを確認します。もし前回と同じであれば、`useMemo`は最後に保存した結果を返します。しかし、異なる場合は、React はラップした関数を再度呼び出します (そして、代わりにその結果を保存します)。

[`useMemo`](/apis/usememo) でラップした関数はレンダリング中に実行されるので、これは [純粋な計算](/learn/keeping-components-pure) でのみ動作します。

<DeepDive title="コストの高い計算かどうかを判断する方法">

一般的に、何千ものオブジェクトを作成したりループさせたりしない限り、おそらく高いコストではないでしょう。より確信を得たい場合は、コンソールログを追加して、コードの一部で費やされた時間を測定することができます：

```js {1,3}
console.time('filter array');
const visibleTodos = getFilteredTodos(todos, filter);
console.timeEnd('filter array');
```

（input に入力するなどによって）測定したいインタラクションを実行します。すると、コンソールに `filter array.0.15ms` のようなログが表示されます。もし、記録された時間全体がかなりの量(`1ms`以上)になるのであれば 、その計算をメモしておくことが意味を持つかもしれません。その計算を `useMemo` でラップして、そのインタラクションで記録される時間の合計が減少したかどうかを検証することができます。

```js
console.time('filter array');
const visibleTodos = useMemo(() => {
  return getFilteredTodos(todos, filter); // Skipped if todos and filter haven't changed
}, [todos, filter]);
console.timeEnd('filter array');
```

`useMemo` は *最初の* レンダリングを速くするものではありません。更新時の不要な作業を省略できるようになるだけです。

あなたのマシンがおそらくユーザーよりも高速であることを念頭に置き、人工的なスローダウンでパフォーマンスをテストするのは良いアイデアです。例えば、Chrome ではこのために[CPU Throttling](https://developer.chrome.com/blog/new-in-devtools-61/#throttling) というオプションが用意されています。

また、開発環境のパフォーマンス測定は、最も正確な結果を得られないことに注意してください。（例えば、[Strict Mode](/apis/strictmode) がオンの場合、各コンポーネントは一度ではなく二度レンダリングされます。）最も正確な時間を得るには、本番環境向けにアプリを構築し、ユーザーが持つようなデバイスでテストしてください。

</DeepDive>

### props が変更されたときにすべての state をリセットする {/*resetting-all-state-when-a-prop-changes*/}

この `ProfilePage` コンポーネントは `userId` props を受け取ります。このページはコメント入力を含んでおり、その値を保持するために state 変数 `comment` を使用します。ある日、あなたは問題に気づきました。あるプロファイルから別のプロファイルに移動するとき、`comment` の state がリセットされないのです。その結果、誤って間違ったユーザーのプロファイルにコメントを投稿してしまうことがあります。この問題を解決するために、 `userId` が変更されるたびに `comment` state 変数をクリアするようにします：

```js {4-7}
export default function ProfilePage({ userId }) {
  const [comment, setComment] = useState('');

  // 🔴 Avoid: Resetting state on prop change in an Effect
  useEffect(() => {
    setComment('');
  }, [userId]);
  // ...
}
```

これは非効率的です。なぜなら `ProfilePage` とその子要素は、まず古い値でレンダリングし、それからもう一度レンダリングするからです。また、`ProfilePage` の内部に state を持つ *すべての* コンポーネントでこれを行う必要があるため、複雑です。例えば、コメント UI がネストされている場合、ネストされたコメントの state もクリアしたいと思うでしょう。

その代わりに、明示的にキーを与えることで、各ユーザーのプロファイルが概念的に*異なる*プロファイルであることを React に伝えることができます。コンポーネントを 2 つに分割し、外側のコンポーネントから内側のコンポーネントに `key` 属性を渡します。

```js {5,11-12}
export default function ProfilePage({ userId }) {
  return (
    <Profile
      userId={userId} 
      key={userId}
    />
  );
}

function Profile({ userId }) {
  // ✅ This and any other state below will reset on key change automatically
  const [comment, setComment] = useState('');
  // ...
}
```

通常、React は同じコンポーネントを同じ場所にレンダリングする場合、その state を保持します。**キーとして `userId` を `Profile` コンポーネントに渡すことで、異なる `userId` を持つ 2 つの `Profile` コンポーネントを、state を共有しない 2 つの異なるコンポーネントとして扱うように React に要求しています**。 キーが変わるたびに、React は DOM を再作成して `Profile` コンポーネントとその子コンポーネント全ての [state をリセット](/learn/preserving-and-resetting-state#option-2-resetting-state-with-a-key) します。その結果、プロファイル間を移動する際に、`comment`フィールドは自動的にクリアされます。

この例では、外側の `ProfilePage` コンポーネントのみがエクスポートされ、プロジェクト内の他のファイルから見えるようになっていることに注意してください。`ProfilePage` をレンダリングするコンポーネントは、キーを渡す必要はありません。通常の props として `userId` を渡します。`ProfilePage` が `key` として内部の `Profile` コンポーネントに渡すのは、実装の詳細のためです。

### props が変更されたときに state を調整する {/*adjusting-some-state-when-a-prop-changes*/}

props が変更されたときに、すべての state ではなく一部の state をリセットしたり、調整したりしたい場合があります。

この `List` コンポーネントは `items` のリストを prop として受け取り、選択されたアイテムを state 変数 `selection` に保持します。`items` props が異なる配列を受け取るたびに、 `selection` を `null` にリセットしたいと思います：

```js {5-8}
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // 🔴 Avoid: Adjusting state on prop change in an Effect
  useEffect(() => {
    setSelection(null);
  }, [items]);
  // ...
}
```

これもまた、理想的ではありません。`items`が変わるたびに、`List` とその子コンポーネントは、最初は古い `selection` 値でレンダリングされます。その後、React は DOM を更新し、Effects を実行します。最後に、 `setSelection(null)` を呼び出すと、 `List` とその子コンポーネントが再度レンダリングされ、この一連の処理が再開されます。

まずは、Effect を削除することから始めましょう。その代わり、レンダリング中に直接 state を調整します。

```js {5-11}
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // Better: Adjust the state while rendering
  const [prevItems, setPrevItems] = useState(items);
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null);
  }
  // ...
}
```

このような[過去のレンダリング情報の保存](/apis/usestate#storing-information-from-previous-renders)は受け入れがたいかもしれませんが、Effect で同じ state を更新するよりはましでしょう。上記の例では、レンダリング中に `setSelection` が直接呼び出されます。React は `return` 文で終了した後、`List` を _直ちに_ 再レンダリングします。その時点では、React は `List` の子要素をレンダリングしておらず、DOM も更新していないので、`List` の子要素は古い `selection` 値のレンダリングをスキップすることができます。

レンダリング中にコンポーネントを更新すると、React は返された JSX を捨て、直ちにレンダリングを再試行します。非常に遅い再試行を避けるために、React はレンダリング中に _同じ_ コンポーネントの state のみを更新できるようにしています。レンダリング中に別のコンポーネントの state を更新すると、エラーが表示されます。ループを避けるために、`items !== prevItems` のような条件が必要です。このように state を調整することはできますが、その他の副作用（DOM の変更やタイムアウトの設定など）はイベントハンドラや Effect に残して、[コンポーネントを予測可能に保つ](/learn/keeping-components-pure)ようにしてください。

**このパターンは Effect よりも効率的ですが、ほとんどのコンポーネントでは必要としません。**どの方法を取るにしても、props や他の state に基づいて state を調整することは、データフローの理解とデバッグを難しくします。代わりに、[キーですべての state をリセットする](#resetting-all-state-when-a-prop-changes) か [レンダリング中にすべてを計算](#updating-state-based-on-props-or-state) できないかを常にチェックするようにして下さい。たとえば、選択された _項目_ を保存（及びリセット）する代わりに、選択された *項目 ID* を保存することができます：

```js {3-5}
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);
  // ✅ Best: Calculate everything during rendering
  const selection = items.find(item => item.id === selectedId) ?? null;
  // ...
}
```

これで、state を「調整」する必要は全くありません。選択された ID を持つアイテムがリストにあれば、それは選択されたままです。そうでない場合は、レンダリング時に計算される `selection` は、マッチするアイテムが見つからなかったので `null` になります。この動作はこれまでと少し異なりますが、 `items` に対するほとんどの変更が選択を保持するようになったので、間違いなく優れています。しかし、 `selectedId` を持つアイテムが存在しないかもしれないので、これ以降のすべてのロジックで `selection` を使用する必要があります。

### イベントハンドラ間のロジックの共有 {/*sharing-logic-between-event-handlers*/}

商品を購入するための 2 つのボタン(Buy と Checkout)がある商品ページがあるとします。ユーザーが商品をカートに入れるたびに、[トースト](https://uxdesign.cc/toasts-or-snack-bars-design-organic-system-notifications-1236f2883023)通知を表示したいとします。両方のボタンのクリックハンドラに `showToast()` を追加すると繰り返しになるので、このロジックを Effect に配置したくなるかもしれません：

```js {2-7}
function ProductPage({ product, addToCart }) {
  // 🔴 Avoid: Event-specific logic inside an Effect
  useEffect(() => {
    if (product.isInCart) {
      showToast(`Added ${product.name} to the shopping cart!`);
    }
  }, [product]);

  function handleBuyClick() {
    addToCart(product);
  }

  function handleCheckoutClick() {
    addToCart(product);
    navigateTo('/checkout');
  }
  // ...
}
```

この Effect は不要です。また、バグの原因になる可能性も高いです。例えば、ページが再読み込みされる間に、アプリがショッピングカートを「記憶」しているとします。一度商品をカートに入れ、ページを更新すると、トースト通知が再び表示されます。その商品のページを更新するたびに表示され続けることになります。これは、ページロード時に `product.isInCart` が既に `true` になっているためで、上記の Effect は `showToast()` を呼び出すことになります。

**あるコードが Effect の中にあるべきか、イベントハンドラの中にあるべきか迷ったときは、このコードがなぜ実行される必要があるのか、自問自答してください。** この例では、トーストが表示されるのはユーザーがボタンを押したからであり、商品ページが表示されたからではありません。Effect を削除し、共有ロジックを関数に入れ、両方のイベントハンドラから呼び出すようにします：

```js {2-6,9,13}
function ProductPage({ product, addToCart }) {
  // ✅ Good: Event-specific logic is called from event handlers
  function buyProduct() {
    addToCart(product);
    showToast(`Added ${product.name} to the shopping cart!`);    
  }

  function handleBuyClick() {
    buyProduct();
  }

  function handleCheckoutClick() {
    buyProduct();
    navigateTo('/checkout');
  }
  // ...
}
```

不要な Effect が削除され、バグが修正されました。

### POST リクエストを送信する {/*sending-a-post-request*/}

以下の `Form` コンポーネントは 2 種類の POST リクエストを送信します。マウント時には Analytics イベントを送信します。フォームに入力して Submit ボタンをクリックすると、 `/api/register` エンドポイントに POST リクエストが送信されます：

```js {5-8,10-16}
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  // ✅ Good: This logic should run because the component was displayed
  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_form' });
  }, []);

  // 🔴 Avoid: Event-specific logic inside an Effect
  const [jsonToSubmit, setJsonToSubmit] = useState(null);
  useEffect(() => {
    if (jsonToSubmit !== null) {
      post('/api/register', jsonToSubmit);
    }
  }, [jsonToSubmit]);

  function handleSubmit(e) {
    e.preventDefault();
    setJsonToSubmit({ firstName, lastName });
  }
  // ...
}
```

先ほどの例と同じ基準を適用してみましょう。

アナリティクスの POST リクエストは、Effect のままであるべきです。なぜなら、analytics イベントを送信する _理由_ は、フォームが表示されたことだからです。(開発環境では 2 回発火してしまいますが、その対処法は [こちら](/learn/synchronizing-with-effects#sending-analytics) を参照してください。)

しかし、`/api/register` POST リクエストは、フォームが _表示される_ ことによって発生するわけではありません。ユーザーがボタンを押したときという、ある特定の瞬間にだけリクエストを送りたいのです。これは特定のインタラクションでのみ発生するはずです。2 つ目の Effect を削除し、POST リクエストをイベントハンドラに移動します：

```js {12-13}
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  // ✅ Good: This logic runs because the component was displayed
  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_form' });
  }, []);

  function handleSubmit(e) {
    e.preventDefault();
    // ✅ Good: Event-specific logic is in the event handler
    post('/api/register', { firstName, lastName });
  }
  // ...
}
```

あるロジックをイベントハンドラに入れるか、Effect に入れるかを選択するとき、答える必要がある主な質問は、ユーザーの視点から見てそれがどのようなロジックであるかということです。もし、このロジックが特定のインタラクションによって引き起こされるものであれば、イベントハンドラに格納します。ユーザーが画面上のコンポーネントを見ることによって引き起こされるのであれば、Effect に記述してください。

### アプリケーションの初期化 {/*initializing-the-application*/}

アプリのロード時に一度だけ実行されるロジックがあります。そのようなロジックは、トップレベルコンポーネントの Effect に配置します：

```js {2-6}
function App() {
  // 🔴 Avoid: Effects with logic that should only ever run once
  useEffect(() => {
    loadDataFromLocalStorage();
    checkAuthToken();
  }, []);
  // ...
}
```

しかし、このロジックが[開発環境で 2 回実行される](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)ことがすぐに発覚します。この関数は 2 回呼び出されるようには設計されていないため、認証トークンを無効にしてしまうなどの問題を引き起こします。一般に、コンポーネントは再マウントされたときに回復力を持つべきです。これには、トップレベルの `App` コンポーネントも含まれます。実際の運用では再マウントされることはないかもしれませんが、すべてのコンポーネントで同じ制約に従うことで、コードの移動と再利用が容易になります。もし、あるロジックが *コンポーネントマウントに一度*ではなく、_アプリロードに一度_ 実行されなければならない場合、トップレベルの変数を追加して、それが既に実行されたかどうかを追跡し常に再実行をスキップすることができます：

```js {1,5-6,10}
let didInit = false;

function App() {
  useEffect(() => {
    if (!didInit) {
      didInit = true;
      // ✅ Only runs once per app load
      loadDataFromLocalStorage();
      checkAuthToken();
    }
  }, []);
  // ...
}
```

また、モジュールの初期化時やアプリのレンダリング前に実行することも可能です：

```js {1,5}
if (typeof window !== 'undefined') { // Check if we're running in the browser.
   // ✅ Only runs once per app load
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

トップレベルのコードは、たとえそれが最終的にレンダリングされないとしても、コンポーネントがインポートされたときに一度だけ実行されます。任意のコンポーネントをインポートしたときの速度低下や想定外の動作を避けるために、このパターンを使い過ぎないようにしましょう。アプリ全体の初期化ロジックは `App.js` のようなルートコンポーネントモジュールか、アプリケーションのエントリポイントモジュールに留めておいてください。

### state の変更を親コンポーネントに通知する {/*notifying-parent-components-about-state-changes*/}

例えば、`Toggle` コンポーネントを書いていて、内部に `true` か `false` のどちらかを保持する `isOn` stateがあるとします。state を切り替えるには（クリックやドラッグなど）いくつかの方法があります 。`onChange` イベントを公開し、Effect から呼び出すことにします：

```js {4-7}
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  // 🔴 Avoid: The onChange handler runs too late
  useEffect(() => {
    onChange(isOn);
  }, [isOn, onChange])

  function handleClick() {
    setIsOn(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      setIsOn(true);
    } else {
      setIsOn(false);
    }
  }

  // ...
}
```

先ほどと同様に、これは理想的ではありません。まず `Toggle` が state を更新し、React が画面を更新します。次に、React は Effect を実行し、親コンポーネントから渡された `onChange` 関数を呼び出します。ここで、親コンポーネントは自身の state を更新し、別のレンダーパスを開始します。すべてを 1 つのパスで行う方が良いでしょう。

Effect を削除して、代わりに同じイベントハンドラで*両方の*コンポーネントの state を更新します：

```js {5-7,11,16,18}
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  function updateToggle(nextIsOn) {
    // ✅ Good: Perform all updates during the event that caused them
    setIsOn(nextIsOn);
    onChange(nextIsOn);
  }

  function handleClick() {
    updateToggle(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      updateToggle(true);
    } else {
      updateToggle(false);
    }
  }

  // ...
}
```

この方法では、`Toggle`コンポーネントとその親コンポーネントの両方が、イベント中に state を更新します。React は異なるコンポーネントからの更新を[バッチ処理](/learn/queueing-a-series-of-state-updates)でまとめて行うので、結果としてレンダーパスは 1 回だけとなります。

state を完全に削除して、代わりに親コンポーネントから `isOn` を受け取ることも可能かもしれません：

```js {1,2}
// ✅ Also good: the component is fully controlled by its parent
function Toggle({ isOn, onChange }) {
  function handleClick() {
    onChange(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      onChange(true);
    } else {
      onChange(false);
    }
  }

  // ...
}
```

[state をリフトアップ](/learn/sharing-state-between-components) して、親コンポーネント自身の state を切り替えることで、親コンポーネントが `Toggle` を完全に制御できるようにします。これは、親コンポーネントがより多くのロジックを含む必要があることを意味しますが、気を配らなければいけない state は全体的に少なくなります。2 つの異なる state 変数を同期させようとする行為は、代わりに state をリフトアップするべきというサインです！

### 親にデータを渡す {/*passing-data-to-the-parent*/}

以下の `Child` コンポーネントは、いくつかのデータを取得し、それを `Parent` コンポーネントに Effect で渡します：

```js {9-14}
function Parent() {
  const [data, setData] = useState(null);
  // ...
  return <Child onFetched={setData} />;
}

function Child({ onFetched }) {
  const data = useSomeAPI();
  // 🔴 Avoid: Passing data to the parent in an Effect
  useEffect(() => {
    if (data) {
      onFetched(data);
    }
  }, [onFetched, data]);
  // ...
}
```

React では、データは親コンポーネントから子コンポーネントに流れます。画面上で何か不具合があったとき、どのコンポーネントが間違った prop を渡しているか、あるいは間違った state になっているかを見つけるまで、コンポーネントチェーンをさかのぼることで、その情報がどこから来たかを追跡することができます。子コンポーネントが Effect で親コンポーネントの state を更新する場合、データの流れを追跡するのは非常に難しくなります。子コンポーネントと親コンポーネントの両方が同じデータを必要とするので、親コンポーネントにそのデータを取得させ、代わりに子コンポーネントに*受け渡す*ようにします：

```js {4-5}
function Parent() {
  const data = useSomeAPI();
  // ...
  // ✅ Good: Passing data down to the child
  return <Child data={data} />;
}

function Child({ data }) {
  // ...
}
```

データは親から子へと流れるので、よりシンプルで、データの流れを予測しやすくなりました。

### 外部ストアを購読する {/*subscribing-to-an-external-store*/}

時には、あなたのコンポーネントは React の state の外側にあるデータを購読する必要があるかもしれません。このデータは、サードパーティライブラリや組み込みのブラウザ API からのものである可能性があります。このデータは React が知らないうちに変更される可能性があるので、コンポーネントを手動で購読しなければいけません。これは Effect で行われることが多く、以下のような実装になります：

```js {2-17}
function useOnlineStatus() {
  // Not ideal: Manual store subscription in an Effect
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function updateState() {
      setIsOnline(navigator.onLine);
    }

    updateState();

    window.addEventListener('online', updateState);
    window.addEventListener('offline', updateState);
    return () => {
      window.removeEventListener('online', updateState);
      window.removeEventListener('offline', updateState);
    };
  }, []);
  return isOnline;
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```

ここでは、コンポーネントは外部のデータストア（ブラウザの `navigator.onLine` API）を購読します。この API はサーバ上に存在しない（最初の HTML の生成に使用できない）ので、最初は state が `true` に設定されています。データストアの値がブラウザで変更されるたびに、コンポーネントは state を更新します。

Effect を使うのが一般的ですが、React には外部ストアを購読するための専用のフックがあるため、そちらを使うのが望ましいでしょう。Effect を削除して、[`useSyncExternalStore`](/apis/usesyncexternalstore) への呼び出しに置き換えます：

```js {11-16}
function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

function useOnlineStatus() {
  // ✅ Good: Subscribing to an external store with a built-in Hook
  return useSyncExternalStore(
    subscribe, // React won't resubscribe for as long as you pass the same function
    () => navigator.onLine, // How to get the value on the client
    () => true // How to get the value on the server
  );
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```

この方法は、Effect を使って手動で Mutable なデータを React の state に同期させるよりもエラーが起こりにくいです。一般的には、上記の `useOnlineStatus()` のようなカスタムフックを書くことで、個々のコンポーネントでコードを繰り返す必要がないようにします。[React コンポーネントから外部ストアを購読する方法についてはこちら](/apis/usesyncexternalstore)を参照してください。

### データの取得 {/*fetching-data*/}

多くのアプリでは、Effect を使ってデータの取得を開始します。このようにデータ取得の Effect を書くことはよくあります：

```js {5-10}
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);

  useEffect(() => {
    // 🔴 Avoid: Fetching without cleanup logic
    fetchResults(query, page).then(json => {
      setResults(json);
    });
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```

この取得をイベントハンドラに移す必要は*ありません* 。

これは、イベントハンドラにロジックを入れる必要があった先ほどの例と矛盾しているように思えるかもしれません。しかし、今回の取得の主な理由は、*タイピングイベントではないこと*を考慮してください。検索入力は多くの場合、URL からあらかじめ入力されており、ユーザーは入力に触れることなく、戻ったり進んだりすることがあります。`page` と `query` がどこから来るかは重要ではありません。このコンポーネントが表示されている間は、現在の `page` と `query` に従って、ネットワークからのデータで `results` を [同期](/learn/synchronizing-with-effects) しておきたいものです。これが Effect である理由です。

しかし、上のコードにはバグがあります。例えば、あなたが `"hello"` と速くタイプしたとする。すると、`query` が `"h"` から `"he"`, `"hel"`, `"hell"`, そして `"hello"` と変化していくでしょう。これは別々の取得を開始しますが、どの順番で応答が到着するかは保証されません。例えば、`"hell"` のレスポンスは `"hello"` のレスポンスの _後_ に到着するかもしれません。`setResults()` を最後に呼び出すので、間違った検索結果が表示されることになります。これは [「レースコンディション」](https://en.wikipedia.org/wiki/Race_condition) と呼ばれます。2 つの異なるリクエストがお互いに「競争」して、期待とは異なる順番でやってくるのです。

**レースコンディションを修正するには、古い応答を無視する[クリーンアップ関数を追加する](learn/synchronizing-with-effects#fetching-data)必要があります。**

```js {5,7,9,11-13}
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1); 
  useEffect(() => {
    let ignore = false;
    fetchResults(query, page).then(json => {
      if (!ignore) {
        setResults(json);
      }
    });
    return () => {
      ignore = true;
    };
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```

これにより、Effect がデータを取得する際、最後に要求されたもの以外のすべてのレスポンスは無視されるようになります。

データ取得の実装で難しいのは、レースコンディションの処理だけではありません。（ユーザーが Back をクリックすると、スピナーの代わりに前の画面がすぐに表示されるように）レスポンスをキャッシュする方法、（サーバーでレンダリングされた最初の HTML がスピナーの代わりに取得したコンテンツを含むように）サーバーで取得する方法、（データを取得する必要がある子コンポーネントが取得を始める前にその上のすべての親のデータ取得を待つ必要がないように）ネットワークのウォーターフォールを回避する方法も考えたいかも知れません。**これらの問題は、React だけでなく、どの UI ライブラリにも当てはまります。これらを解決するのは簡単ではありません。そのため、最近の[フレームワーク](/learn/start-a-new-react-project#building-with-a-full-featured-framework) では、コンポーネントで直接 Effect を記述するより効率の良い組み込みデータ取得機能を提供しています**。

フレームワークを使っていない（自分で構築もしたくない）が、Effect からのデータ取得をより人間工学的に行いたい場合、以下のように取得ロジックをカスタムフックに抽出することを検討してください：

```js {4}
function SearchResults({ query }) {
  const [page, setPage] = useState(1); 
  const params = new URLSearchParams({ query, page });
  const results = useData(`/api/search?${params}`);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}

function useData(url) {
  const [result, setResult] = useState(null);
  useEffect(() => {
    let ignore = false;
    fetch(url)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setResult(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [url]);
  return result;
}
```

エラー処理のためのロジックや、コンテンツがロードされているかどうかを追跡するためのロジックも追加したいとも思うでしょう。このようなフックを自分で作ることもできますし、React のエコシステムですでに利用可能な多くのソリューションのうちの一つを使うこともできます。**この方法だけではフレームワークの組み込みデータ取得機能ほど効率的ではありませんが、データ取得ロジックをカスタムフックに移動することで、後で効率的なデータ取得戦略を採用することが容易になります。**

一般的に、Effect を書かなければならないときはいつでも、上記の `useData` のように、より宣言的で目的の API を持つカスタムフック に機能の一部を抽出することができるかどうか、目を光らせておいてください。コンポーネントの中に生の `useEffect` の呼び出しが少なければ少ないほど、アプリケーションのメンテナンスが容易になります。

<Recap>

- レンダリング中に何かを計算できるのであれば、Effect は必要ない。
- コストの高い計算をキャッシュするには、`useMemo` を`useEffect`の代わりに追加する。
- コンポーネントツリー全体の state をリセットするには、別の `key` を渡す。
- props の変更に応じて特定の state をリセットするには、レンダリング中に実行する。
- コンポーネントが *表示* されたために実行する必要があるコードは Effect に、それ以外はイベントに記述する。
- 複数のコンポーネントの state を更新する必要がある場合は、1 つのイベントの中で行う方がよい。
- 異なるコンポーネントの state 変数を同期させようとするときは、必ず state を持ち上げることを考える。
- Effect でデータを取得することは可能だが、race condition を避けるためにクリーンアップを実装する必要がある。

</Recap>

<Challenges>

### Effects を使わずにデータを変換する {/*transform-data-without-effects*/}

下の `TodoList` は、Todo の一覧を表示します。「Show only active todos」チェックボックスをチェックすると、完了した TODO はリストに表示されません。どの Todo が表示されているかにかかわらず、フッターにはまだ完了していない Todo の数が表示されます。

不要な state と Effect をすべて削除して、このコンポーネントを簡素化しましょう：

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { initialTodos, createTodo } from './todos.js';

export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);
  const [activeTodos, setActiveTodos] = useState([]);
  const [visibleTodos, setVisibleTodos] = useState([]);
  const [footer, setFooter] = useState(null);

  useEffect(() => {
    setActiveTodos(todos.filter(todo => !todo.completed));
  }, [todos]);

  useEffect(() => {
    setVisibleTodos(showActive ? activeTodos : todos);
  }, [showActive, todos, activeTodos]);

  useEffect(() => {
    setFooter(
      <footer>
        {activeTodos.length} todos left
      </footer>
    );
  }, [activeTodos]);

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={showActive}
          onChange={e => setShowActive(e.target.checked)}
        />
        Show only active todos
      </label>
      <NewTodo onAdd={newTodo => setTodos([...todos, newTodo])} />
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ? <s>{todo.text}</s> : todo.text}
          </li>
        ))}
      </ul>
      {footer}
    </>
  );
}

function NewTodo({ onAdd }) {
  const [text, setText] = useState('');

  function handleAddClick() {
    setText('');
    onAdd(createTodo(text));
  }

  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAddClick}>
        Add
      </button>
    </>
  );
}
```

```js todos.js
let nextId = 0;

export function createTodo(text, completed = false) {
  return {
    id: nextId++,
    text,
    completed
  };
}

export const initialTodos = [
  createTodo('Get apples', true),
  createTodo('Get oranges', true),
  createTodo('Get carrots'),
];
```

```css
label { display: block; }
input { margin-top: 10px; }
```

</Sandpack>

<Hint>

レンダリング中に何かを計算できるのであれば、state やそれを更新する Effect は必要ありません。

</Hint>

<Solution>

この例では、必要不可欠な state は `todos` のリストと、チェックボックスがチェックされているかどうかを表す `showActive` の 2 つだけです。その他の state 変数は[余計](/learn/choosing-the-state-structure#avoid-redundant-state) で、レンダリング中に計算することができます。これには `footer` も含まれ、JSX に直接移動させることが可能です。

結果はこのようになるはずです：

<Sandpack>

```js
import { useState } from 'react';
import { initialTodos, createTodo } from './todos.js';

export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);
  const activeTodos = todos.filter(todo => !todo.completed);
  const visibleTodos = showActive ? activeTodos : todos;

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={showActive}
          onChange={e => setShowActive(e.target.checked)}
        />
        Show only active todos
      </label>
      <NewTodo onAdd={newTodo => setTodos([...todos, newTodo])} />
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ? <s>{todo.text}</s> : todo.text}
          </li>
        ))}
      </ul>
      <footer>
        {activeTodos.length} todos left
      </footer>
    </>
  );
}

function NewTodo({ onAdd }) {
  const [text, setText] = useState('');

  function handleAddClick() {
    setText('');
    onAdd(createTodo(text));
  }

  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAddClick}>
        Add
      </button>
    </>
  );
}
```

```js todos.js
let nextId = 0;

export function createTodo(text, completed = false) {
  return {
    id: nextId++,
    text,
    completed
  };
}

export const initialTodos = [
  createTodo('Get apples', true),
  createTodo('Get oranges', true),
  createTodo('Get carrots'),
];
```

```css
label { display: block; }
input { margin-top: 10px; }
```

</Sandpack>

</Solution>

### Effect を使わずに計算をキャッシュする {/*cache-a-calculation-without-effects*/}

この例では、Todos のフィルタリングは `getVisibleTodos()` という別の関数に抽出されています。この関数の内部には `console.log()` が含まれており、この関数が呼び出されたことに気づくことができます。「Show only active todos」を切り替えて、`getVisibleTodos()`が再実行されることに注目してください。表示するべき TODO を切り替えると、表示が更新されるので、これは予想通りの結果です。

あなたの仕事は、`TodoList` コンポーネントの `visibleTodos` リストを再計算する Effect を削除することです。また、入力時に `getVisibleTodos()` が再実行されない (つまりログが出力されない) ことを確認する必要があります。

<Hint>

一つの解決策は、 `useMemo` 呼び出しを追加して、表示されている Todos をキャッシュすることです。もうひとつ、あまり目立たない解決策もあります。

</Hint>

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { initialTodos, createTodo, getVisibleTodos } from './todos.js';

export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);
  const [text, setText] = useState('');
  const [visibleTodos, setVisibleTodos] = useState([]);

  useEffect(() => {
    setVisibleTodos(getVisibleTodos(todos, showActive));
  }, [todos, showActive]);

  function handleAddClick() {
    setText('');
    setTodos([...todos, createTodo(text)]);
  }

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={showActive}
          onChange={e => setShowActive(e.target.checked)}
        />
        Show only active todos
      </label>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAddClick}>
        Add
      </button>
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ? <s>{todo.text}</s> : todo.text}
          </li>
        ))}
      </ul>
    </>
  );
}
```

```js todos.js
let nextId = 0;
let calls = 0;

export function getVisibleTodos(todos, showActive) {
  console.log(`getVisibleTodos() was called ${++calls} times`);
  const activeTodos = todos.filter(todo => !todo.completed);
  const visibleTodos = showActive ? activeTodos : todos;
  return visibleTodos;
}

export function createTodo(text, completed = false) {
  return {
    id: nextId++,
    text,
    completed
  };
}

export const initialTodos = [
  createTodo('Get apples', true),
  createTodo('Get oranges', true),
  createTodo('Get carrots'),
];
```

```css
label { display: block; }
input { margin-top: 10px; }
```

</Sandpack>

<Solution>

state 変数と Effect を削除し、代わりに `getVisibleTodos()` の呼び出し結果をキャッシュする `useMemo` 呼び出しを追加します。

<Sandpack>

```js
import { useState, useMemo } from 'react';
import { initialTodos, createTodo, getVisibleTodos } from './todos.js';

export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);
  const [text, setText] = useState('');
  const visibleTodos = useMemo(
    () => getVisibleTodos(todos, showActive),
    [todos, showActive]
  );

  function handleAddClick() {
    setText('');
    setTodos([...todos, createTodo(text)]);
  }

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={showActive}
          onChange={e => setShowActive(e.target.checked)}
        />
        Show only active todos
      </label>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAddClick}>
        Add
      </button>
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ? <s>{todo.text}</s> : todo.text}
          </li>
        ))}
      </ul>
    </>
  );
}
```

```js todos.js
let nextId = 0;
let calls = 0;

export function getVisibleTodos(todos, showActive) {
  console.log(`getVisibleTodos() was called ${++calls} times`);
  const activeTodos = todos.filter(todo => !todo.completed);
  const visibleTodos = showActive ? activeTodos : todos;
  return visibleTodos;
}

export function createTodo(text, completed = false) {
  return {
    id: nextId++,
    text,
    completed
  };
}

export const initialTodos = [
  createTodo('Get apples', true),
  createTodo('Get oranges', true),
  createTodo('Get carrots'),
];
```

```css
label { display: block; }
input { margin-top: 10px; }
```

</Sandpack>

この変更により、 `getVisibleTodos()` は `todos` または `showActive` が変化した場合にのみ呼び出されるようになります。input へのタイピングは state 変数 `text` を変更するだけなので、 `getVisibleTodos()` の呼び出しのトリガーにはなりません。

また、`useMemo` を必要としない別の解決策もあります。state 変数 `text` が ToDo のリストに影響を与えることはありえないので、`NewTodo` フォームを別のコンポーネントに取り出して、state 変数 `text` をその中に移動させることができます：

<Sandpack>

```js
import { useState, useMemo } from 'react';
import { initialTodos, createTodo, getVisibleTodos } from './todos.js';

export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);
  const visibleTodos = getVisibleTodos(todos, showActive);

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={showActive}
          onChange={e => setShowActive(e.target.checked)}
        />
        Show only active todos
      </label>
      <NewTodo onAdd={newTodo => setTodos([...todos, newTodo])} />
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ? <s>{todo.text}</s> : todo.text}
          </li>
        ))}
      </ul>
    </>
  );
}

function NewTodo({ onAdd }) {
  const [text, setText] = useState('');

  function handleAddClick() {
    setText('');
    onAdd(createTodo(text));
  }

  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAddClick}>
        Add
      </button>
    </>
  );
}
```

```js todos.js
let nextId = 0;
let calls = 0;

export function getVisibleTodos(todos, showActive) {
  console.log(`getVisibleTodos() was called ${++calls} times`);
  const activeTodos = todos.filter(todo => !todo.completed);
  const visibleTodos = showActive ? activeTodos : todos;
  return visibleTodos;
}

export function createTodo(text, completed = false) {
  return {
    id: nextId++,
    text,
    completed
  };
}

export const initialTodos = [
  createTodo('Get apples', true),
  createTodo('Get oranges', true),
  createTodo('Get carrots'),
];
```

```css
label { display: block; }
input { margin-top: 10px; }
```

</Sandpack>

このやり方も要件を満たしています。入力に文字を入力すると、state 変数 `text` のみが更新されます。state 変数 `text` は子コンポーネントである `NewTodo` にあるので、親コンポーネントの `TodoList` は再描画されることがありません。これが、文字を入力しても `getVisibleTodos()` が呼び出されない理由です。(別の理由で `TodoList` が再レンダリングされた場合は呼び出されます。)

</Solution>

### Effect を使わずに state をリセットする {/*reset-state-without-effects*/}

`EditContact` コンポーネントは、 `{ id, name, email }` のような形の連絡先オブジェクトを `savedContact` props として受け取ります。名前とメールアドレスの入力フィールドを編集してみてください。保存を押すと、フォームの上にある連絡先のボタンが編集された名前に更新されます。Reset を押すと、保留中のフォームの変更がすべて破棄されます。この UI を実際に操作してみて、感覚をつかんでください。

上部のボタンで連絡先を選択すると、フォームがリセットされ、その連絡先の詳細が反映されます。これは `EditContact.js` 内の Effect によって行われます。この Effect を削除してください。また、`savedContact.id`が変更されたときにフォームをリセットする別の方法を探してください：

<Sandpack>

```js App.js hidden
import { useState } from 'react';
import ContactList from './ContactList.js';
import EditContact from './EditContact.js';

export default function ContactManager() {
  const [
    contacts,
    setContacts
  ] = useState(initialContacts);
  const [
    selectedId,
    setSelectedId
  ] = useState(0);
  const selectedContact = contacts.find(c =>
    c.id === selectedId
  );

  function handleSave(updatedData) {
    const nextContacts = contacts.map(c => {
      if (c.id === updatedData.id) {
        return updatedData;
      } else {
        return c;
      }
    });
    setContacts(nextContacts);
  }

  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={selectedId}
        onSelect={id => setSelectedId(id)}
      />
      <hr />
      <EditContact
        savedContact={selectedContact}
        onSave={handleSave}
      />
    </div>
  )
}

const initialContacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```

```js ContactList.js hidden
export default function ContactList({
  contacts,
  selectedId,
  onSelect
}) {
  return (
    <section>
      <ul>
        {contacts.map(contact =>
          <li key={contact.id}>
            <button onClick={() => {
              onSelect(contact.id);
            }}>
              {contact.id === selectedId ?
                <b>{contact.name}</b> :
                contact.name
              }
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```js EditContact.js active
import { useState, useEffect } from 'react';

export default function EditContact({ savedContact, onSave }) {
  const [name, setName] = useState(savedContact.name);
  const [email, setEmail] = useState(savedContact.email);

  useEffect(() => {
    setName(savedContact.name);
    setEmail(savedContact.email);
  }, [savedContact]);

  return (
    <section>
      <label>
        Name:{' '}
        <input
          type="text"
          value={name}
          onChange={e => setName(e.target.value)}
        />
      </label>
      <label>
        Email:{' '}
        <input
          type="email"
          value={email}
          onChange={e => setEmail(e.target.value)}
        />
      </label>
      <button onClick={() => {
        const updatedData = {
          id: savedContact.id,
          name: name,
          email: email
        };
        onSave(updatedData);
      }}>
        Save
      </button>
      <button onClick={() => {
        setName(savedContact.name);
        setEmail(savedContact.email);
      }}>
        Reset
      </button>
    </section>
  );
}
```

```css
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li { display: inline-block; }
li button {
  padding: 10px;
}
label {
  display: block;
  margin: 10px 0;
}
button {
  margin-right: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

<Hint>

React に、`savedContact.id` が異なる場合、`EditContact` フォームは概念的に_異なる連絡先のフォーム_であり、state を保持すべきではないと伝える方法があればいいのですが、覚えているでしょうか？

</Hint>

<Solution>

`EditContact` コンポーネントを二つに分割します。すべてのフォームの state を内側の `EditForm` コンポーネントに移動します。外側の `EditContact` コンポーネントをエクスポートし、 `savedContact.id` を `key` として内側の `EditContact` コンポーネントに渡します。その結果、内側の `EditForm` コンポーネントはフォームの state をすべてリセットし、別の連絡先を選択するたびに DOM を再作成します。

<Sandpack>

```js App.js hidden
import { useState } from 'react';
import ContactList from './ContactList.js';
import EditContact from './EditContact.js';

export default function ContactManager() {
  const [
    contacts,
    setContacts
  ] = useState(initialContacts);
  const [
    selectedId,
    setSelectedId
  ] = useState(0);
  const selectedContact = contacts.find(c =>
    c.id === selectedId
  );

  function handleSave(updatedData) {
    const nextContacts = contacts.map(c => {
      if (c.id === updatedData.id) {
        return updatedData;
      } else {
        return c;
      }
    });
    setContacts(nextContacts);
  }

  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={selectedId}
        onSelect={id => setSelectedId(id)}
      />
      <hr />
      <EditContact
        savedContact={selectedContact}
        onSave={handleSave}
      />
    </div>
  )
}

const initialContacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```

```js ContactList.js hidden
export default function ContactList({
  contacts,
  selectedId,
  onSelect
}) {
  return (
    <section>
      <ul>
        {contacts.map(contact =>
          <li key={contact.id}>
            <button onClick={() => {
              onSelect(contact.id);
            }}>
              {contact.id === selectedId ?
                <b>{contact.name}</b> :
                contact.name
              }
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```js EditContact.js active
import { useState } from 'react';

export default function EditContact(props) {
  return (
    <EditForm
      {...props}
      key={props.savedContact.id} 
    />
  );
}

function EditForm({ savedContact, onSave }) {
  const [name, setName] = useState(savedContact.name);
  const [email, setEmail] = useState(savedContact.email);

  return (
    <section>
      <label>
        Name:{' '}
        <input
          type="text"
          value={name}
          onChange={e => setName(e.target.value)}
        />
      </label>
      <label>
        Email:{' '}
        <input
          type="email"
          value={email}
          onChange={e => setEmail(e.target.value)}
        />
      </label>
      <button onClick={() => {
        const updatedData = {
          id: savedContact.id,
          name: name,
          email: email
        };
        onSave(updatedData);
      }}>
        Save
      </button>
      <button onClick={() => {
        setName(savedContact.name);
        setEmail(savedContact.email);
      }}>
        Reset
      </button>
    </section>
  );
}
```

```css
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li { display: inline-block; }
li button {
  padding: 10px;
}
label {
  display: block;
  margin: 10px 0;
}
button {
  margin-right: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

</Solution>

### Effect を使わずにフォームを送信する {/*submit-a-form-without-effects*/}

この `Form` コンポーネントでは、友人にメッセージを送信することができます。フォームを送信すると、state 変数 `showForm` が `false` に設定されます。これにより、 `sendMessage(message)` を呼び出す Effect がトリガーされ、メッセージが送信されます（コンソールで確認することができます）。メッセージが送信された後、「Thank you」ダイアログと「Open chat」ボタンが表示され、フォームに戻ることができます。

あなたのアプリのユーザーは、あまりにも多くのメッセージを送信しています。チャットを少し難しくするために、フォームではなく「Thank you」ダイアログを *最初に* 表示することにしました。state 変数 `showForm` を `true` ではなく `false` に初期化するように変更します。この変更を行うとすぐに、コンソールに空のメッセージが 2 回送信されたことが表示されます。このロジックの何かが間違っているのです！

この問題の根本的な原因は何でしょうか？そして、どうすれば修正できるのでしょうか？

<Hint>

メッセージは、ユーザーが「Thank you」ダイアログを見たから送信されるべきなのでしょうか？それとも、別の理由でしょうか？
</Hint>

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function Form() {
  const [showForm, setShowForm] = useState(true);
  const [message, setMessage] = useState('');

  useEffect(() => {
    if (!showForm) {
      sendMessage(message);
    }
  }, [showForm, message]);

  function handleSubmit(e) {
    e.preventDefault();
    setShowForm(false);
  }

  if (!showForm) {
    return (
      <>
        <h1>Thanks for using our services!</h1>
        <button onClick={() => {
          setMessage('');
          setShowForm(true);
        }}>
          Open chat
        </button>
      </>
    );
  }

  return (
    <form onSubmit={handleSubmit}>
      <textarea
        placeholder="Message"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit" disabled={message === ''}>
        Send
      </button>
    </form>
  );
}

function sendMessage(message) {
  console.log('Sending message: ' + message);
}
```

```css
label, textarea { margin-bottom: 10px; display: block; }
```

</Sandpack>

<Solution>

state 変数 `showForm` は、フォームを表示するか、「Thank you」ダイアログを表示するかを決定します。しかし、「Thank you」ダイアログが表示されたからメッセージを送信するわけではありません。誤解を招きやすい Effect を削除し、`sendMessage`の呼び出しを `handleSubmit` イベントハンドラの中に移動させます：

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function Form() {
  const [showForm, setShowForm] = useState(true);
  const [message, setMessage] = useState('');

  function handleSubmit(e) {
    e.preventDefault();
    setShowForm(false);
    sendMessage(message);
  }

  if (!showForm) {
    return (
      <>
        <h1>Thanks for using our services!</h1>
        <button onClick={() => {
          setMessage('');
          setShowForm(true);
        }}>
          Open chat
        </button>
      </>
    );
  }

  return (
    <form onSubmit={handleSubmit}>
      <textarea
        placeholder="Message"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit" disabled={message === ''}>
        Send
      </button>
    </form>
  );
}

function sendMessage(message) {
  console.log('Sending message: ' + message);
}
```

```css
label, textarea { margin-bottom: 10px; display: block; }
```

</Sandpack>

このバージョンでは、_フォームの投稿_（イベント）だけがメッセージの送信を引き起こすことに注意してください。`showForm` が `true` と `false` のどちらに初期設定されていても、同じように動作します。(`false` に設定しても、余計なコンソールメッセージは表示されません。)

</Solution>

</Challenges>

