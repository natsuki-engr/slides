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
  <img src="/phpstan-playground.png" />
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

---

# 例

<div style="display: flex; justify-content: center;">
  <img width="50%" src="/json-parser-diagram.png" />
</div>

https://chevrotain.io/playground/

---
layout: two-cols-header
---

# 例

::left::

```js{all|3-8|10-19}
class JsonParser extends CstParser {
  constructor() {
    this.RULE("json", () => {
      this.OR([
        { ALT: () => this.SUBRULE(this.object) },
        { ALT: () => this.SUBRULE(this.array) },
      ]);
    });

    this.RULE("object", () => {
      this.CONSUME(LCurly);
      this.MANY_SEP({
        SEP: Comma,
        DEF: () => {
          this.SUBRULE(this.objectItem);
        },
      });
      this.CONSUME(RCurly);
    });

    this.RULE("objectItem", () => {
      this.CONSUME(StringLiteral);
      this.CONSUME(Colon);
      this.SUBRULE(this.value);
    });
    
    // ...
  }
}
```

::right::

<img width="100%" src="/json-parser-diagram.png" />

---

# パーサーの定義の前に字句解析(Lexer)

```js
const True = createToken({ name: "True", pattern: /true/ });
const False = createToken({ name: "False", pattern: /false/ });
const Null = createToken({ name: "Null", pattern: /null/ });
const LCurly = createToken({ name: "LCurly", pattern: /{/ });
const RCurly = createToken({ name: "RCurly", pattern: /}/ });
```

<style>
  pre {
    font-size: 1.2rem !important;
  }
</style>

---

# phpstan-error-parserを作っています

<img width="60%" src="/phpstan-error-parser-diagram.png">

https://github.com/natsuki-engr/phpstan-error-parser

---

# 使い方

```ts
import { parse } from 'phpstan-error-parser';

const result = parse('PHPDoc tag @mixin contains unresolvable type.')
```

```json
[
  {
    type: 'common_word',
    value: 'PHPDoc',
    location: {
      startColumn: 0,
      endColumn: 6,
    },
  },
  {
    type: 'common_word',
    value: 'tag',
    location: {
      startColumn: 7,
      endColumn: 10,
    },
  },
  {
    type: 'doc_tag',
    value: '@mixin',
    location: {
      startColumn: 11,
      endColumn: 17,
    },
  },
  {
    type: 'common_word',
    value: 'contains',
    location: {
      startColumn: 18,
      endColumn: 26,
    },
  },
  {
    type: 'common_word',
    value: 'unresolvable',
    location: {
      startColumn: 27,
      endColumn: 39,
    },
  },
  {
    type: 'common_word',
    value: 'type',
    location: {
      startColumn: 40,
      endColumn: 44,
    },
  },
  {
    type: 'period',
    value: '.',
    location: {
      startColumn: 44,
      endColumn: 45,
    },
  },
];
```

---

# VSCode拡張の組み込み

<p>アプローチは二つ</p>

<ul>
  <li v-click>pretty-ts-errorsのように別拡張として提供</li>
  <li v-click>phpstan-vscode自体に組み込む</li>
</ul>

---

# pretty-ts-errorsとphpstan-vscodeがエラーを表示している仕組み

- pretty-ts-errors ⇒ `Diagnostic`
- phpstan-vscode ⇒ `HoverProvider`

---

# Diagnostic(診断)

場所とメッセージを渡すとエラー表示してくれる

```ts
interface Diagnostic {
	range: Range;

	severity?: DiagnosticSeverity;

	source?: string;

	message: string;
}
```

https://microsoft.github.io/language-server-protocol/specifications/lsp/3.17/specification/#diagnostic

<div v-click>
phpstan-vscodeはPHPStanの結果をこれでVSCodeに渡している
</div>

<span v-click>
  messageにMarkdownを渡せばリッチな表示ができる？
</span>
<span v-click>⇒ できない🙅</span>

<style>
  a {
    color: #7f7f7fff;
    font-size: 0.6em;
  }
</style>
---

# HoverProvider

ホバーイベントに応じてメッセージを返す

```json{all|5}
interface Hover {
	/**
	 * The hover's content
	 */
	contents: MarkedString | MarkedString[] | MarkupContent;

	/**
	 * An optional range is a range inside a text document
	 * that is used to visualize a hover, e.g. by changing the background color.
	 */
	range?: Range;
}
```

https://microsoft.github.io/language-server-protocol/specifications/lsp/3.17/specification/#textDocument_hover

<div v-click>
  `Hover`ではcontentsにMarkdownを渡せばリッチな表示ができる❗️❗️
</div>

<style>
  a {
    color: #7f7f7fff;
    font-size: 0.6em;
  }
</style>
---

# pretty-ts-errorsでもそれによるDiscussionがある

https://github.com/yoavbls/pretty-ts-errors/discussions/43

<img src="/discussion.png" />

---

# LSP 3.18 ならDiagnosticにもMarkdownが使える

<img src="/lsp-3-18.png" />

https://microsoft.github.io/language-server-protocol/specifications/lsp/3.18/specification/#diagnostic

いつになるか分からないので

別拡張としてphpstan-error-parserでパースして、HoverProviderで実装する予定

<div v-click style="display: flex; justify-content: center; margin-top: 2rem; font-size: 2rem;">
おしまい
</div>

<style>
  a {
    color: #7f7f7fff;
    font-size: 0.6em;
  }
</style>