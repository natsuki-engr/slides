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

普段の業務ではVueを使用していて、去年の6月ごろにReactも業務外で勉強し始めました


※「優れている」とは表現しない

---

# 自己紹介

<div class="m-y-8">
Natsuki
</div>

普段はLaravel + Vueで開発

<div>
  <img src="https://avatars.githubusercontent.com/u/63272932" />
</div>

::right::

<div v-click>
  <img src="/1731512017408.jpg" />
  <p class="text-right">Vue Fes Japan 2024</p>
</div>

<style>
  h1 {
    line-height: 1;
  }
</style>

---

# 宣伝

Reactを学ぶ上で、VSCodeの拡張機能を作ってみました。

Your Themes
![capture of marketplace](/marketplace.png)
https://marketplace.visualstudio.com/items?itemName=Natsuki.your-themes

---

<img src="/demo.gif" />

<!-- 「コードの色も反映する予定です」 -->
<!-- ページを開いた時にgifをスタートしたい -->
---

# 本題

<div class="text-center">
  <h3>「Reactのいいなと思ったところ」</h3>
</div>

---

# 一貫してJS

<ul>
  <li>JSX</li>
  <li>コンポーネント引数</li>
</ul>

---

# 再レンダリングのサイクルが理解しやすい

再レンダリングのときに基本的に全て再計算される
再計算や再処理をスキップしたい場合に、useEffectやuseMemoを使う

---

# dangerouslySetInnerHtml

「危険なんだぞ」という強い警告を感じる

