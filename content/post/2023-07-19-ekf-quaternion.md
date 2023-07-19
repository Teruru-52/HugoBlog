---
title: "EKFでクォータニオンを推定する"
date: 2023-07-19T19:23:31+09:00
draft: false
description: 9軸センサに拡張カルマンフィルタ（Extended Kalman Filter:EKF）を適用してクォータニオンを推定します。
tags:
  - "制御工学"
categories:
  - "技術資料"
---

9軸センサに拡張カルマンフィルタ（Extended Kalman Filter:EKF）を適用してクォータニオンを推定します。

<!--more-->

{{< rawhtml >}}
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config">
    MathJax.Hub.Config({tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}});
</script>
{{< /rawhtml >}}

### 1. 状態方程式
クォータニオン$q=[q_0,q_1,q_2,q_3]$，角速度$\omega=[\omega_x,\omega_y,\omega_z]$，角速度のバイアス$\omega_{\text{bias}}$，システム雑音$v$とします。状態方程式は

{{< rawhtml >}}
$$
x=
\left[
\begin{matrix}
    q\\
    \omega_{\text{bias}}
\end{matrix}
\right],\quad \dot{x}=f(x)+v
$$
{{< /rawhtml >}}

{{< rawhtml >}}
$$
f(x)=
\left[
\begin{matrix}
    \dfrac{1}{2}q\otimes(\omega-\omega_{\text{bias}})\\
    0_{3}
\end{matrix}
\right]
% ,\quad
% G=
% \left[
% \begin{matrix}
%     O_{4\times3}\\
%     I_{3\times3}
% \end{matrix}
% \right]
$$
{{< /rawhtml >}}

と表されます。

### 2. 観測方程式
$w$を観測雑音とします。
観測方程式は，
加速度センサから得られる加速度$a=[0,a_x,a_y,a_z]$と，地磁気センサから得られる地磁気$m=[0,m_x,m_y,m_z]$を用いて

{{< rawhtml >}}
$$
y=h(x)+w
$$
{{< /rawhtml >}}

{{< rawhtml >}}
$$
y=
\left[
\begin{matrix}
    a\\
    m
\end{matrix}
\right],\quad
h(x)=
\left[
\begin{matrix}
    q^*\otimes a_I \otimes q\\
    q^*\otimes m_I \otimes q
\end{matrix}
\right]
$$
{{< /rawhtml >}}

と表されます。ここで，$a_I,m_I$はそれぞれ慣性座標系における加速度と地磁気の値で，

{{< rawhtml >}}
$$
a_I=
\left[
\begin{matrix}
    0\\
    0\\
    0\\
    g
\end{matrix}
\right],\quad
m_I=
\left[
\begin{matrix}
    0\\
    m_{x0}\\
    m_{y0}\\
    m_{z0}\\
\end{matrix}
\right]
$$
{{< /rawhtml >}}

とします。ここでは航空系に合わせて$z$軸を重力方向が正となるようにしています。
また，航空系では北を$x$軸，東を$y$軸とするため，$y$軸方向の地磁気を$m_{y0}=0$として扱うことができます。

### 3. EKFでオイラー角を推定する
#### 3.1 離散時間状態方程式
オイラー法により離散時間の非線形状態方程式を求めます。

{{< rawhtml >}}
$$
x_{k+1}=x_k+f(x_k)\Delta t+v_k
$$
{{< /rawhtml >}}

{{< rawhtml >}}
$$
y_k=h(x_k)+w_k
$$
{{< /rawhtml >}}

$v_k,w_k$はそれぞれシステム雑音と観測雑音です。簡単化のために，$x_k+f(x_k)\Delta t\rightarrow f(x_k)$と新しく置き換えて状態方程式を記述します。

{{< rawhtml >}}
$$
x_{k+1}=f(x_k)+v_k
$$
{{< /rawhtml >}}

#### 3.2 瞬時線形化
EKFでは$f(x_k),h(x_k)$を事後推定値$\hat x_k$周りでテイラー展開による1次近似をして扱います。

{{< rawhtml >}}
$$
f(x_k) = f(\hat x_k) + A_k(x_k - \hat x_k)
$$
{{< /rawhtml >}}

{{< rawhtml >}}
$$
h(x_k) = h(\hat x_k^-) + C_k(x_k - \hat x_k^-)
$$
{{< /rawhtml >}}

{{< rawhtml >}}
$$
A_k=\left.\dfrac{\partial f(x)}{\partial x}\right|_{x=\hat x_k},\quad
C_k=\left.\dfrac{\partial h(x)}{\partial x}\right|_{x=\hat x_k^-}
$$
{{< /rawhtml >}}

#### 3.3 EKF
予測ステップで事前状態推定値$\hat x_{k}^-$と事前誤差共分散行列$P_k^{-}$を更新します。$P_{k-1}$は事後誤差共分散行列，$V$はシステム雑音の共分散行列です。

{{< rawhtml >}}
$$
\hat x_{k}^- = f(\hat x_{k-1})
$$
{{< /rawhtml >}}

{{< rawhtml >}}
$$ 
P^{-}_k = A_{k-1}P_{k-1}A_{k-1}^\text{T}+V
$$
{{< /rawhtml >}}

フィルタリングステップではカルマンゲイン$G_k$，状態推定値$\hat x_k$，事後誤差共分散行列$P_k$を更新します。
$W$は観測雑音の共分散行列です。

{{< rawhtml >}}
$$
G_k = \dfrac{P^-_kC_k^\text{T}}{C_kP^{-}_kC_k^\text{T} + W}
$$
{{< /rawhtml >}}

{{< rawhtml >}}
$$
\hat x_k = \hat x_{k}^- + G_k\{y_k - h(\hat x_{k}^-)\}
$$
{{< /rawhtml >}}

{{< rawhtml >}}
$$
P_k = (I - G_kC_k)P^-_k
$$
{{< /rawhtml >}}

### 4. 実践する
[プロペラを用いた倒立振子の製作](https://teruru-52.github.io/post/2023-06-03-propeller-pendulum/)の姿勢推定に組み込んでいます。MadgwickフィルタとEKFを選択できるように実装しています。
