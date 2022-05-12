---
title: "2次元フライホイール倒立振子の製作"
date: 2022-05-12T23:32:37+09:00
draft: false
description: 2次元フライホイール倒立振子を製作します。
tags:
  - "制御工学"
  - "倒立振子"
categories:
  - "技術資料"
---

2次元フライホイール倒立振子を製作します。

<!--more-->

{{< rawhtml >}}
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config">
    MathJax.Hub.Config({tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}});
</script>
{{< /rawhtml >}}

### 1. 運動方程式
物理変数を以下に示します。

|  物理変数 |    |
| ---- |----|
|  $m_b$  |  ボディの質量|
|  $m_w$  |  ホイールの質量 |
|  $I_b$  |  ピボット点周りのボディの慣性モーメント |
|  $I_w$  |  ホイール回転軸周りのホイールの慣性モーメント |
|  $l_b$  |  ボディ重心とピボット点間の距離 |
|  $l_w$  |  ホイール回転軸とピボット点間の距離 |
|  $c_b$  |  ボディの回転動摩擦係数 |
|  $c_w$  |  ホイールの回転動摩擦係数 |
|  $i$  |  モータへの入力電流 |
|  $\tau$  |  ホイールに作用するモータトルク |
|  $k_\tau$  |  モータのトルク定数 |
|  $\theta_b$  |  ピボット点周りのボディの回転角 |
|  $\dot\theta_w$  |  ホイールの回転速度 |

ピボット点周りのボディの運動方程式は次の通りです。

$$
(I_b + m_wl_w^2) \ddot\theta_b = (m_bl_b + m_wl_w)g\ \text{sin}\theta_b - (\tau - c_w\theta_b) - c_b\dot\theta_b
$$

ホイール回転軸周りのホイールの運動方程式は次の通りです。

$$
I_w(\ddot\theta_b + \ddot\theta_w) = \tau -c_w\dot\theta_w
$$

### 2. 状態方程式
#### 2.1 非線形状態方程式
状態量を$\theta_b,\dot\theta_b,\dot\theta_w$とします。
運動方程式より，$\ddot\theta_b,\ddot\theta_w$は次のようになります。

$$
\ddot\theta_b = \dfrac{(m_bl_b + m_wl_w)g\ \text{sin}\theta_b - (\tau - c_w\dot\theta_w) - c_b\dot\theta_b}{I_b + m_wl_w^2}
$$

$$
\ddot\theta_w = \dfrac{(I_b + I_w + m_wl_w^2)(\tau - c_w\dot\theta_w)}{I_w(I_b + m_wl_w^2)} - \dfrac{(m_bl_b + m_wl_w)g\ \text{sin}\theta_b - c_b\dot\theta_b}{I_b + m_wl_w^2}
$$

非線形状態方程式は以下のようになります。
$$
\dot x = f(x) + Bu
$$

{{< rawhtml >}}
$$
\dot x=\left[
\begin{matrix}
    \dot\theta_b\\
    \ddot\theta_b\\
    \ddot\theta_w 
\end{matrix}
\right],\quad u = i,\quad \tau = k_\tau u
$$

$$
f(x)=\left[
\begin{matrix}
    \dot\theta_b\\
    \dfrac{(m_bl_b + m_wl_w)g\ \text{sin}\theta_b + c_w\dot\theta_w - c_b\dot\theta_b}{I_b + m_wl_w^2}\\
    -\dfrac{(I_b + I_w + m_wl_w^2)c_w\dot\theta_w}{I_w(I_b + m_wl_w^2)} - \dfrac{(m_bl_b + m_wl_w)g\ \text{sin}\theta_b - c_b\dot\theta_b}{I_b + m_wl_w^2}

\end{matrix}
\right]$$

$$
B=\left[
\begin{matrix}
    0\\
    -\dfrac{k_\tau}{I_b + m_wl_w^2}\\
    \dfrac{k_\tau(I_b + I_w + m_wl_w^2)}{I_w(I_b + m_wl_w^2)}
\end{matrix}
\right]
$$
{{< /rawhtml >}}

#### 2.2 線形化
$f(x)$を平衡点$x = 0$周りでテイラー展開で1次近似します。

$$
f(x)=Ax
$$

{{< rawhtml >}}
$$
A=\left.\dfrac{\partial f}{\partial x}\right|_{x=0}=
\left.\left[
\begin{matrix}
    \dfrac{\partial \dot\theta_b}{\partial \theta_b} & \dfrac{\partial \dot\theta_b}{\partial \dot\theta_b} & \dfrac{\partial \dot\theta_b}{\partial \dot\theta_w}\\
    \dfrac{\partial \ddot\theta_b}{\partial \theta_b} & \dfrac{\partial \ddot\theta_b}{\partial \dot\theta_b} & \dfrac{\partial \ddot\theta_b}{\partial \dot\theta_w}\\
    \dfrac{\partial \ddot\theta_w}{\partial \theta_b} & \dfrac{\partial \ddot\theta_w}{\partial \dot\theta_b} & \dfrac{\partial \ddot\theta_w}{\partial \dot\theta_w}\\
\end{matrix}
\right]\right|_{x=0}=
\left[
\begin{matrix}
    0 & 1 & 0 \\
    \dfrac{(m_bl_b + m_wl_w)g}{I_b + m_wl_w^2} & -\dfrac{c_b}{I_b + m_wl_w^2} & \dfrac{c_w}{I_b + m_wl_w^2}\\
    -\dfrac{(m_bl_b + m_wl_w)g}{I_b + m_wl_w^2} & \dfrac{c_b}{I_b + m_wl_w^2} & -\dfrac{c_w(I_b + I_w + m_wl_w^2)}{I_b + m_wl_w^2}
\end{matrix}
\right]
$$
{{< /rawhtml >}}

よって線形状態方程式は
$$\dot x = Ax + Bu$$
となり，システムは可制御です。

また，出力方程式は

$$
y=Cx
$$

{{< rawhtml >}}
$$
C=
\left[
\begin{matrix}
    1 & 0 & 0\\
    0 & 1 & 0\\
    0 & 0 & 1
\end{matrix}
\right]
$$
{{< /rawhtml >}}

となり，システムは可観測です。

#### 2.3 離散化
0次ホールドで離散化します。制御周期は$t_s$とします。

$$
x_{k+1} = A_dx_k + B_du_k
$$

$$
y_k = C_dx_k
$$

{{< rawhtml >}}
$$
A_d=e^{At_s},\quad B_d = \int_{0}^{t_s} e^{At}dtB,\quad C_d = C
$$
{{< /rawhtml >}}

### 3. 最適制御
LQRの最適制御を設計します。

$$u_k = -K_dx_k$$

### 4. $\theta_b, \dot\theta_b$の測定
今後追記します。

### 5. 実機製作
今後追記します。