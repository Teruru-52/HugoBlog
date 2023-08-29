---
title: "マイクロマウスの横滑り角の考え方"
date: 2023-01-07T02:09:34+09:00
draft: false
description: マイクロマウスの横滑り角の考え方の種類を示します。
tags:
  - "マイクロマウス"
categories:
  - "技術資料"
---

マイクロマウスの横滑り角の考え方の種類を示します。

<!--more-->
{{< rawhtml >}}
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config">
    MathJax.Hub.Config({tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}});
</script>
{{< /rawhtml >}}

参考にした記事は，
- [マイクロマウスの横滑り運動](https://rikei-tawamure.com/entry/2020/05/24/235156)
- [Fノート](http://fnote013.blog.fc2.com/)
- [マイクロマウスロボットにおける計測・制御技術の最先端](https://www.jstage.jst.go.jp/article/sicejl/52/6/52_528/_pdf)
- [今年のマイクロマウス製作の話](https://www.slideshare.net/dangorogoro/ss-138942947)

です．

前提として，マイクロマウスは既に一定の速度で走行しており，駆動力は転がり抵抗と釣り合っているとして$F_x=0$とおきます。
ここでは簡単のために2輪マウスについて考えます．

<!-- 機体の並進方向に対する横滑り角$\beta$は以下の式で与えられます。

$$
\beta = \text{tan}^{-1}\dfrac{V\text{sin}\beta}{V\text{cos}\beta} = \text{tan}^{-1}\dfrac{v_y}{v_x}
$$ -->

### 1. 厳密なモデル
左右の車輪の横滑り角をそれぞれ$\beta_l$，$\beta_r$とします。これらは機体の横滑り角$\beta$とトレッド$d$を用いて，

$$ 
\beta_l = \text{tan}^{-1}\dfrac{V\text{sin}\beta}{V\text{cos}\beta-d\omega},\  \beta_r = \text{tan}^{-1}\dfrac{V\text{sin}\beta}{V\text{cos}\beta+d\omega}
$$

と表せます。
コーナリングフォース$Y$は横力$C$を用いて

$$
Y_l = C_l\text{cos}\beta_l,\ Y_r = C_r\text{cos}\beta_r
$$

となります。一定の速度で走行し，駆動力が転がり抵抗と釣り合っているような状況ではMagic Formulaを用いて

$$
C_l = K\ \text{sin}[F\text{tan}^{-1}[B\beta_l-E(B\beta_l-\text{tan}^{-1}(B\beta_l))]]
$$

$$
C_r = K\ \text{sin}[F\text{tan}^{-1}[B\beta_r-E(B\beta_r-\text{tan}^{-1}(B\beta_r))]]
$$

と表せます．$K$，$B$，$E$，$F$は係数です．

横滑り角の運動方程式は

$$
mV(\dot\beta + \omega) = -F_x\text{sin}\beta + F_y\text{cos}\beta   
$$

であり，機体にかかる力は$F_x=0$，$F_y=Y_l\text{cos}\beta_l + Y_r\text{cos}\beta_r$であることを考慮すると

$$
mV(\dot\beta + \omega) = (Y_l\text{cos}\beta_l + Y_r\text{cos}\beta_r)\text{cos}\beta   
$$

より

$$
\dot\beta = \dfrac{\text{cos}\beta}{mV}(Y_l\text{cos}\beta_l + Y_r\text{cos}\beta_r)-\omega
$$

となります。

### 2. 各車輪の横滑り角を微小として近似したモデル
厳密なモデルでは，未知パラメータが多く，また計算も複雑になります。

そこで，$\beta$，$\beta_l$，$\beta_r$が微小であるとして近似します。

$$
\beta_l\approx\dfrac{V\beta}{V-d\omega},\ \beta_r\approx\dfrac{V\beta}{V+d\omega}
$$

$$
Y_l\approx C_l,\ Y_r\approx C_r
$$

$$
C_l\approx K\beta_l,\ C_r\approx K\beta_r
$$

運動方程式を考えると，

$$
mV(\dot\beta + \omega) = F_y
$$

より

$$
\dot\beta = \dfrac{2KV}{m(V^2-d^2\omega^2)}\beta-\omega
$$

となります。

### 3. さらに簡略化したモデル
トレッドを$d=0$とすると，

$$
\dot\beta = \dfrac{2K}{mV}\beta-\omega
$$
となり，簡単な微分方程式となります。$\frac{2K}{m}$はまとめて係数として実験しながら決めるケースがよく見られます。

### 4. 前後の時刻の横滑り角の影響を無視する方法
前後の時刻の横滑り角の影響を無視して，とりあえず現時刻の横滑り角を$V$と$\omega$から求める方法です。

$$
0 = \dfrac{2K}{mV}\beta-\omega
$$

より

$$
\beta = \dfrac{m}{2K}V\omega
$$

となります。