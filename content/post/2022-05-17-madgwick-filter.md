---
title: "Madgwickフィルタでクォータニオンを推定する"
date: 2022-05-17T01:58:13+09:00
draft: false
description: Madgwickフィルタでクォータニオンを推定します。
tags:
  - "制御工学"
  - "IMU"
categories:
  - "技術資料"
thumbnail:
  src: "posts/2022-05-17-madgwick-filter/madgwick.PNG"
---

Madgwickフィルタでクォータニオンを推定します。
加速度とジャイロを使ったMadgwickフィルタと，地磁気を考慮したMadgwickフィルタの2つを考えます。

<!--more-->

[An efficient orientation filter for inertial and
inertial/magnetic sensor arrays](https://www.samba.org/tridge/UAV/madgwick_internal_report.pdf)を参考にしています。

MadgwickフィルタはKalmanフィルタに比べて高速，同程度以上の高精度の推定が可能なフィルタです。また，低周波での動作も可能です。

{{< rawhtml >}}
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config">
    MathJax.Hub.Config({tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}});
</script>
{{< /rawhtml >}}

### 1. 加速度とジャイロを用いる場合
#### 1.1 クォータニオン
![](https://i.imgur.com/3bRiCD8.png)

クォータニオンの表記には以下を用います。
$$
^A_B\hat q=\left[
\begin{matrix}
    q_1 & q_2 & q_3 & q_4
\end{matrix}
\right]=\left[
\begin{matrix}
    \text{cos}\theta & -r_x\text{sin}\theta & -r_y\text{sin}\theta & -r_z\text{sin}\theta 
\end{matrix}
\right]
$$

共役クォータニオンは
$$
^A_B\hat q^*=\left[
\begin{matrix}
    q_1 & -q_2 & -q_3 & -q_4
\end{matrix}
\right]
$$

と表され，${}^Av$の空間回転は
$$
^Bv={}^A_B\hat q\otimes{}^Av\otimes{}^A_B\hat q^*
$$
となります。

#### 1.2 角速度からクォータニオンの算出
角速度ベクトルを
$$
^S\omega =\left[
\begin{matrix}
    0 & \omega_x & \omega_y & \omega_z
\end{matrix}
\right]
$$

とするとクォータニオンの時間変化はテンソル積を用いて

$$
^S_E\dot q=\dfrac{1}{2} {}^S_E\hat q\otimes{}^S\omega
$$

と表されます。Madgwickフィルタでは1時刻前のクォータニオンと角速度のテンソル積をとります。

$$
^S_E\dot q_{\omega,t}=\dfrac{1}{2} {}^S_E\hat q_{\text{est},t-1}\otimes{}^S\omega_t
$$

角速度を用いてクォータニオンを求める場合は積分をします。

$$
^S_E q_{\omega,t}={}^S_E\hat q_{\text{est},t-1} + {}^S_E\dot q_{\omega,t}\Delta t
$$

#### 1.3 加速度からクォータニオンの算出
地球座標系(E)での重力加速度は既知であり，この重力加速度のベクトルにクォータニオンによる回転を施せばセンサ座標系での重力加速度ベクトル${}^S_E\hat q^*\otimes{}^E\hat g\otimes{}^S_E\hat q$が求まります。

ですが，センサ座標系(S)で計測される重力加速度${}^S\hat a$の間には誤差$f_g$が生まれます。Madgwickフィルタではこの誤差を最小化するような最適化問題を考えます。
$$
\text{min}\ \ f_g({}^S_E\hat q,{}^S\hat a)
$$

{{< rawhtml >}}
$$
f_g({}^S_E\hat q,{}^S\hat a)={}^S_E\hat q^*\otimes{}^E\hat g\otimes{}^S_E\hat q - {}^S\hat a=\left[
\begin{matrix}
    2(q_2q_4 - q_1q_3) - a_x\\
    2(q_1q_2 + q_3q_4) - a_y\\
    2(\dfrac{1}{2} - q_2^2 - q_3^2) - a_z
\end{matrix}
\right]
$$
{{< /rawhtml >}}

$$
^E\hat g =\left[
\begin{matrix}
    0 & 0 & 0 & 1
\end{matrix}
\right],\quad ^E\hat a=\left[
\begin{matrix}
    0 & a_x & a_y & a_z
\end{matrix}
\right]
$$

この最適化問題の解法として勾配降下法を用いています。

$$
^S_Eq_{k+1} = {}^S_E\hat q_k - \mu_t \dfrac{\nabla f_g}{||\nabla f_g||}
$$

$$
\nabla f_g = J_g^\text{T}({}^S_E\hat q_{\text{est},t-1})f_g({}^S_E\hat q_{\text{est},t-1},{}^S\hat a_t)
$$

{{< rawhtml >}}
$$
J_g({}^S_E\hat q) = \left[
\begin{matrix}
    -2q_3 & 2q_4 & -2q_1 & 2q_2\\
    2q_2 & 2q_1 & 2q_4 & 2q_3\\
    0 & -4q_2 & -4q_3 & 0
\end{matrix}
\right]
$$
{{< /rawhtml >}}

$$
^S_Eq_{\nabla,t} = {}^S_E\hat q_{\text{est},t-1} - \mu_t\dfrac{\nabla f_g}{||\nabla f_g||}
$$

最適な$\mu_t$については以下の通りです。
この$\mu_t$は$^S_Eq_{\nabla,t}$の収束率を${}^S_E\dot q_{\omega,t}$の大きさで制限することによって，勾配降下法におけるオーバーシュートを避けることができます。
$\alpha$は加速度や地磁気に含まれるノイズの大きさによって変えるようです。
$$
\mu_t = \alpha ||{}^S_E\dot q_{\omega,t}||\Delta t,\quad \alpha>1
$$

#### 1.4 フィルタフュージョン
相補フィルタのような形で勾配降下法から得られたクォータニオンと角速度から得られたクォータニオンを合わせます。それぞれのフィルタの時定数が角速度から求まるクォータニオンの時間微分の大きさによって変化するイメージです。パラメータ$\beta$については明確な最適値があります。$\beta$が大きいとジャイロドリフトを抑えられるが，小さいと勾配降下法の大きなステップによるノイズを抑えられるというトレードオフの関係もあります。
$$
^S_E\hat q_{\text{est},t} = \gamma_t\ {}^S_E\hat q_{\nabla,t} + (1-\gamma_t){}^S_Eq_{\omega,t}
$$

$$
\gamma_t = \dfrac{\beta}{\frac{\mu_t}{\Delta t} + \beta} = \dfrac{\beta}{\alpha ||{}^S_E\dot q_{\omega,t}|| + \beta}
$$

$\gamma_t$について微小な部分を無視すると

$$
^S_E\hat q_{\text{est},t} \approx \dfrac{\beta\Delta t}{\mu_t}{}^S_E\hat q_{\nabla,t} + (1-0){}^S_Eq_{\omega,t} = {}^S_E\hat q_{\text{est},t-1} + ({}^S_E\dot q_{\omega,t} - \beta\dfrac{\nabla f_g}{||\nabla f_g||})\Delta t
$$

となります。

ブロック図は以下のようになります。

![](https://i.imgur.com/q1m0xgh.png)

### 2. 地磁気も考慮する場合
#### 2.1 加速度と地磁気からクォータニオンを算出
地磁気から得られる値を考慮した上で最小化問題に加え，同様に勾配降下法を解きます。

$$
^S\hat m=\left[
\begin{matrix}
    0 & m_x & m_y & m_z
\end{matrix}
\right]
$$

$$
^S\hat h_t=\left[
\begin{matrix}
    0 & h_x & h_y & h_z
\end{matrix}
\right]={}^S_E\hat q_{\text{est},t-1}\otimes{}^S\hat m_t\otimes{}^S_E\hat q^*_{\text{est},t-1}
$$

$$
^E\hat b_t=\left[
\begin{matrix}
    0 & b_x & 0 & b_z
\end{matrix}
\right]=
\left[
\begin{matrix}
    0 & \sqrt{h_x^2+h_y^2} & 0 & h_z
\end{matrix}
\right]
$$

{{< rawhtml >}}
$$
f_b({}^S_E\hat q,{}^E\hat b,{}^S\hat m) = \left[
\begin{matrix}
    2b_x(\frac{1}{2} - q_3^2 - q_4^2) + 2b_z(q_2q_4 - q_1q_3) - m_x\\
    2b_x(q_2q_3 - q_1q_4) + 2b_z(q_1q_2 + q_3q_4) - m_y\\
    2b_x(q_1q_3 + q_2q_4) + 2b_z(\frac{1}{2} - q_2^2 - q_3^2) -m_z
\end{matrix}
\right]
$$
{{< /rawhtml >}}

{{< rawhtml >}}
$$
J_b({}^S_E\hat q,{}^E\hat b) = \left[
\begin{matrix}
    -2b_zq_3 & 2b_zq_4 & -4b_xq_3-2b_zq_1 & -4b_xq_4+2b_zq_2\\
    -2b_xq_4+2b_zq_2 & 2b_xq_3+2b_zq_1 & 2b_xq_2+2b_zq_4 & -2b_xq_1+2q_3\\
    2b_xq_3 & 2b_xq_4-4b_zq_2 & 2b_xq_1-4b_zq_3 & 2b_xq_2
\end{matrix}
\right]
$$
{{< /rawhtml >}}


{{< rawhtml >}}
$$
f_{g,b}({}^S_E\hat q,{}^S\hat a,{}^E\hat b,{}^S\hat m) = \left[
\begin{matrix}
    f_g({}^S_E\hat q,{}^S\hat a)\\
    f_b({}^S_E\hat q,{}^E\hat b,{}^S\hat m)
\end{matrix}
\right]
$$
{{< /rawhtml >}}

{{< rawhtml >}}
$$
J_{g,b}({}^S_E\hat q,{}^E\hat b) = \left[
\begin{matrix}
    J_g^\text{T}({}^S_E\,   \hat q)\\
    J_b^\text{T}({}^S_E\hat q,{}^E\hat b)
\end{matrix}
\right]
$$
{{< /rawhtml >}}

$$
^S_Eq_{k+1} = {}^S_E\hat q_k - \mu_t \dfrac{\nabla f_{g,b}}{||\nabla f_{g,b}||}
$$

$$
\nabla f_{g,b} = J_{g,b}^\text{T}({}^S_E\hat q_{\text{est},t-1},{}^E\hat b_t)f_{g,b}({}^S_E\hat q_{\text{est},t-1},{}^S\hat a_t,{}^E\hat b_t,{}^S\hat m_t)
$$

#### 2.2 角速度のオフセット補償
1時刻前の共役クォータニオンと勾配に直交するベクトルとのテンソル積をとって積分し，適当なパラメータ$\zeta$をかけてその時刻でのオフセットとします。
$$
{}^S\omega_{\epsilon,t} = 2{}^S_E\hat q^*_{\text{est},t-1}\otimes\dfrac{\nabla f_{g,b}}{||\nabla f_{g,b}||}
$$

$$
{}^S\omega_{b,t} = \zeta\sum_t {}^S\omega_{\epsilon,t}\Delta t
$$

$$
{}^S\omega_{c,t} = {}^S\omega_t - {}^S\omega_{b,t}
$$

$$
^S_E\dot q_{\omega,t}=\dfrac{1}{2} {}^S_E\hat q_{\text{est},t-1}\otimes{}^S\omega_{c,t}
$$

$$
^S_E q_{\omega,t}={}^S_E\hat q_{\text{est},t-1} + {}^S_E\dot q_{\omega,t}\Delta t
$$

#### 2.4 フィルタフュージョン
加速度とジャイロを用いた場合と同様に考えます。
$$
^S_E\hat q_{\text{est},t} \approx {}^S_E\hat q_{\text{est},t-1} + ({}^S_E\dot q_{\omega,t} - \beta\dfrac{\nabla f_{g,b}}{||\nabla f_{g,b}||})\Delta t
$$

ブロック図は以下のようになります。

![](https://i.imgur.com/GZj7Ncv.png)

### 3. 実践する
[プロペラを用いた倒立振子の製作](https://teruru-52.github.io/post/2023-06-03-propeller-pendulum/)の姿勢推定に用いました。

