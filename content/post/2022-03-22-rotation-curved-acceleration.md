---
title: "曲線加速を用いた超信地旋回"
date: 2022-03-22T04:36:23+09:00
draft: false
tags:
  - "クラシックマウス"
  - "制御工学"
categories:
  - "技術資料"
---

<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
    <script type="text/x-mathjax-config">
    MathJax.Hub.Config({tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}});
</script>

[keriさんの記事](https://www.kerislab.jp/posts/2018-04-29-accel-designer1/)を参考に曲線加速を設計しました.

<div>
$$
\begin{align}
    j_\omega(t)
     & :=
    \left\{ \begin{array}{ll}
        j_m  & (0 < t \le t_1)         \\
        0    & (t_1 < t \le t_2)         \\
        -j_m & (t_2 < t \le t_3)         \\
    \end{array} \right.
    \\
    a_\omega(t)
     & :=
    \left\{ \begin{array}{ll}
        j_mt  & (0 < t \le t_1)         \\
        a_m         & (t_1 < t \le t_2)         \\
        -j_m(t-t_2) & (t_2 < t \le t_3)         \\
    \end{array} \right.
    \\
    \omega(t)
     & :=
    \left\{ \begin{array}{ll}
        \frac{1}{2}j_mt^2 & (0 < t \le t_1)          \\
        \omega_1 + a_m(t-t_1)              & (t_1 < t \le t_2)          \\
        \omega_3 - \frac{1}{2}j_m(t-t_3)^2 & (t_2 < t \le t_3)  
    \end{array} \right.
    \\
    \theta(t)
     & :=
    \left\{ \begin{array}{ll}
\frac{1}{6}j_mt^3 & (0 < t \le t_1)          \\
        \theta_1 + \omega_1(t-t_1) + \frac{1}{2}a_m(t-t_1)^2 & (t_1 < t \le t_2)          \\
        \theta_2 + \omega_3(t-t_2) - \frac{1}{6}j_m(t-t_2)^3 & (t_2 < t \le t_3)          \\
    \end{array} \right.
\end{align}
$$
</div>

### 1. パラメータを決定する
$t_1,t_2,t_3,j_m,a_m$を決定する. (ただし, $t_1=t_3-t_2=\dfrac{a_m}{j_m}$)
とりあえずMATLABで描画しながら次のようにパラメータを決定しました.

$$t_1=0.03[s]$$

$$t_2=0.07[s]$$

$$t_3=0.1[s]$$

$$j_m=3500[rad/s^3]$$

$$a_m=105[rad/s^2]$$

これらを決定したとき$\omega_3=7.35[rad/s]$となりました.

### 2. 速度の積分値が$\dfrac{\pi}{2}$となるように設計する
次のように時間$t$をおきます.
![](https://i.imgur.com/UyYlHqy.png)
面積は次から求めました.
![](https://i.imgur.com/sJGaxyC.png)

$$S_1=\theta_3=\theta(t_3)=\omega_1(t_2-t_1)+\dfrac{1}{2}a_m(t_2-t_1)^2+\omega_3(t_3-t_2)$$

$$S_2=\omega_3(t_4-t_3)$$

$2S_1+S_2=\dfrac{\pi}{2}$となるように$t_4=213[ms]$と求めました. $t_4は$整数で求まらなかったので切り上げました.

### 3.　マイコンで目標値を生成する
マイコンで生成した目標値をMATLABでplotしました.
#### 角速度
![](https://i.imgur.com/4EviluY.jpg)
#### 角加速度
![](https://i.imgur.com/LC3Smny.jpg)
#### 角躍度
![](https://i.imgur.com/TKDOxgI.jpg)

### 4. 2自由度制御をする
$$P(s)=\dfrac{0.11081}{3.2262s^2+3.10658s+1}$$

#### フィードバック　($\omega(t)$が目標値, $\omega$が測定値です. 紛らわしくてすみません.)
$$e_{\omega}=\omega(t)-\omega$$

$$u_{fb}=(k_{p_\omega}+k_{i_\omega}\dfrac{1}{s}+k_{d_\omega}\dfrac{s}{\tau s+1})e_{\omega}$$

#### フィードフォワード
$$u_{ff}=\omega(t)P^{-1}(s)=\omega(t)\dfrac{3.2262s^2+3.10658s+1}{0.11081}=\dfrac{3.2262j_{\omega}(t)+3.10658a_{\omega}(t)+\omega(t)}{0.11081}$$

#### 2自由度制御
$$u_{\omega}=u_{ff}+u_{fb}$$
