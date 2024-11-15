---
# You can also start simply with 'default'
theme: light-icons
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides (markdown enabled)
title: jsファイルでも型チェックを行う
# apply unocss classes to the current slide
class: text-left
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
# take snapshot for each slide in the overview
overviewSnapshots: true
layout: center
twoslash: true
---

# jsが型安全になったっていい

<div class="text-right">
Natsuki
</div>

---
layout: two-cols
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

# JSDocで型注釈

## JSDocとは

<p>
  JavaDocやPHPDocのようなコメント内の注釈から<br>ドキュメントを生成するツール
</p>
<p>
  または、そのコメント自体
</p>

https://jsdoc.app/

<div v-click>
```js {all} twoslash
/** @type {number} */
const i = 0;
```
</div>

<p v-click>
  <code>@type</code> などのタグで型情報を書くことができ、tscもこの型情報を参照して型チェックを行ってくれる
</p>

<p v-click>
  これとtsconfig.jsonとtscで型チェックを行うことができる
</p>

<style>
  pre {
    --slidev-code-font-size: 1.6rem;
  }
</style>

<!-- 
まず、JSDocとはJavaDocやPHPDocのようなコメントからドキュメントを生成するツール、またはそのコメント自体を呼ぶこともあります。

例として、このように変数の前に書いたコメントによって、変数iがnumber型として解釈されます。

tsconfigでcheckJsをtrue
このコメント

実際現在私が関わっているプロダクトではこの方法で型チェックを行っています
 -->

---
transition: fade-out
---

# なぜtsファイルじゃないのか

<div class="m-t-10">
<p v-click>
  リプレース案件にアサインされた
</p>

<p v-click>
  「TypeScript使って堅牢なシステムにしたい！！」
</p>

<p>
  <span v-click>上の偉い人</span><span v-click>「TypeScript？そんなことより開発を急いでくれ」</span>
</p>

<div class="text-center text-8xl m-y-12" v-click>
  😇
</div>
</div>

<style>
  .slidev-layout leading-10 {
    line-height: 2.5rem;
  }
</style>

---

# なぜtsファイルじゃないのか

<div class="m-t-10">
  <div class="m-y-12 text-12">
    <p>
      どうにかして、<br>
      こっそりTypeScriptを使えないか
    </p>
  </div>

  <p v-click>
    JSDocなら、コメントに型が書けるらしい<br>
    コメントならバレない…
  </p>

  <p class="text-right" v-click>
    ※誇張が含まれています
  </p>
  
  <div v-click class="m-t-8">
    <p>
      ほかにも
    </p>
    <p>
      既存プロダクトをtsに移行したいけど、テストコードも無いしエンバグが怖い
    </p>
    <p>
      トランスパイルの時間を無くしたい
    </p>
    
      とかとか
  </div>
</div>

---
layout: two-cols-header
---

# JSDoc(+tsc)でできること

## 1. 型注釈の記述

::left::

<p>

```ts
// index.ts
function hello(name: string) {
  // ...
}

const LIMIT = {
  min: 0,
  max: 1000,
} as const
```

</p>

::right::
<div v-click>

```js
// index.js
/**
 * @param {string} name
 */
function hello(name) {
  // ...
}

const LIMIT = /** @type {const} */ {
  min: 0,
  max: 100,
}
```
</div>

<style>
  pre {
    --slidev-code-font-size: 1.2rem;
  }
</style>

---
layout: two-cols-header
---

# JSDoc(+tsc)でできること

## 2. 型の定義

::left::

<div>

```ts
// index.ts
interface User<T> {
  name: string
  age: number
  customData: T
}
```

</div>

::right::

<div v-click>

```js twoslash
// index.js
/**
 * @extends T
 * @typedef {{
 *  name: string
 *  age: number
 *  customData: T
 * }} User
 */
```

</div>

<style>
  pre {
    --slidev-code-font-size: 1.6rem;
  }
</style>

---

# JSDoc(+tsc)でできること

## 3. 型のインポート・エクスポート

```js
// index.js
/**
 * @import { User } './index.ts'
 */
```

<style>
  pre {
    --slidev-code-font-size: 1.3rem;
  }
</style>

---
transition: fade-out
---

# JSDocでできないこと

## 1. 条件型(Conditional Types)

<div v-click>

```ts
type IsNumber<T> = T extends number ? true : false;

type T1 = IsNumber<10>;
```

</div>

<div v-click class="m-y-8 text-xl">
  型定義だけ、`.d.ts`に書けば解決
</div>

<style>
  pre {
    --slidev-code-font-size: 1.3rem;
  }
</style>

---
transition: fade-out
---

# JSDocでできないこと

## 2. TypeScript独自の実装

Enumとかdeclareとか

<div v-click>
  <img src="/image.png" />
</div>

<div v-click>
  一応`@enum`はあるが、ただの連想配列なので、TSのように値からアクセスできない
</div>

---
transition: fade-out
---

# JSDocでできないこと

## 3. tsファイルでの型補完

JSDocの型はtsファイル内では無視される

<div v-click>

```ts twoslash
/**
 * @param {string} name
 */
function hello(name) {
  // ...
}
```

</div>

<style>
  pre {
    --slidev-code-font-size: 1.3rem;
  }
</style>

---

# JSDocを使う上での注意

<ul>
  <li v-click>
    「tsファイルよりJSDocの方がオススメ！」というわけではない<br>
    単純に書く文字数が増えるので、通常のWebアプリケーションの開発にはオススメしない
  </li>
  <li v-click>
    tsファイルへの移行は楽じゃない<br>
    JSDocはtsファイルに書いても型注釈として解釈されないので、書き直す必要がある
  </li>
  <li v-click>
    その他使いたいエコシステムが対応してないことも
  </li>
</ul>

<style>
  ul li {
    font-size: 1.6rem;
    margin-top: 0.5rem;
  }
</style>

---

おわり
