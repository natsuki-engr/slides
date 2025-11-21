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


---

# VSCodeのエラーは見づら過ぎる!!

<!-- 米倉涼子 -->


---

# pretty-ts-errorsみたいなのが欲しい


---

# エラーをパスーすればいいのでは

PHP勉強会@東京でのきんじょうさんの話
Umeda.rbにてわいださんがyaccを紹介
=> パーサージェネレーターがあればパーサーを作れる？


---

# もうひとつのアプローチ

パーサーは作らずエラーパターンからマッチさせる
エラーパターンの変更を追従するのが大変 & プラグインの数だけ対応が必要

=> 汎用的にエラー文をパースできた方がいい


---

# Chevrotain

- マメジカ
- js
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
