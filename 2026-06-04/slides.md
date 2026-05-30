---
# try also 'default' to start simple
theme: light-icons
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides (markdown enabled)
title: Welcome to Slidev
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# apply UnoCSS classes to the current slide
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: fade-out
# enable Comark Syntax: https://comark.dev/syntax/markdown
comark: true
# duration of the presentation
duration: 35min
layout: center
colorSchema: light
twoslash: true
addons:
  - "@katzumi/slidev-addon-ogp-image"
---

# AIに頼った開発でもOSSに貢献したい

Presentation slides for developers

<div @click="$slidev.nav.next" class="mt-12 py-1" hover:bg="white op-10">
  Press Space for next page <carbon:arrow-right />
</div>

<div class="abs-br m-6 text-xl">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="slidev-icon-btn">
    <carbon:edit />
  </button>
  <a href="https://github.com/slidevjs/slidev" target="_blank" class="slidev-icon-btn">
    <carbon:logo-github />
  </a>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---

# OSS活動が趣味


---

# 最近の開発スタイル

---

# 不安

---

# ある日Reloadの無限ループが

---

# Claude Codeと一緒になんとか原因を特定

---

# これってほかの人も困らない？

修正方法はシンプル

---

# どこのパッケージを修正したらいいのか

- OpenAPI Validator
- l5-swagger
- vite
- laravel-vite-plugin

---

# OpenAPI Validatorのスキーマ生成の方法が悪いのでは？

---

# generatorの生成方法が違う

Laravel OpenAPIはメモリ経由、l5-swaggerはファイル経由

これはメモリ効率の問題で一長一短があるので、変えるのは判断が難しそう

---

# 悩んだ結果

OpenAPI Validatorのドキュメントを修正したPRを作成

<img
  src="https://opengraph.githubassets.com/a8f0a96c8d01c1bade3fd20ab0c0eed758084b692f709fc04c6a3034194ece4e/KentarouTakeda/laravel-openapi-validator/pull/69"
  :width="1110 / 2"
  :height="555 / 2"
  style="border: solid 2px black; border-radius: 8px;"
/>

## Readme.md

```md
+ ```md
+ // vite.config.js
+ export default defineConfig({
+     server: {
+         watch: {
+             ignored: ['**/storage/api-docs/**'],
+         },
+     },
+     // ...
+ });
+ ```
```

merged !!

---

# これってそもそもstorageがviteのwatch対象になってるのが問題なのでは？

laravel-vite-pluginのwatch対象からstorageを外すPRも作成

devinでissueとPRを調査して、claudeに修正方針を相談

---

# laravel-vite-pluginにも出そう

関連のissueやPRの調査はdevin、laravel-vite-pluginの実装はDeepWikiで

過去にLaravel本家側にvite.config.jsのテンプレートを追加するPRがあったり
tailwindでも同様の問題が起きているissueがあったりして、同様の修正は歓迎されそう

議論されている内容をAIで要約して、実装方針を決定

<img
  src="https://opengraph.githubassets.com/cb520aba2db11a1525a8b74f55d5a20d1ecdeb59af9bd80e53b9d3c5438c0bca/laravel/vite-plugin/pull/349"
  :width="1110 / 2"
  :height="555 / 2"
  style="border: solid 2px black; border-radius: 8px;"
/>

---

# そしてLaravel Live Japan

laravel/vite-plugin のContributorsの先頭に名前が載っているTim Macdonaldが来る！
issueとかPRでも彼が一番活発にやりとりしている人だった

![alt text](contributers.png)

![alt text](tim-macdonald.png.png)

---

# 勇気を出して話しかけに行きました


---
