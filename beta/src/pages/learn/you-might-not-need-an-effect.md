---
title: 'Effectは必要ないかもしれない'
---

<Intro>

Effectは、Reactのパラダイムからの脱出口です。Reactの外に出て、コンポーネントを非Reactウィジェット、ネットワーク、ブラウザDOMなどの外部システムと同期させることができます。外部システムが関与しない場合（例えば、propsやstateが変化したときにコンポーネントのstateを更新したい場合）、Effectは必要ありません。不要なEffectを削除することで、コードが分かりやすくなり、実行速度が向上し、エラーの発生が少なくなります。

</Intro>

<YouWillLearn>

* コンポーネントから不要なEffectを取り除く理由と方法
* Effectを使用せずにコストの高い計算をキャッシュする方法
* Effectを使用せずにコンポーネントのstateをリセットし調整する方法
* イベントハンドラ間でロジックを共有する方法
* どのロジックをイベントハンドラに移すべきか
* 親コンポーネントに変更を通知する方法

</YouWillLearn>

## 不要なEffectを取り除く方法 {/*how-to-remove-unnecessary-effects*/}

Effectが不要なケースは、よく2つあります。

* **レンダリングのためのデータ変換にEffectsは必要ありません。** 例えば、リストを表示する前にフィルタをかけたい場合、リストが変化したときにstate変数を更新するEffectを書きたくなるかもしれません。しかし、これは非効率的です。コンポーネントのstateを更新するとき、Reactはまずコンポーネントの関数を呼び出して、画面に表示されるべき内容を計算します。次に、Reactはこれらの変更をDOMに["commit"](/learn/render-and-commit)し、画面を更新するのです。その後、ReactはEffectを実行します。もし、Effectもすぐにstateを更新してしまうと、プロセス全体を一からやり直すことになります。不要なレンダーパスを避けるには、コンポーネントの最上位ですべてのデータを変換します。このコードは、propsやstateが変更されるたびに、自動的に再実行されます。
* **ユーザーイベントの処理にEffectsは必要ありません。** 例えば、`/api/buy` の POST リクエストを送信し、ユーザーが製品を購入したときに通知を表示したいとします。購入ボタンのクリックイベントハンドラでは、何が起こったかを正確に知ることができます。Effectが実行される頃には、ユーザが何をしたのか（例えば、どのボタンがクリックされたのか）、わかりません。このため、通常ユーザーイベントは対応するイベントハンドラで処理します。

しかし、外部システムと[同期](/learn/synchronizing-with-effects#what-are-effects-and-how-are-they-different-from-events))するときにはEffectが必要です。例えば、jQueryのウィジェットをReactのstateと同期させるEffectを書くことができます。また、Effectでデータを取得することもできます。たとえば、検索結果を現在の検索クエリに同期させることができます。最近の[フレームワーク](/learn/start-a-new-react-project#building-with-a-full-featured-framework)では、コンポーネントに直接Effectを記述するより効率的な組み込みデータ取得機能を提供しているので覚えておいてください。

正しい感覚を得るために、いくつかの一般的な具体例を見てみましょう！

### propsやtateに基づくstateの更新 {/*updating-state-based-on-props-or-state*/}

例えば、`firstName` と `lastName` という2つのstate変数を持つコンポーネントがあるとします。それらを連結して、`fullName`を計算したいとします。さらに、 `firstName` や `lastName` が変更されるたびに、 `fullName` が更新されるようにしたいと思います。最初の直感は、`fullName`のstate変数を追加して、Effectでそれを更新することかもしれません：

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

これは必要以上に複雑です。また、非効率的でもあります。`fullName`の値が古いまま全てのレンダリングを行い、その後すぐに更新された値で再レンダリングします。state変数とEffectの両方を削除してください：

```js {4-5}
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  // ✅ Good: calculated during rendering
  const fullName = firstName + ' ' + lastName;
  // ...
}
```

**すでにあるpropsやstateから値が計算できる場合、 [stateを使用しないでください](/learn/choosing-the-state-structure#avoid-redundant-state) その代わり、レンダリング中に値を計算します。** これにより、（余分な更新「cascading updates」を回避して）コードが速くなり、（いくつかのコードを削除できるので）シンプルになり、（異なるstate変数が互いに同期しないことによって引き起こされるバグを回避して）エラーが起こりにくくなります。このアプローチがなじまない場合は、[Thinking in React](/learn/thinking-in-react#step-3-find-the-minimal-but-complete-representation-of-ui-state) に、stateに入れるべきものについてのガイダンスがあります。

### コストの高い計算をキャッシュする {/*caching-expensive-calculations*/}

このコンポーネントは、propsで受け取った `todos` を `filter` propsに従ってフィルタリングすることで、 `visibleTodos` を計算します。この結果をstate変数に格納し、Effectで更新したくなるかもしれません：

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

先の例と同様、これは不要かつ非効率的です。まず、stateとEffectを削除します：

```js {3-4}
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // ✅ This is fine if getFilteredTodos() is not slow.
  const visibleTodos = getFilteredTodos(todos, filter);
  // ...
}
```

多くの場合、このコードで大丈夫です! しかし、もしかしたら `getFilteredTodos()` が遅いかもしれないし、たくさんの `todos` があるかもしれません。そのような場合、`newTodo` のような無関係なstate変数が変化したときに `getFilteredTodos()` を再計算したくありません。

[`useMemo`](/apis/usememo) フックでラップすることで、コストの高い計算をキャッシュ(または ["memoize"](https://en.wikipedia.org/wiki/Memoization) )することができます。

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

**これは `todos` か `filter` のどちらかが変更されない限り、内部関数を再実行させないことを React に伝えます。** React は最初のレンダリング時に `getFilteredTodos()` の戻り値を記憶しています。次のレンダリングでは、`todos` または `filter` が変わっていないかどうかを確認します。もし前回と同じであれば、`useMemo`は最後に保存した結果を返します。しかし、異なる場合は、React はラップした関数を再度呼び出します (そして、代わりにその結果を保存します)。

[`useMemo`](/apis/usememo) でラップした関数はレンダリング中に実行されるので、これは [純粋な計算](/learn/keeping-components-pure) でのみ動作します。

<DeepDive title="コストの高い計算かどうかを判断する方法">

一般的に、何千ものオブジェクトを作成したりループさせたりしない限り、おそらく高いコストではないでしょう。より確信を得たい場合は、コンソールログを追加して、コードの一部で費やされた時間を測定することができます：

```js {1,3}
console.time('filter array');
const visibleTodos = getFilteredTodos(todos, filter);
console.timeEnd('filter array');
```

測定したいインタラクションを実行します (たとえば、inputに入力します)。すると、コンソールに `filter array.0.15ms` のようなログが表示されます。もし、記録された時間全体がかなりの量(`1ms`以上)になるのであれば 、その計算をメモしておくことが意味を持つかもしれません。実験として、その計算を `useMemo` でラップして、そのインタラクションで記録される時間の合計が減少したかどうかを検証することができます。

```js
console.time('filter array');
const visibleTodos = useMemo(() => {
  return getFilteredTodos(todos, filter); // Skipped if todos and filter haven't changed
}, [todos, filter]);
console.timeEnd('filter array');
```

`useMemo` は *最初の* レンダリングを速くするものではありません。更新時の不要な作業を省略できるようになるだけです。

あなたのマシンがおそらくユーザーよりも高速であることを念頭に置き、人工的なスローダウンでパフォーマンスをテストするのは良いアイデアです。例えば、Chromeではこのために[CPU Throttling](https://developer.chrome.com/blog/new-in-devtools-61/#throttling)というオプションが用意されています。

また、開発中のパフォーマンス測定は、最も正確な結果を得られないことに注意してください。(例えば、[Strict Mode](/apis/strictmode) がオンの場合、各コンポーネントは一度ではなく二度レンダリングされます)。最も正確なタイミングを得るには、実稼働環境向けにアプリを構築し、ユーザーが持つようなデバイスでテストしてください。

</DeepDive>

### propsが変更されたときにすべてのstateをリセットする {/*resetting-all-state-when-a-prop-changes*/}

この `ProfilePage` コンポーネントは `userId` propsを受け取ります。このページはコメント入力を含んでおり、その値を保持するために `comment` state変数を使用します。ある日、あなたは問題に気づきました。あるプロファイルから別のプロファイルに移動するとき、`comment` のstateがリセットされないのです。その結果、誤って間違ったユーザーのプロファイルにコメントを投稿してしまうことがあります。この問題を解決するために、 `userId` が変更されるたびに `comment` state変数をクリアするようにします。

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

これは非効率的です。なぜなら `ProfilePage` とその子要素は、まず古い値でレンダリングし、それからもう一度レンダリングするからです。また、`ProfilePage` の内部にstateを持つ *すべての* コンポーネントでこれを行う必要があるため、複雑です。例えば、コメント UI がネストされている場合、ネストされたコメントのstateもクリアしたいと思うでしょう。

その代わりに、明示的にキーを与えることで、各ユーザーのプロファイルが概念的に_異なる_プロファイルであることをReactに伝えることができます。コンポーネントを2つに分割し、外側のコンポーネントから内側のコンポーネントに `key` 属性を渡します。

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

通常、React は同じコンポーネントを同じ場所にレンダリングする場合、そのstateを保持します。**キーとして `userId` を `Profile` コンポーネントに渡すことで、異なる `userId` を持つ二つの `Profile` コンポーネントを、stateを共有しない二つの異なるコンポーネントとして扱うように React に要求しています**。 キーが変わるたびに、React は DOM を再作成して `Profile` コンポーネントとその子コンポーネント全ての [stateをリセット] (/learn/preserving-and-resetting-state#option-2-resetting-state-with-a-key) します。その結果、プロファイル間を移動する際に、`comment`フィールドは自動的にクリアされます。

この例では、外側の `ProfilePage` コンポーネントのみがエクスポートされ、プロジェクト内の他のファイルから見えるようになっていることに注意してください。`ProfilePage` をレンダリングするコンポーネントは、キーを渡す必要はありません。通常のpropsとして `userId` を渡します。`ProfilePage` が `key` として内部の `Profile` コンポーネントに渡すのは、実装の詳細のためです。

### propsが変更されたときにstateを調整する {/*adjusting-some-state-when-a-prop-changes*/}

時には、propsが変更されたときに、すべてのstateではなく、一部のstateをリセットしたり、調整したりしたい場合があります。

この `List` コンポーネントは `items` のリストを prop として受け取り、選択されたアイテムを `selection` state変数に保持します。`items` propsが異なる配列を受け取るたびに、 `selection` を `null` にリセットしたいと思います：

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

まずは、Effectを削除することから始めましょう。その代わり、レンダリング中に直接stateを調整します。

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

このような[過去のレンダリング情報の保存](/apis/usestate#storing-information-from-previous-renders) は理解しにくいかもしれませんが、Effectで同じstateを更新するよりはましでしょう。上記の例では、レンダリング中に `setSelection` が直接呼び出されます。React は `return` 文で終了した後、`List` を *直ちに* 再レンダリングします。その時点では、React は `List` の子要素をレンダリングしておらず、DOM も更新していないので、`List` の子要素は古い `selection` 値のレンダリングをスキップすることができます。

レンダリング中にコンポーネントを更新すると、React は返された JSX を捨て、直ちにレンダリングを再試行します。非常に遅い再試行を避けるために、React はレンダリング中に *同じ* コンポーネントのstateのみを更新できるようにしています。レンダリング中に別のコンポーネントのstateを更新すると、エラーが表示されます。ループを避けるために、`items !== prevItems` のような条件が必要です。このようにstateを調整することはできますが、その他の副作用（DOMの変更やタイムアウトの設定など）はイベントハンドラやEffectに残して、[コンポーネントを予測可能に保つ]（/learn/keep-components-pure）ようにしてください。

**このパターンはEffectよりも効率的ですが、ほとんどのコンポーネントは必要としないでしょう。**どのように行うにしても、propsや他のstateに基づいてstateを調整することは、データフローの理解とデバッグを難しくします。代わりに、[キーですべてのstateをリセットする](#resetting-all-state-when-a-prop-changes) か [レンダリング中にすべてを計算](#updating-state-based-on-props-or-state) が出来ないかを常にチェックするようにして下さい。たとえば、選択された *項目* を保存する（そしてリセットする）代わりに、選択された *項目ID:* を保存することができます：

```js {3-5}
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);
  // ✅ Best: Calculate everything during rendering
  const selection = items.find(item => item.id === selectedId) ?? null;
  // ...
}
```

これで、stateを「調整」する必要は全くありません。選択された ID を持つアイテムがリストにあれば、それは選択されたままです。そうでない場合は、レンダリング時に計算される `selection` は、マッチするアイテムが見つからなかったので `null` になります。この動作は少し異なりますが、 `items` に対するほとんどの変更が選択stateを保持するようになったので、間違いなくこの動作の方が優れています。しかし、 `selectedId` を持つアイテムが存在しないかもしれないので、以下のすべてのロジックで `selection` を使用する必要があります。

### イベントハンドラ間のロジックの共有 {/*sharing-logic-between-event-handlers*/}

例えば、商品を購入するための2つのボタン(BuyとCheckout)がある商品ページがあるとします。ユーザーが商品をカートに入れるたびに、[トースト](https://uxdesign.cc/toasts-or-snack-bars-design-organic-system-notifications-1236f2883023)通知を表示したいとします。両方のボタンのクリックハンドラに `showToast()` を追加すると、繰り返しになるので、このロジックを Effect に配置したくなるかもしれません：

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

このEffectは不要です。また、バグの原因になる可能性も高いです。例えば、ページが再読み込みされる間に、アプリがショッピングカートを「記憶」しているとします。一度商品をカートに入れ、ページを更新すると、トースト通知が再び表示されます。その商品のページを更新するたびに表示され続けることになります。これは、ページロード時に `product.isInCart` が既に `true` になっているためで、上記の Effect は `showToast()` を呼び出すことになります。

**あるコードがEffectの中にあるべきか、イベントハンドラの中にあるべきか迷ったときは、このコードがなぜ実行される必要があるのか、自問自答してください。** この例では、トーストが表示されるのは、ユーザーがボタンを押したからであり、商品ページが表示されたからではありません。Effectを削除し、共有ロジックを関数に入れ、両方のイベントハンドラから呼び出すようにします：

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

これにより、不要なEffectが削除され、バグが修正されます。

### POST リクエストを送信する {/*sending-a-post-request*/}

この `Form` コンポーネントは2種類の POST リクエストを送信します。マウント時にAnalyticsイベントを送信します。フォームに入力して Submit ボタンをクリックすると、 `/api/register` エンドポイントに POST リクエストが送信されます。

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

アナリティクスのPOSTリクエストは、Effectのままであるべきです。なぜなら、analytics イベントを送信する _理由_ は、フォームが表示されたことだからです。(開発環境では2回発火してしまいますが、その対処法は [こちら](/learn/synchronizing-with-effects#sending-analytics) を参照してください)。

しかし、`/api/register` POSTリクエストは、フォームが _表示される_ ことによって発生するわけではありません。ユーザーがボタンを押したときという、ある特定の瞬間にだけリクエストを送りたいのです。これは特定のインタラクションでのみ発生するはずです。2つ目のEffectを削除し、POSTリクエストをイベントハンドラに移動します：

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

あるロジックをイベントハンドラに入れるか、Effectに入れるかを選択するとき、答える必要がある主な質問は、ユーザーの視点から見て、それがどのようなロジックであるかということです。もし、このロジックが特定のインタラクションによって引き起こされるものであれば、イベントハンドラに格納します。もし、ユーザーが画面上のコンポーネントを見ることによって引き起こされるのであれば、Effectに記述してください。

### アプリケーションの初期化 {/*initializing-the-application*/}

アプリのロード時に一度だけ実行されるロジックがあります。そのようなロジックは、トップレベル・コンポーネントのEffectに配置します：

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

しかし、このロジックが[開発中に2回実行される](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)ことがすぐに発覚します。この関数は2回呼び出されるようには設計されていないため、認証トークンを無効にしてしまうなどの問題を引き起こします。一般に、コンポーネントは再マウントされたときに回復力を持つべきです。これには、トップレベルの `App` コンポーネントも含まれます。実際の運用では再マウントされることはないかもしれませんが、すべてのコンポーネントで同じ制約に従うことで、コードの移動と再利用が容易になります。もし、あるロジックが *コンポーネントマウントに一度*ではなく、*アプリロードに一度* 実行されなければならない場合、トップレベルの変数を追加して、それが既に実行されたかどうかを追跡し常に再実行をスキップすることができます：

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

トップレベルのコードは、たとえそれが最終的にレンダリングされないとしても、コンポーネントがインポートされたときに一度だけ実行されます。任意のコンポーネントをインポートしたときの速度低下や驚くべき動作を避けるために、このパターンを使い過ぎないようにしましょう。アプリ全体の初期化ロジックは `App.js` のようなルートコンポーネントモジュールか、アプリケーションのエントリポイントモジュールに留めておいてください。

### stateの変更を親コンポーネントに通知する {/*notifying-parent-components-about-state-changes*/}

例えば、`Toggle` コンポーネントを書いていて、内部に `isOn` state (`true` か `false` のどちらか) があるとします。stateを切り替えるには、(クリックやドラッグなど)いくつかの方法があります 。そこで、`onChange` イベントを公開し、Effect から呼び出すことにします：

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

先ほどと同様に、これは理想的ではありません。まず `Toggle` がstateを更新し、React が画面を更新します。次に、React は Effect を実行し、親コンポーネントから渡された `onChange` 関数を呼び出します。ここで、親コンポーネントは自身のstateを更新し、別のレンダリングパスを開始します。すべてを1つのパスで行う方が良いでしょう。

Effectを削除して、代わりに同じイベントハンドラで*両方の*コンポーネントのstateを更新します：

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

この方法では、`Toggle`コンポーネントとその親コンポーネントの両方が、イベント中にstateを更新します。React は異なるコンポーネントからの更新を [バッチ処理] (/learn/queueing-a-series-of-state-updates) でまとめて行うので、結果としてレンダーパスは 1 回だけとなります。

また、stateを完全に削除して、代わりに親コンポーネントから `isOn` を受け取ることも可能かもしれません：

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

[stateをリフトアップ](/learn/sharing-state-between-components) して、親コンポーネント自身のstateを切り替えることで、親コンポーネントが `Toggle` を完全に制御できるようにします。これは、親コンポーネントがより多くのロジックを含む必要があることを意味しますが、心配するようなstateは全体的に少なくなります。2つの異なるstate変数を同期させようとするときはいつでも、代わりにstateをリフトアップするサインです!

### 親にデータを渡す {/*passing-data-to-the-parent*/}

この `Child` コンポーネントは、いくつかのデータを取得し、それを `Parent` コンポーネントに Effect で渡します：

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

Reactでは、データは親コンポーネントから子コンポーネントに流れます。画面上で何か不具合があったとき、どのコンポーネントが間違ったpropを渡しているか、あるいは間違ったstateになっているかを見つけるまで、コンポーネントチェーンをさかのぼることで、その情報がどこから来たかを追跡することができます。子コンポーネントがEffectで親コンポーネントのstateを更新する場合、データの流れを追跡するのは非常に難しくなります。子コンポーネントと親コンポーネントの両方が同じデータを必要とするので、親コンポーネントにそのデータを取得させ、代わりに子コンポーネントに*pass down*するようにします：

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

これはよりシンプルで、データの流れを予測しやすくします。データは親から子へと流れます。

### 外部ストアを購読する {/*subscribing-to-an-external-store*/}

時には、あなたのコンポーネントはReactのstateの外側にあるデータを購読する必要があるかもしれません。このデータは、サードパーティライブラリや組み込みのブラウザAPIからのものである可能性があります。このデータはReactが知らないうちに変更される可能性があるので、コンポーネントを手動で購読する必要があります。これはEffectで行われることが多く、以下のような例になります：

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

ここでは、コンポーネントは外部のデータストア(この場合はブラウザの `navigator.onLine` API)を購読しています。このAPIはサーバ上に存在しない（最初のHTMLを生成するために使用できない）ので、最初はstateが `true` に設定されています。そのデータストアの値がブラウザで変更されるたびに、コンポーネントはそのstateを更新します。

このためにEffectを使うのが一般的ですが、Reactには外部ストアを購読するための専用のフックがあり、そちらを使うのが望ましいでしょう。Effectを削除して、[`useSyncExternalStore`](/apis/usesyncexternalstore)への呼び出しに置き換えます：

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

この方法は、Effectを使って手動でMutableなデータをReactのstateに同期させるよりもエラーが起こりにくいです。一般的には、上記の `useOnlineStatus()` のようなカスタムフックを書くことで、個々のコンポーネントでこのコードを繰り返す必要がないようにします。Reactコンポーネントから外部ストアを購読する方法については[こちら](/apis/usesyncexternalstore)を参照してください。

### データの取得 {/*fetching-data*/}

多くのアプリでは、Effectを使ってデータの取得を開始します。このようにデータ取得のEffectを書くことはよくあることです：

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

これは、イベントハンドラにロジックを入れる必要があった先ほどの例と矛盾しているように思えるかもしれません。しかし、取得する主な理由は、*タイピングイベントではないこと*を考慮してください。検索入力は多くの場合、URLからあらかじめ入力されており、ユーザーは入力に触れることなく、戻ったり進んだりすることがあります。`page` と `query` がどこから来るかは重要ではありません。このコンポーネントが表示されている間は、現在の `page` と `query` に従って、ネットワークからのデータで `results` を [同期](/learn/synchronizing-with-effects) しておきたいものです。これがEffectである理由です。

しかし、上のコードにはバグがあります。例えば、あなたが `"hello"` と速くタイプしたとする。すると、`query` が `"h"` から `"he"`, `"hel"`, `"hell"`, そして `"hello"` と変化していくでしょう。これは別々の取得を開始しますが、どの順番で応答が到着するかは保証されません。例えば、`"hell"` のレスポンスは `"hello"` のレスポンスの *後* に到着するかもしれません。`setResults()` を最後に呼び出すので、間違った検索結果が表示されることになります。これは ["race condition"](https://en.wikipedia.org/wiki/Race_condition) と呼ばれます。2つの異なるリクエストがお互いに「競争」して、期待とは異なる順番でやってくるのです。

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

データ取得の実装で難しいのは、レースコンディションの処理だけではありません。レスポンスをキャッシュする方法（ユーザーが Back をクリックすると、スピナーの代わりに前の画面がすぐに表示されるように）、サーバーで取得する方法（サーバーでレンダリングされた最初の HTML がスピナーの代わりに取得したコンテンツを含むように）、ネットワークのウォーターフォールを回避する方法（データを取得する必要がある子コンポーネントが取得を始める前にその上のすべての親のデータ取得を待つ必要がないように）も考えたいかも知れません。**これらの問題は、Reactだけでなく、どのUIライブラリにも当てはまります。これらを解決するのは簡単ではありません。そのため、最近の[フレームワーク](/learn/start-a-new-react-project#building with-a-full-featured-framework) では、コンポーネントで直接Effectを記述するより効率の良い組み込みデータ取得機能を提供しています**。

フレームワークを使っていない(そして自分で作りたくない)が、Effectからのデータ取得をより人間工学的に行いたい場合、以下のように取得ロジックをカスタムHookに抽出することを検討してください：

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

また、エラー処理のためのロジックや、コンテンツがロードされているかどうかを追跡するためのロジックも追加したいと思うでしょう。このようなフックを自分で作ることもできますし、Reactのエコシステムですでに利用可能な多くのソリューションのうちの一つを使うこともできます。**この方法だけではフレームワークの組み込みデータ取得機能ほど効率的ではありませんが、データ取得ロジックをカスタムフックに移動することで、後で効率的なデータ取得戦略を採用することが容易になります**。

一般的に、Effect を書かなければならないときはいつでも、上記の `useData` のように、より宣言的で目的の API を持つカスタムフック に機能の一部を抽出することができるかどうか、目を光らせておいてください。コンポーネントの中で生の `useEffect` の呼び出しが少なければ少ないほど、アプリケーションのメンテナンスが容易になります。

<Recap>

- レンダリング中に何かを計算できるのであれば、Effect は必要ありません。
- コストの高い計算をキャッシュするには、`useMemo` を`useEffect`の代わりに追加します。
- コンポーネントツリー全体のstateをリセットするには、別の `key` を渡します。
- propsの変更に応じて特定のstateをリセットするには、レンダリング中に行います。
- コンポーネントが *表示* されたために実行する必要があるコードは Effect に、それ以外はイベントに記述する必要があります。
- 複数のコンポーネントのstateを更新する必要がある場合は、1つのイベントの中で行う方がよいでしょう。
- 異なるコンポーネントのstate変数を同期させようとするときは、必ずstateを持ち上げることを考えます。
- Effectでデータを取得することは可能ですが、race conditionを避けるためにクリーンアップを実装する必要があります。

</Recap>

<Challenges>

### Effectsを使わずにデータを変換する {/*transform-data-without-effects*/}

下の `TodoList` は、Todo の一覧を表示します。"Show only active todos" チェックボックスをチェックすると、完了したTODOはリストに表示されません。どのTodoが表示されているかにかかわらず、フッターにはまだ完了していないTodoの数が表示されます。

不要なstateとEffectをすべて削除して、このコンポーネントを簡素化しましょう：

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

レンダリング中に何かを計算できるのであれば、stateやそれを更新するEffectは必要ありません。

</Hint>

<Solution>

この例では、`todos`のリストと`showActive`というstate変数が、チェックボックスがチェックされているかどうかを表しています。その他のstate変数は[余計](/learn/choosing-the-state-structure#avoid-redundant-state)で、代わりにレンダリング中に計算することができます。これには `footer` が含まれ、周囲のJSXに直接移動させることができます。

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

### Effectを使わずに計算をキャッシュする {/*cache-a-calculation-without-effects*/}

この例では、Todosのフィルタリングは `getVisibleTodos()` という別の関数に抽出されています。この関数の内部には `console.log()` が含まれており、この関数が呼び出されたことに気づくことができます。"Show only active todos "を切り替えてして、`getVisibleTodos()`が再実行されることに注目してください。これは、表示するTODOを切り替えると、表示されるTODOが変わるので、予想されることです。

あなたの仕事は、`TodoList` コンポーネントの `visibleTodos` リストを再計算する Effect を削除することです。しかし、入力時に `getVisibleTodos()` が再実行されない (つまりログが出力されない) ことを確認する必要があります。

<Hint>

一つの解決策は、 `useMemo` 呼び出しを追加して、表示されているTodosをキャッシュすることです。もうひとつ、あまり目立たない解決策もあります。

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

state変数と Effect を削除し、代わりに `getVisibleTodos()` の呼び出し結果をキャッシュする `useMemo` 呼び出しを追加します。

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

この変更により、 `getVisibleTodos()` は `todos` または `showActive` が変化した場合にのみ呼び出されるようになります。入力へのタイピングはstate変数 `text` を変更するだけなので、 `getVisibleTodos()` の呼び出しのトリガーにはなりません。

また、`useMemo` を必要としない別の解決策もあります。state変数 `text` が ToDo のリストに影響を与えることはありえないので、`NewTodo` フォームを別のコンポーネントに取り出して、state変数 `text` をその中に移動させることができます：

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

このやり方も要件を満たしています。入力に文字を入力すると、state変数 `text` のみが更新されます。state変数 `text` は子コンポーネントである `NewTodo` にあるので、親コンポーネントの `TodoList` は再描画されることがありません。これが、文字を入力しても `getVisibleTodos()` が呼び出されない理由です。(別の理由で `TodoList` が再レンダリングされた場合は呼び出されます。)

</Solution>

### Effectを使用しないstateのリセット {/*reset-state-without-effects*/}

この `EditContact` コンポーネントは、 `{ id, name, email }` のような形のコンタクトオブジェクトを `savedContact` propsとして受け取ります。名前とメールアドレスの入力フィールドを編集してみてください。保存を押すと、フォームの上にあるコンタクトのボタンが編集された名前に更新されます。Reset を押すと、保留中のフォームの変更がすべて破棄されます。このUIは、実際に操作してみて、感覚をつかんでください。

上部のボタンでコンタクトを選択すると、フォームがリセットされ、そのコンタクトの詳細が反映されます。これは `EditContact.js` 内のEffectによって行われます。このEffectを削除してください。また、`savedContact.id`が変更されたときにフォームをリセットする別の方法を探してください：

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

Reactに、`savedContact.id`が異なる場合、`EditContact`フォームは概念的に_異なるコンタクトのフォームであり、stateを保持すべきではないと伝える方法があればいいのですが、そのような方法はありますか？

</Hint>

<Solution>

`EditContact` コンポーネントを二つに分割します。すべてのフォームのstateを内側の `EditForm` コンポーネントに移動します。外側の `EditContact` コンポーネントをエクスポートし、 `savedContact.id` を `key` として内側の `EditContact` コンポーネントに渡します。その結果、内側の `EditForm` コンポーネントはフォームのstateをすべてリセットし、別のコンタクトを選択するたびに DOM を再作成します。

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

### Effectを使わずにフォームを送信する {/*submit-a-form-without-effects*/}

この `Form` コンポーネントでは、友人にメッセージを送信することができます。フォームを送信すると、`showForm` state変数が `false` に設定されます。これにより、 `sendMessage(message)` を呼び出す Effect がトリガーされ、メッセージが送信されます（コンソールで確認することができます）。メッセージが送信された後、"Thank you" ダイアログと "Open chat" ボタンが表示され、フォームに戻ることができます。

あなたのアプリのユーザーは、あまりにも多くのメッセージを送信しています。チャットを少し難しくするために、フォームではなく「ありがとう」ダイアログを *最初に* 表示することにしました。state変数 `showForm` を `true` ではなく `false` に初期化するように変更します。この変更を行うとすぐに、コンソールに空のメッセージが 2 回送信されたことが表示されます。このロジックの何かが間違っているのです!

この問題の根本的な原因は何でしょうか？そして、どうすれば修正できるのでしょうか？

<Hint>

メッセージは、ユーザーが「ありがとう」ダイアログを見たから送信されるべきなのでしょうか？それとも、その逆でしょうか？
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

state変数 `showForm` は、フォームを表示するか、"Thank you" ダイアログを表示するかを決定します。しかし、"Thank you" ダイアログが表示されたからメッセージを送信するわけではありません。誤解を招きやすいEffectを削除し、`sendMessage`の呼び出しを `handleSubmit` イベントハンドラの中に移動させます：

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

このバージョンでは、_submit the form_ (これはイベントです) だけがメッセージの送信を引き起こすことに注意してください。これは、 `showForm` が `true` と `false` のどちらに初期設定されていても、同じように動作します。(`false` に設定すると、余計なコンソールメッセージは表示されません)。

</Solution>

</Challenges>

