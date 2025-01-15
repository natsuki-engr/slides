---
# You can also start simply with 'default'
theme: light-icons
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides (markdown enabled)
title: Reactのいいなと思ったところ
# apply unocss classes to the current slide
class: text-left
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
overviewSnapshots: true
layout: center
twoslash: true
---

# Reactのいいなと思ったところ

<div class="text-right">
Natsuki
</div>

---

# 今日話すこと

<div class="text-2xl">
普段の業務ではVueを使用していて、最近Reactも業務外で勉強し始めました

初心者の目線で「Reactのココいいな」と思った点を共有したいと思います
</div>

<div v-click class="mt-8">
  <h3 class="opacity-100">自己紹介</h3>

  <div class="m-y-4">
  Natsuki
  </div>

  普段はLaravel + Vueで開発

  <div class="absolute right-8 rounded-full w-[150px] h-[150px] overflow-hidden">
    <img src="https://avatars.githubusercontent.com/u/63272932" />
  </div>
</div>

<style>
  h1 {
    line-height: 1;
  }
  h3 {
    font-size: 1.75rem;
  }
</style>

---

# 宣伝

<div>
Reactを学ぶ上で、VSCodeの拡張機能を作ってみました。
</div>

Your Themes
<div class="h-[20rem] overflow-hidden">
<img src="/marketplace.png">
</div>
https://marketplace.visualstudio.com/items?itemName=Natsuki.your-themes

---

demo

<img src="/demo.gif" />

<!-- 「コードの色も反映する予定です」 -->
<!-- ページを開いた時にgifをスタートしたい -->
---

# 本題

<div class="text-center">
  <h2 class="absolute inset-0 m-auto opacity-100 text-6xl h-4">「Reactのいいなと思ったところ」</h2>
</div>

---

# dangerouslySetInnerHtml

<div v-click>

```tsx
const markup = '<p>some raw html</p>';
return <div dangerouslySetInnerHTML={{__html: markup}} />;
```

</div>

<div v-click>
  <div class="text-xl m-t-4">
    タグを含んだ文字列をHTMLタグとしてレンダリングしたい場合の`setInnerHtml`と同じ
  </div>
  <div class="text-xl m-t-4">
    XSS攻撃の危険性があるので、注意が必要
  </div>
</div>

<div v-click class="text-xl m-t-4">
「危険なんだぞ」という強い警告を感じるところがいい
</div>


<div v-click>
<div class="text-xl m-t-6">
vueだと<code>v-html</code>で同じ実装ができるが、罪悪感が無い
</div>

```vue{5}
<script setup>
const markup = '<p>some raw html</p>'
</script>

<div v-html="markup"></div>
```

</div>

<style>
  pre.slidev-code {
    font-size: 1.25rem !important;
    margin-inline: .5rem;
  }
</style>

---

# 一貫してJS

<p class="lines">
「JSを知っていれば大体書ける」
</p>

<p v-click class="lines">
たとえばJSX
</p>

<style>
  .slidev-layout p.lines {
    opacity: 1;
    font-size: 2rem;
    line-height: 1;
    margin-top: 2rem;
    margin-bottom: 2rem;
  }
</style>
---

# JSX

<ul>
  <li>JSXの構成はJSとタグなので、テンプレートのために必要な知識が少ない</li>
  <li>JSのパラダイム/設計を持ち込める</li>
</ul>

```jsx{all|8-10|2-4}
function ItemList({ items: Item[] }) {
  if(items.length === 0) {
    return <p>empty...</p>
  }
  
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

<style>
  ul li {
    font-size: 1.2rem;
  }
  pre.slidev-code {
    font-size: 1rem !important;
    margin-inline: .5rem;
  }
</style>
<!-- 
たとえばItemの配列を受け取って表示するコンポーネントがあるとして、
ループさせたければ、
 -->
---

# レンダリングの更新が分かりやすい

<div class="text-xl my-8" v-click>
基本的に、引数やステートが変わって再レンダリングされるときに<br>
「全て再計算される」と思っておいて、
</div>

<div v-click>
  <div class="text-2xl my-4">
  ・特定の再計算をスキップしたければ<code class="text-orange">useMemo</code>
  </div>
  <div class="text-xl mb-4 ml-4">
    配列から値の検索とか
  </div>
</div>

<div v-click>
  <div class="text-2xl my-4">
  ・特定の処理をスキップしたければ<code class="text-orange">useEffect</code>
  </div>
  <div class="text-xl mb-4 ml-4">
    APIの呼び出しとか
  </div>
</div>

---
layout: two-cols-header
---

# useEffect/useMemo の悪い例

::left::

## 不要な`useEffect`

```tsx{all|6-8|all}
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');

  const [fullName, setFullName] = useState('');
  useEffect(() => {
    setFullName(firstName + ' ' + lastName);
  }, [firstName, lastName]);

  return /** ... */
}
```

::right::

## 不要な`useMemo`

```tsx{all|5-8|all}
function Form() {
  const [firstName, setFirstName] = useState("Taylor");
  const [lastName, setLastName] = useState("Swift");

  const fullName = useMemo(
    () => firstName + " " + lastName,
    [firstName, lastName]
  );

  return; /** ... */
}
```

::bottom::

```tsx
const fullName = firstName + " " + lastName
```

https://react.dev/learn/you-might-not-need-an-effect#updating-state-based-on-props-or-state

<style>
  .two-cols-header {
    grid-template-rows: 2.5rem 1fr;
    grid-gap: 1em;
  }
  .col-left,.col-right {
    margin-top: 1rem;
  }
  .col-bottom .slidev-code {
    font-size: 1.5rem !important;
  }
</style>

---

# なぜ今日この話をしようと思ったか

<img src="/post.png" style="height: 400px">

https://x.com/jordwalke/status/1875336115009573268

---

# 参考

<div>
【React】誤解される useMemo と 誤用される useState ―― 「A の変更に反応して B の値が変わる」と考えるべきでない<br>
https://qiita.com/honey32/items/58e56e407d4d87e294a4

【React】useEffect の標準動作は「依存配列の中身が変わると実行」ではない<br>
https://qiita.com/honey32/items/62edf5165aced7d0c4bf

useState + useEffect -> 再レンダリング中に毎回計算 (重ければ useMemo)<br>
https://scrapbox.io/honey32/useState_+_useEffect_-%3E_%E5%86%8D%E3%83%AC%E3%83%B3%E3%83%80%E3%83%AA%E3%83%B3%E3%82%B0%E4%B8%AD%E3%81%AB%E6%AF%8E%E5%9B%9E%E8%A8%88%E7%AE%97_(%E9%87%8D%E3%81%91%E3%82%8C%E3%81%B0_useMemo)

props または state に基づいて state を更新する - そのエフェクトは不要かも<br>
https://ja.react.dev/learn/you-might-not-need-an-effect#updating-state-based-on-props-or-state
</div>