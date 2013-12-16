---
layout: page
title: レイトレーサーもどき
date: 2011-12-17 04:07
image: https://gist.github.com/snipsnipsnip/1369770/raw/2983e02b3293e5956cd9551923b1aa76acdcebe3/_ss.gif
excerpt: SDLでレイトレーサーを作った。ぐぐりながら適当に作って詰まった。
---

[![dumb raytraceroid](https://gist.github.com/snipsnipsnip/1369770/raw/2983e02b3293e5956cd9551923b1aa76acdcebe3/_ss.gif)](https://gist.github.com/raw/1369770/1c7c58ef76b57fc72059fc10e89db2020bcc1b39/_ray.zip)

## ダウンロード

[ray.zip](https://gist.github.com/raw/1369770/1c7c58ef76b57fc72059fc10e89db2020bcc1b39/_ray.zip)

[gist:1369770](https://gist.github.com/1369770)

去年(2010年)できそこないのレイトレーサを作った。ぐぐりながら適当に作って詰まった。オブジェクトの配置がハードコードで、反射を一度しかしていないので3Dエンジンとしては使えない。シェーダの勉強をしていたと思ったらいつのまにか出来上がっていたものだが、これ以上ほうっておくと忘れそうなのでまとめておく。

## 操作

操作 | 効果
-- | --
左ドラッグ | ライトを操作
マウスホイール | カメラ前進・後退
右ドラッグ | カメラ見回し
右クリック | モード切り替え (鏡面とか)
左クリックしながらマウスホイール | ズーム (視野角の調整)
右クリックしながらマウスホイール | スレッドの増減 (増やすと早くなるかも)
マウスホイールをクリック | リセット
Hキー | ヘルプを隠す
