---
# try also 'default' to start simple
theme: light-icons
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides (markdown enabled)
title: PHPでサンプリングパッドを作った
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
transition: slide-left
# enable Comark Syntax: https://comark.dev/syntax/markdown
comark: true
# duration of the presentation
duration: 35min
layout: center
colorSchema: light
twoslash: true
---

# PHPでサンプリングパッドを作った

Natsuki

---

# デモ

とりあえず見てもらった方が

<SlidevVideo src="/2026-06-02 01-22-56.mov" autoplay controls style="width: 80%;" />

---

# なぜ作ろうと思ったか

Inspired By Laravel Live Japan

<!-- ![alt text](./title-marcel.png) -->
<img src="./title-marcel.png" style="width: 80%" />

---
layout: center
---

<iframe
  :width="560 * 1.3"
  :height="315 * 1.3"
  src="https://www.youtube.com/embed/TR25AkhjiRc?si=jsSPP-c2PP_J9Tr1&amp;start=18988"
  title="YouTube video player"
  frameborder="0"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
  referrerpolicy="strict-origin-when-cross-origin"
  allowfullscreen
></iframe>

---

# 構成

- SuperCollider
- ReactPHP (valzargaming/php-osc)
- GUI (Vue)

---

# SuperCollider とは

<div style="display: flex; justify-content: space-between;">
  <ul>
    <li>音響合成用プログラミング言語 (IDE)</li>
    <li>EDMの制作などに使われる</li>
    <li>OSC(Open Sound Control)というプロトコルでも外部と通信できる</li>
  </ul>
  
  <img src="https://scsynth.org/uploads/default/original/1X/4727150d43766eb27cf9ab704c719659c3dbba80.png" style="width: 15em;" />
</div>

<img v-click src="./scd-sample.png" />

---

# SuperCollider でOSCを受け取る

<div style="display: flex; justify-content: space-between; align-items: flex-start;">
  <img src="./scd-load-cmd.png" style="width: 48%;" />
  <img src="./scd-play-cmd.png" style="width: 48%;" />
</div>

<p v-click>
  これで <code>/load pad1 [path]</code> でwavファイルを読み込ませて、<code>/play 1</code>というOSCメッセージを受け取ると、サンプルを再生するようになる
</p>

<p v-click>
  これをPHPからUDPで送れれば音が鳴らせる
</p>

<!--
```
/* `/load pad1 /User/samples/abc.wav` を受信 */
OSCdef(\loadSample, { |msg|
    var name = msg[1];
    var path = msg[2].asString;

    Buffer.read(
        s,
        path,
        action: { |buffer|
            ~samples[name] = buffer;
        }
    );
}, '/load');
```
-->

<!--
```scd [boot.scd]
OSCdef(\playSample, { |msg|
    var name = msg[1];
    var buffer;

    buffer = ~samples[name];

    // サンプルを再生する処理
    PlayBuf.ar(
        numChannels: 1,
        bufnum: buffer,
        rate: BufRateScale.kr(buffer),
        doneAction: 2
    );
}, '/play');
```
-->

---

# ReactPHP

OSCを送信するために valzargaming/php-osc を使用

以下を実行すればOSCが送られる

```php [server.php] {all|4-5|7-9}
$client = new \PhpOSC\OSCClient($loop);
$client->set_destination('127.0.0.1', 57120);

$promise = $client->sendAsync(
    new \PhpOSC\OSCMessage('/load', ['pad1', __DIR__ . '/samples/346.wav'])
)->then(function () use ($client) {
    $client->sendAsync(
        new \PhpOSC\OSCMessage('/play', ['pad1'])
    );
});
```

<p v-click>
  この処理をWebSocketサーバーにして、GUIから呼び出せるようにする
</p>

<style>
  .slidev-code-wrapper {
    --slidev-code-font-size: 18px;
  }
</style>

---

# 録音時のフロー

<img src="./load-cmd.svg" style="width: 100%;" />

---

# 再生時のフロー

<img src="./play-cmd.svg" style="width: 100%;" />

---

# まとめ

<div class="text">
すごい人たちの熱に触れて、勢いで作ったらできた
</div>

<p v-click class="zen-old-mincho-regular">
すごいぞ、カンファレンス！！！
</p>

<p v-click class="zen-old-mincho-regular">
すごいぞ、PHP！！！！！！！
</p>

<style>
@import url('https://fonts.googleapis.com/css2?family=Zen+Old+Mincho:wght@900&display=swap');

.text {
  font-size: 1.5em;
}

.zen-old-mincho-regular {
  font-family: "Zen Old Mincho", serif;
  font-weight: 900;
  font-style: normal;
  font-size: 3em;
  line-height: 1.2;
}
</style>