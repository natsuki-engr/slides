---
# You can also start simply with 'default'
theme: light-icons
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides (markdown enabled)
title: PHPStanのエラーをprettyにしようとしている
# apply unocss classes to the current slide
class: text-left
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: fade-out
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
# take snapshot for each slide in the overview
overviewSnapshots: true
layout: center
twoslash: true
colorSchema: dark
---

# PHPStanのエラーをprettyにしようとしている

<div class="text-right">
Natsuki
</div>

---

# PHPStan使ってますか

```php{all|6,10}
<?php

class User {
  public function __construct(
    public string $name,
    public ?int $age
  ) {}

	public function nextAge(): int {
		return $this->age + 1;
	}
}
```

<div v-click>
  <img src="/image.png" />
</div>


---

# VSCodeのエラーは見づら過ぎる!!

```php
/**
 * @return array{
 *     id: int,
 *     name: string,
 *     // ...
 * }
 */
function getProfile(): array
{
  return [
    'id' => 123,
    'name' => 'John Doe',
    // ...
  ];
}
```

<div v-click>
  <img src="/long-phpstan-error.png" />
</div>

---

# pretty-ts-errorsみたいなのが欲しい

<div v-click>
  <img src="/ext-pretty-ts-errors.png" />
</div>

<div v-click>
  <img src="/pretty-ts-errors-comparison.png" />
</div>

<div v-click>
  こんな風になったらなと思ってた
</div>

---

# エラー文をパスーすればいいのでは(1)

<p>php-yacc</p>

<p>第179回 PHP勉強会＠東京でのきんじょうさんの話</p>

<iframe width="560" height="315" src="https://www.youtube.com/embed/dObaHQD_jic?si=BaRIWfXzZ91C3UHt&amp;start=6741" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

https://www.youtube.com/live/dObaHQD_jic?si=qmN3z1ZGNNnTKwPZ&t=6736

<p v-click>PHP-astとか作ったあのNikita作</p>

---
layout: two-cols-header
---

# エラー文をパスーすればいいのでは(2)

::left::

<p>racc</p>

<p>Umeda.rbにて わいださん のraccの話</p>
<img style="width: 100%; height: auto;" src="/umeda-rb.png" />


::right::

<div v-click>
  <p>ぺちこん関西2025でのトーク</p>
  <iframe :width="560 * 0.8" :height="315 * 0.8" src="https://speakerdeck.com/player/45bf47f208fb4a58b42854b5ce009d59?slide=18"></iframe>
</div>

::bottom::

<div v-click>
  LRパーサーとか難しいのでなんとなくの理解
</div>

---

# パーサーを使わないアプローチ

パーサーは作らずエラーパターンからマッチさせる

<div>
  <img src="/long-phpstan-error.png" />
</div>

<p v-click>
  <pre>/Function <code>関数名</code> should return <code>型(array{...})</code> but returns <code>型(array{...})</code>./</pre>
</p>

<p v-click>
エラーの種類だけ、パターンマッチで拾う
</p>

<ul>
  <li v-click>エラーパターンの変更を追従するのが大変 & プラグインの数だけ対応が必要</li>
  <li v-click>=> 汎用的にエラー文をパースできた方がいい</li>
</ul>

<style>
  p {
    margin: 1rem 0;
  }
  
  pre {
    font-size: 1.1rem !important;
    color: #dba226ff;
  }
  pre code {
    color: #50fa7b;
  }
</style>

---

# ちなみにpretty-ts-errorsは正規表現

```js{all|3|5-9}
message
.replaceAll(
  /(is missing the following properties from type\s?)'(.*)': ((?:#?\w+, )*(?:(?!and)\w+)?)/g,
  (_, pre, type, post) =>
    `${pre}${formatTypeBlock("", type, codeBlock)}: <ul>${post
      .split(", ")
      .filter(Boolean)
      .map((prop: string) => `<li>${prop}</li>`)
      .join("")}</ul>`
  )
```

https://github.com/yoavbls/pretty-ts-errors/blob/8527285178e9a88c80ad63fa7f0129192b67bad2/packages/formatter/src/formatDiagnosticMessage.ts#L4

<div v-click>
  TSはエラーパターンが決まってるからできる
</div>

---

# Chevrotain

- TypeScript製のパーサージェネレーター

- マメジカ
<img src="https://preview.aflo.com/jFBj9j66IdjO/aflo_31160821.jpg" />

- グラフが見れる


---

# 例


---

# パース難しい

```txt
Method bar ...
The method might has ...
```


---

# VSCode拡張の組み込み


---

# アプローチは二つ

- pretty-ts-errorsのように別拡張として提供
  - 新たにインストールしてもらう必要がある
- phpstan-vscode自体に組み込む


---

# pretty-ts-errorsとphpstan-vscodeがエラーを表示している仕組み

- pretty-ts-errors
  - Diagnostic
- phpstan-vscode
  - HoverProvider


---

# Diagnostic(診断)

場所とメッセージを渡すとエラー表示してくれる

<!-- API仕様 -->
```json
```

---

# HoverProvider

ホバーイベントに応じてメッセージを返す

<!-- API仕様 -->
```json
```

---

# MarkdownはHoverProviderにしか使える

---

# pretty-ts-errorsでもそれによるissueがある

Hide Unformatted TypeScript errors / move to after the pretty errors
https://github.com/yoavbls/pretty-ts-errors/issues/3

---

# LSP 3.18 ならDiagnosticにもMarkdownが使える

# いつになるか分からないのでHoverProviderで実装する

---

# ちなみにpretty-ts-errorsの対応方針

TS server plugin
