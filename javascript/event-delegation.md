# JavaScriptのイベント委譲について

## イベント委譲（Event Delegation）とは

イベント委譲とは、子要素ひとつひとつにイベントリスナーを登録するのではなく、**親要素に1つだけ登録し、子要素で起きたイベントを親側で受け取って処理する**手法です。

イベント委譲は、子要素で起きたイベントが親要素へと伝わっていく「バブリング」という仕組みを利用しています。

この「上に伝わってくる」性質を使って、子のイベントを親でまとめて受け取っています。

## バブリング（Bubbling）とは

クリックなどのイベントは、クリックされた要素だけで処理されるわけではなく、その親要素、さらに親要素……と、DOMツリーを上へ上へと伝わっていきます。

```text
document
  └── body
      └── ul
          └── li
              └── button  ← ここでクリックが発生
```

`button` をクリックすると、イベントは `button` → `li` → `ul` → `body` → `document` の順に上へ伝わっていきます。
泡が下から上へのぼるイメージから、この現象を「バブリング」と呼びます。

経路の途中にある複数の要素にリスナーを登録すると、下から順に発火する様子が確認できます。

```html
<ul id="list">
  <li id="item">
    りんご
    <button id="btn">クリックしてね</button>
  </li>
</ul>
```

```js
const btn = document.getElementById('btn');
const item = document.getElementById('item');
const list = document.getElementById('list');

btn.addEventListener('click', () => console.log('1. button が受け取った'));
item.addEventListener('click', () => console.log('2. li に伝わってきた'));
list.addEventListener('click', () => console.log('3. ul に伝わってきた'));
document.body.addEventListener('click', () => console.log('4. body に伝わってきた'));

// button をクリックすると、コンソールには次の順で出力される:
// 1. button が受け取った
// 2. li に伝わってきた
// 3. ul に伝わってきた
// 4. body に伝わってきた
```

▶ **動くデモ**: [CodePenで開く](https://codepen.io/kakariko/pen/pvRyXNO)

途中の要素にリスナーが無くても、イベントの通り道は変わりません。リスナーが無い要素は素通りし、リスナーが付いている要素だけが順に反応します。

## バブリングを止める

「これ以上、上の要素には伝えたくない」というときは、`event.stopPropagation()` を呼ぶと、そこから先への伝搬が止まります。

```js
list.addEventListener('click', (event) => {
  event.stopPropagation();   // ここで止まるので body より先には伝わらない
  console.log('ul で止める');
});
```

▶ **動くデモ**: [CodePenで開く](https://codepen.io/kakariko/pen/yygOdqr)

## イベント委譲を使わないと起きる問題

イベント委譲が特に役立つのは、**項目が増えたり減ったりするリストやテーブル**です。

「項目を追加でき、各項目に削除ボタンがあるリスト」を例に、まずはイベント委譲を使わずに書いてみます。

```html
<ul id="list">
  <li>りんご <button class="delete">削除</button></li>
  <li>ばなな <button class="delete">削除</button></li>
</ul>
<button id="add">項目を追加</button>
```

```js
// 今ある削除ボタンに、1つずつリスナーを付ける
document.querySelectorAll('.delete').forEach((button) => {
  button.addEventListener('click', () => {
    button.closest('li').remove();   // 押されたボタンの行を削除する
  });
});

// 「項目を追加」ボタンで、新しい li を追加
document.getElementById('add').addEventListener('click', () => {
  const li = document.createElement('li');
  li.innerHTML = 'みかん <button class="delete">削除</button>';
  document.getElementById('list').appendChild(li);
});
```

▶ **動くデモ**: [CodePenで開く](https://codepen.io/kakariko/pen/qERNdog)

このコードでは、最初からあった「りんご」「ばなな」の削除ボタンは動きますが、あとから追加した「みかん」の削除ボタンは反応しません。

リスナーを登録したのは、ページ読み込み時点で存在したボタンだけだからです。あとから増えたボタンには、リスナーが付いていません。

## イベント委譲で解決する

この問題を解決する一つの方法が、**イベント委譲**です。

削除ボタンを押したときのイベントが、親の `<ul>` までバブリングして伝わってくるので、親側で「クリックされたのが削除ボタンか」を確かめて、削除ボタンだったときだけ処理します。

HTMLは先ほどと同じで、変わるのは JavaScript だけです。

```html
<ul id="list">
  <li>りんご <button class="delete">削除</button></li>
  <li>ばなな <button class="delete">削除</button></li>
</ul>
<button id="add">項目を追加</button>
```

```js
const list = document.getElementById('list');

list.addEventListener('click', (event) => {
  // クリックされた位置から、一番近い削除ボタンを探す
  const button = event.target.closest('.delete');

  // 削除ボタン以外（余白など）のクリックは無視する
  if (!button) return;

  button.closest('li').remove();   // 押されたボタンの行を削除する
});

// 「項目を追加」ボタンは、リスナーのことを気にせず li を追加するだけ
document.getElementById('add').addEventListener('click', () => {
  const li = document.createElement('li');
  li.innerHTML = 'みかん <button class="delete">削除</button>';
  list.appendChild(li);
});
```

▶ **動くデモ**: [CodePenで開く](https://codepen.io/kakariko/pen/JoEKdaz)

「項目を追加」する処理の中に、リスナーを登録するコードを書いていませんが、あとから追加した「みかん」の削除ボタンも問題なく動きます。

リスナーは個々のボタンではなく親に付いているので、あとから増やした項目のクリックも、同じ親のリスナーで処理できるためです。

またリスナーは親に1つで済むため、項目の数だけリスナーを登録せずにすみます。

## 補足: 個別に登録して動かすこともできる

イベント委譲を使わない場合でも、**項目を追加するたびに、その項目にもリスナーを登録する**ことで、あとから増やした項目の削除ボタンも動かすことができます。

```js
const list = document.getElementById('list');

// li を作り、その場で削除用のリスナーも付けて返す関数
function createItem(text) {
  const li = document.createElement('li');
  li.innerHTML = `${text} <button class="delete">削除</button>`;
  li.querySelector('.delete').addEventListener('click', () => li.remove());
  return li;
}

// 既存の項目も、上記の関数で作る
['りんご', 'ばなな'].forEach((text) => list.appendChild(createItem(text)));

// 追加も同じ関数を使う → 新しい項目にもリスナーを付与
document.getElementById('add').addEventListener('click', () => {
  list.appendChild(createItem('みかん'));
});
```

▶ **動くデモ**: [CodePenで開く](https://codepen.io/kakariko/pen/RNKRPzr)

ただしこの方法では、同じ処理をボタンの数だけ登録することになり、項目が増えるほどリスナーも1つずつ増えていきます。

## フレームワークでは（React）

これまでのリストの例を React で書くと、次のようになります。各ボタンの `onClick` に処理を書くだけで、リスナーの登録は自分では行いません。

```jsx
function FruitList() {
  const [items, setItems] = useState(['りんご', 'ばなな']);
  const remove = (target) => setItems(items.filter((item) => item !== target));
  const add = () => setItems([...items, 'みかん']);

  return (
    <>
      <ul>
        {items.map((item) => (
          <li key={item}>
            {item}
            <button onClick={() => remove(item)}>削除</button>
          </li>
        ))}
      </ul>
      <button onClick={add}>項目を追加</button>
    </>
  );
}
```

React には、データ（state）が変わるたびに描画をし直す「再レンダリング」という仕組みがあります。

項目を追加すると再レンダリングによって新しいボタンにも `onClick` が自動で付くため、「あとから追加した要素が動かない」問題はそもそも起きず、自分でイベント委譲を書く場面はほとんどありません。

ただし、何もしなくてよいのは **React が内部でイベント委譲を使っている**ためです。各ボタンに個別のリスナーを付けるのではなく、アプリ全体のルート要素に1つだけリスナーを置き、そこから各 `onClick` に振り分けています。つまり `onClick` の裏側で行われているのは、この記事で説明したイベント委譲そのものです。

そのため、バブリングや `event.target`、`stopPropagation()` の理解は、React を使うときにも役立ちます。

## まとめ

イベント委譲は、バブリング（子で起きたイベントが親へ伝わる性質）を利用して、子要素のイベントを親でまとめて受け取る手法です。

あとから追加した要素も、追加のたびにリスナーを追加せずにそのまま動きます。そのため、項目が増えたり減ったりするリストやテーブルで特に役立ちます。

注意したい点として、親に付けたリスナーは、**親の範囲の中ならどこをクリックしても発火**するため、余白など処理したい要素ではない場所をクリックしたときも反応してしまいます。
そのため、`event.target` や `closest()` を使って「処理したい要素であるか」を確かめる必要があります。

## 参考

- [イベント入門（MDN）](https://developer.mozilla.org/ja/docs/Learn_web_development/Core/Scripting/Events)
- [Event.stopPropagation（MDN）](https://developer.mozilla.org/ja/docs/Web/API/Event/stopPropagation)
- [意外とハマるJavaScriptのイベントバブリング、ちゃんと理解しよう（kengineer.jp）](https://kengineer.jp/blog/javascript-bubbling/)
- [【JavaScriptの基本】イベント移譲（TCD）](https://tcd-theme.com/2022/09/javascript-event-delegation.html)
- [JavaScriptのイベントの設定方法とバブリングについて（note）](https://note.com/yoko_0413/n/n669e88fe5b83)
