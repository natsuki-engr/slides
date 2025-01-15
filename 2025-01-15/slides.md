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
<div class="h-[90vh] overflow-hidden">
<img src="/marketplace.png">
</div>
https://marketplace.visualstudio.com/items?itemName=Natsuki.your-themes

---

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

<div class="text-xl m-t-4">
  タグを含んだ文字列をHTMLタグとしてレンダリングしたい場合の`setInnerHtml`と同じ
</div>
<div class="text-xl m-t-4">
  XSS攻撃の危険性があるので、注意が必要
</div>

```tsx
const markup = '<p>some raw html</p>';
return <div dangerouslySetInnerHTML={{__html: markup}} />;
```

<div v-click class="text-xl m-t-4">
「危険なんだぞ」という強い警告を感じるところがいい
</div>

<div v-click class="text-xl m-t-6">
vueだと<code>v-html</code>で同じ実装ができるが、罪悪感が無い
</div>

<div v-click="3">

```vue
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

<div>
JSを知っていれば大体書ける

たとえばJSX
</div>

---

# JSX

<ul>
  <li>JSXの構成はJSとタグなので、テンプレートのために必要な知識が少ない</li>
  <li>JSのパラダイム/設計を持ち込める</li>
</ul>

```jsx
function ItemList({ items }) {
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

---

# レンダリング結果の更新が分かりやすい

基本的に「再レンダリングのときに全て再計算される」と思っておいて、
再計算や再処理をスキップしたい場合に、useEffectやuseMemoを使う

---

# ただの特徴とも取れる

「優れている点」ではなく、
