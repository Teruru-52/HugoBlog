---
title: "倒立振子"
date: 2022-03-25T01:41:21+09:00
draft: false
tags:
  - "倒立振子"
categories:
  - "Works"
---

<!--more-->

{{< rawhtml >}}
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
    <script type="text/x-mathjax-config">
    MathJax.Hub.Config({tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}});
</script>
{{< /rawhtml >}}

## 1. フライホイール倒立振子
#### 1号機
PID制御で辺倒立しました。
{{< figure src="/works/pendulum/flywheel_v1.jpg" width="30%">}}

#### 2号機
角を作って起き上がり辺倒立を可能にしました。
{{< figure src="/works/pendulum/flywheel_v2.jpg" width="30%">}}

#### 3号機
画像はないですが，3号機から点倒立に向けて開発を始めました。

#### 4号機
ブレーキをつけましたが，うまくブレーキがかからなかったため断念。
また，点倒立まではできませんでした。
{{< figure src="/works/pendulum/flywheel_v4.jpg" width="30%">}}

#### 5号機
[PD制御を用いた3次元倒立振子の製作](https://teruru-52.github.io/post/2022-07-16-3d-inverted-pendulum/)に記述しています。
初めて点倒立ができた機体です。
{{< figure src="/posts/2022-07-16-3d-inverted-pendulum/inverted_pendulum.jpg" width="30%">}}

#### 6号機
[状態空間表現を用いた3次元倒立振子の製作](https://teruru-52.github.io/post/2022-05-14-3d-inverted-pendulum2/)に記述しています。製作中。
{{< figure src="/posts/2022-05-14-3d-inverted-pendulum2/cubli.jpg" width="30%">}}

## 2. プロペラ倒立振子
[プロペラを用いた倒立振子の製作](https://teruru-52.github.io/post/2023-06-03-propeller-pendulum/)に記述しています。
姿勢推定の精度が出せず，点倒立はできていません。

{{< figure src="/posts/2023-06-03-propeller-pendulum/pendulum.jpg" width="30%">}}

## 3. 二輪倒立振子
LQR制御で倒立制御しました。
{{< figure src="/works/pendulum/pendulum_2wheels.jpg" width="30%">}}
