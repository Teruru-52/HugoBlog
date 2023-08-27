---
title: "PD制御を用いた3次元倒立振子の製作"
date: 2022-07-16T20:46:42+09:00
draft: false
description: PD制御を用いた3次元フライホイール倒立振子を製作しました。
tags:
  - "制御工学"
  - "倒立振子"
categories:
  - "技術資料"
thumbnail:
  src: "posts/2022-07-16-3d-inverted-pendulum/inverted_pendulum.jpg"
---

PD制御を用いた3次元フライホイール倒立振子を製作しました。

<!--more-->

{{< rawhtml >}}
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config">
    MathJax.Hub.Config({tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}});
</script>
{{< /rawhtml >}}

### 1. 回路
[HomeMadeGarbage](https://shop.homemadegarbage.com/product-category/hmg/)さんのものを使わせていただいています。

### 2. 機体
[ReM-RC](https://www.youtube.com/watch?v=AJQZFHJzwt4&list=LL&index=10&t=263s)さんのものを使わせていただいています。

### 3. 制御
HomeMadeGarbageさんとReM-RCさんの倒立振子とはセンサーの位置を変えて，倒立時にセンサの$xy$平面が地面と水平になるようにしています。

倒立状態からの$x$軸周りの傾き角とその角速度を$\theta_x,\omega_x$とし，$y$軸についても同様に$\theta_y,\omega_y$とします。

$$u_x(k)=k_1\hat\theta_x(k)+k_2\hat\omega_x(k)+k_3\sum_{i=0}^{k-1}u_x(i)$$

$$u_y(k)=k_1\hat\theta_y(k)+k_2\hat\omega_y(k)+k_3\sum_{i=0}^{k-1}u_y(i)$$

$$u_z(k) = 0$$

$k_1,k_2,k_3$はゲインです。姿勢角$\hat\theta_x,\hat\theta_y$とその角速度$\hat\omega_x,\hat\omega_y$は定常カルマンフィルタによって推定しています。とりあえず倒立できればいいので，$u_z=0$としています。

この入力$u_x,u_y,u_z$を3つのホイールに分配します。センサ座標$xyz$での入力$u_x,u_y,u_z$をホイール座標$x'y'z'$での各ホイールの入力$u_{x'},u_{y'},u_{z'}$に分配します。

{{< rawhtml >}}
$$
u_xx+u_yy+u_zz=u_{x`}x'+u_{y'}y'+u_{z'}z'
$$
{{< /rawhtml >}}

$$
x = \dfrac{-y'-z'+2x'}{\sqrt6}
$$

$$
y = \dfrac{y'-z'}{\sqrt2}
$$

$$
z = -\dfrac{x'+y'+z'}{\sqrt3}
$$

より，

$$
u_{x'} = \dfrac{2}{\sqrt6}u_x-\dfrac{1}{\sqrt3}u_z
$$

$$
u_{y'} = -\dfrac{1}{\sqrt6}u_x+\dfrac{1}{\sqrt2}u_y-\dfrac{1}{\sqrt3}u_z
$$

$$
u_{z'} = -\dfrac{1}{\sqrt6}u_x-\dfrac{1}{\sqrt2}u_y-\dfrac{1}{\sqrt3}u_z
$$

としてホイールに入力します。
制御周期$10\ \text{ms}$程度で制御しています。

### 4. 結果
{{< youtube ZBS4nwo4Kak >}}

$z$軸周りの入力$u_z$を考慮して制御に加えると，もう少しふらつきが収まるかと考えています。

今後また更新します。

