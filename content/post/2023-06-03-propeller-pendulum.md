---
title: "プロペラを用いた倒立振子の製作"
date: 2023-06-03T04:20:32+09:00
draft: false
description: プロペラを用いた倒立振子を製作しました。
tags:
  - "制御工学"
  - "倒立振子"
categories:
  - "技術資料"
thumbnail:
  src: "posts/2023-06-03-propeller-pendulum/pendulum.jpg"
---

プロペラを用いた倒立振子を製作しました。

<!--more-->

[Design, Modeling and Control of an Omni-Directional Aerial Vehicle ](https://flyingmachinearena.org/wp-content/publications/2016/breIEEE16.pdf)を参考にしています。

以下は以前に作成した説明スライドです。
{{< figure src="/posts/2023-06-03-mft2023/intro.png">}}

{{< rawhtml >}}
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config">
    MathJax.Hub.Config({tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}});
</script>
{{< /rawhtml >}}

### 1. プロペラの方向最適化
機体にかかる力とトルクのL2ノルムが最大になるようにプロペラの方向が最適化されています。

### 2. 機体作成
カーボンロッドを繋いで立方体としています。
回路基板は4枚で，組み合わせて小さな立方体になるようにしました。

### 3. 姿勢推定
[Madgwickフィルタでクォータニオンを推定する](Madgwickフィルタでクォータニオンを推定する)に記述したようにクォータ二オンを推定しています。
EKFを用いて姿勢推定することができるようにも実装しています。

### 4. 姿勢制御
最初に記述した参考文献に載っていたものです。
角速度の求め方ですが，Lyapnov安定となるように制御設計されています。

### 5. 推力配分
姿勢制御で求めたトルク入力を満たすように8つのプロペラ推力に配分します。
[CVXGEN](https://cvxgen.com/docs/index.html)を用いて2次計画問題を解いています。

### 6. 実験結果
{{< youtube 2SDSoaEJgWo >}}
