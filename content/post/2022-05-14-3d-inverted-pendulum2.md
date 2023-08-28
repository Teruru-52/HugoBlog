---
title: "状態空間表現を用いた3次元倒立振子の製作"
date: 2022-05-14T08:08:32+09:00
draft: false
description: 状態空間表現を用いて，3次元フライホイール倒立振子を製作します。
tags:
  - "制御工学"
  - "倒立振子"
categories:
  - "技術資料"
---

状態空間表現を用いて，3次元フライホイール倒立振子を製作します。

<!--more-->

[The Cubli: A Reaction Wheel Based 3D Inverted Pendulum](https://ethz.ch/content/dam/ethz/special-interest/mavt/dynamic-systems-n-control/idsc-dam/Research_DAndrea/Cubli/Cubli_ECC2013.pdf)を参考にしています。

{{< rawhtml >}}
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config">
    MathJax.Hub.Config({tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}});
</script>
{{< /rawhtml >}}

### 1. 機体
#### 1.1 CAD
{{< figure src="/posts/2022-05-14-3d-inverted-pendulum2/cubli_cad.png">}}

#### 1.2 回路
|    |       |
| ---- |----|
|  マイコン(Main Board)  | STM32F405RGT6 |
|  IMU  |  MPU-6500 × 6 |
|  BLMD  |  自作 × 3|
|  BLDC　|   EC 45 flat (30W) × 3　|
|  電源  |  Lipo 3S 850mAh  |

### 2. 運動方程式
物理変数を以下に示します。

|  物理変数 |    |
| ---- |----|
|  $m_b$  |  ボディの質量|
|  $m_w$  |  ホイールの質量|
|  $r_b$  |  ボディ座標系の質量中心位置|
|  $r_w$  |  ボディ座標系のホイール質量中心位置 |
|  $\omega_b$  |  ボディ座標系のボディの角速度 |
|  $\omega_w$  |  各ホイールの角速度 |
|  $J_b$  |  ボディの慣性テンソル |
|  $J_w$  |  ホイールの慣性テンソル |
|  $c_w$  |  ホイールの回転動摩擦係数 |
|  $u$  |  各モータへの入力電流 |
|  $\tau$  |  ホイールに作用するモータトルク |
|  $k_\tau$  |  モータのトルク定数 |
|  $g_b$  |  ボディ座標系での重力加速度ベクトル |
|  $g$  |  慣性座標系での重力加速度（スカラー） |

ZYXオイラー角の運動方程式は次の通りです。

{{< rawhtml >}}
$$
\left[
\begin{matrix}
    \dot\phi \\
    \dot\theta \\
    \dot\psi
\end{matrix}
\right]=
\left[
\begin{matrix}
    1 & \text{sin}\phi\text{tan}\theta & \text{cos}\phi\text{tan}\theta \\
    0 & \text{cos}\phi & -\text{sin}\phi \\
    0 & \text{sin}\phi\text{sec}\theta & \text{cos}\phi\text{sec}\theta
\end{matrix}
\right]
\left[
\begin{matrix}
    \omega_x \\
    \omega_y \\
    \omega_z
\end{matrix}
\right]
$$
{{< /rawhtml >}}

$r=[\phi,\theta,\psi]$，$\omega_b=[\omega_x,\omega_y,\omega_z]$として，

$$
\dot r = \Phi(r)\omega_b
$$

とおきます。

ボディとホイールの回転方向の運動方程式は

$$
\hat J\dot\omega_b = J\omega_b\times\omega_b + Mg_b + J_w\omega_w\times\omega_b
$$

$$
J_w\omega_w = (k_\tau u - c_w\omega_w) - J_w\omega_b
$$

と表されます。ここで，

{{< rawhtml >}}
$$
g_b=\left[
\begin{matrix}
    g\ \text{sin}\theta \\
    -g\ \text{sin}\phi\text{cos}\theta \\
    -g\ \text{cos}\phi\text{cos}\theta
\end{matrix}
\right]
$$

$$
M = m_b\tilde r_b + \sum_{i=1}^{3}m_{w_i}\tilde r_{w_i}
$$
{{< /rawhtml >}}

{{< rawhtml >}}
$$
J = J_b - m_b\tilde r_b^2 + \sum_{i=1}^{3}(J_{w_i} - m_w\tilde r_{w_i}^2)
$$
{{< /rawhtml >}}

$$
J_w = \text{diag}(J_{w_1},J_{w_2},J_{w_3})
$$

$$
\hat J  =J - J_w
$$

です。チルダは外積行列を表し，歪対称行列となります。

### 3. 状態方程式
### 3.1 非線形状態方程式
状態ベクトルを$x=[r,\omega_b,\omega_w]$とします。

$$
\dot x = f(x) + Bu
$$

{{< rawhtml >}}
$$
f(x)=
\left[
\begin{matrix}
    \Phi(r) \omega_b\\
    \hat J^{-1}(J\omega_b\times\omega_b + Mg_b + J_w\omega_w\times\omega_b + c_w\omega_w)\\
    -\hat J^{-1}(J\omega_b\times\omega_b + Mg_b + J_w\omega_w\times\omega_b) - (J_w^{-1}+\hat J^{-1})c_w\omega_w
\end{matrix}
\right]
$$
{{< /rawhtml >}}

{{< rawhtml >}}
$$
B=
\left[
\begin{matrix}
    0_{3\times3}\\
    -\hat J^{-1} k_\tau\\
    (J_w^{-1}+\hat J^{-1})k_\tau
\end{matrix}
\right]
$$
{{< /rawhtml >}}

### 3.2 線形化
平衡点$x_d=[r_d,\omega_{b_d},\omega_{w_d}]$周りで1次近似します。

$r_d = [\phi_d,\theta_d,\psi_d]$，$\omega_{b_d}=0\in R^3$，$\omega_{b_d}=0\in R^3$です。

頂点で倒立する際は$\phi_d=\dfrac{\pi}{4},\theta_d=\text{atan}\dfrac{1}{\sqrt{2}},\psi_d=$(任意)となります。

線形状態方程式は次のようになります。

$$
\dot{\hat x} = A\hat x + B\hat u
$$

{{< rawhtml >}}
$$
A=
\left[
\begin{matrix}
    0_{3\times3} & \Phi(r_d) & 0_{3\times3}\\
    \hat J^{-1}M\left.\dfrac{\partial g_b(r)}{\partial r}\right|_{r=r_d} & 0_{3\times3} & \hat J^{-1}c_w\\
    -\hat J^{-1}M\left.\dfrac{\partial g_b(r)}{\partial r}\right|_{r=r_d} & 0_{3\times3} & - (J_w^{-1}+\hat J^{-1})c_w
\end{matrix}
\right]
$$
{{< /rawhtml >}}

$$
\hat x = x - x_d,\quad \hat u = u - u_d
$$

{{< rawhtml >}}
$$
\left.\dfrac{\partial g_b(r)}{\partial r}\right|_{r=r_d}=\left[
\begin{matrix}
    0 & \text{cos}\theta_d & 0\\
    -\text{cos}\phi_d\text{cos}\theta_d & \text{sin}\phi_d\text{sin}\theta_d & 0\\
    \text{sin}\phi_d\text{cos}\theta_d & \text{cos}\phi_d\text{sin}\theta_d & 0 
\end{matrix}
\right]
$$
{{< /rawhtml >}}

#### 3.3 直接制御できない状態量を除去
線形状態方程式

$$
\dot{\hat x} = A\hat x + B\hat u
$$

において，ヨー角の角速度$\dot\psi$はボディの角速度$\omega_b$のみの影響を受けます。
また，入力行列$B$の第3行は0となるため，入力$u$によって$\dot\psi$を直接制御することができません。

ヨー角$\psi$の方程式はその角速度$\dot\psi$の積分によって記述されますが，直接6軸センサからヨー角$\psi$を求めることはできません。
角速度$\dot\psi$を積分してフィードバックすれば良いと思うかもしれませんが，6軸センサではセンサの$z$軸周りのジャイロのオフセットを除去することが
できないので，積分するとドリフトが生じてしまいます。

そのため，ヨー角$\psi$を状態量から取り除きます。$\psi$を取り除いた線形状態方程式を

$$
\dot{\bar x} = \bar A\bar x + \bar B\bar u
$$

$$
\bar y = \bar C\bar x
$$
とします。

### 4. フィードバック制御
#### 4.1 可制御正準分解
$(\bar A,\bar B)$に関する可制御性行列$V$において$\text{rank}V=7<8$となるので，このままでは不可制御です。
そこで，可制御正準分解をして可制御な部分のみでフィードバック制御します。

$$
\text{rank}V=\text{rank}\left[
\begin{matrix}
    \bar B & \bar A\bar B & \cdots & \bar A^7\bar B
\end{matrix}
\right]=7<8
$$

となっていると，$1\times8$行列$P$が存在して

$$
P\left[
\begin{matrix}
    \bar B & \bar A\bar B & \cdots & \bar A^7\bar B
\end{matrix}
\right]=0
$$

となります。Cayley-Hamiltonの定理より

$$
P\bar A\left[
\begin{matrix}
    \bar B & \bar A\bar B & \cdots & \bar A^7\bar B
\end{matrix}
\right]=0
$$

が成り立ち，0でない$\bar A_3$が存在して$P\bar A = \bar A_3P$と書けます。ここで，$z_2=P\bar x$とおくと

$$
\dot z_2 = P\bar A\bar x + P\bar B\bar u = P\bar A\bar x = \bar A_3P\bar x = \bar A_3z_2
$$
となります。ここで，次の座標変換を考えます。

{{< rawhtml >}}
$$
z=T\bar x=\left[
\begin{matrix}
    Q\\
    P
\end{matrix}
\right]\bar x
$$
{{< /rawhtml >}}

ただし，$T$が正則となるように$Q$を選びます。すると，変換後のシステムは

{{< rawhtml >}}
$$
\dot z = \left[
\begin{matrix}
    \bar A_1 & \bar A_2\\
    0 & \bar A_3
\end{matrix}
\right]z+
\left[
\begin{matrix}
    \bar B_1\\
    0
\end{matrix}
\right]\bar u
$$
{{< /rawhtml >}}

{{< rawhtml >}}
$$
z = \left[
\begin{matrix}
    z_1\\
    z_2
\end{matrix}
\right]
$$
{{< /rawhtml >}}

の形になります。よって，可制御なシステムは

$$
\dot z_1 = \bar A_1z_1 + \bar B_1\bar u
$$

となります。

#### 4.2 離散化
0次ホールドで離散化します。制御周期は$t_s$とします。

$$
z_{1_{k+1}} = \bar A_{1_d}z_{1_{k}} + \bar B_{1_d}\bar u_k
$$

{{< rawhtml >}}
$$
\bar A_{1_d}=e^{\bar A_1t_s},\quad \bar B_{1_d} = \int_{0}^{t_s} e^{\bar A_1t}dt\bar B_1
$$
{{< /rawhtml >}}

#### 4.3 最適制御
離散時間LQRの最適制御を設計します。

$$\bar u_k = -\bar K_d\bar z_{1_k}$$

$\text{rank}V=7<8$より不可制御な次元が1次元あり，可制御正準分解で可制御な部分を抜き出して制御をします。
そのため，具体的にはボディの角速度とホイールの角速度の両方を同時に0にするようなことはできません。

### 5. 姿勢推定
6つのセンサでそれぞれ測定した加速度から，重力ベクトルを推定します。

それぞれのセンサの位置ベクトルを$p_i (i=1,2,...,6)$とします。
測定した加速度は，センサが位置する点の加速度ベクトル$\ddot{p}_i$と重力ベクトル$^Bg$の和で表されます。

$$^Bm_i=\ ^B\ddot{p}_i+\ ^Bg$$

ここで，

$$^B\ddot{p}_i=\ ^B_IR\ ^I\ddot{p}_i=\ ^B_IR\ ^I_B\ddot{R}\ ^Bp_i$$

であり，$\tilde{R}=\ ^B_IR\ ^I_B\ddot{R}$とすると，

$$^Bm_i=\tilde{R}\ ^Bp_i+\ ^Bg$$

と表せます。この加速度の測定値$m_i$から，重力ベクトル$^Bg$を最小二乗推定します。

{{< rawhtml >}}
$$
\left[
\begin{matrix}
    ^Bm_1&    ^Bm_2&    ^Bm_3&    ^Bm_4&    ^Bm_5&    ^Bm_6\\
\end{matrix}
\right]=
\left[
\begin{matrix}
    ^Bg & \tilde{R}
\end{matrix}
\right]
\left[
\begin{matrix}
    1 & 1 & 1 & 1 & 1 & 1\\
    ^Bp_1 & ^Bp_2 & ^Bp_3 & ^Bp_4 & ^Bp_5 & ^Bp_6
\end{matrix}
\right]
$$
{{< /rawhtml >}}

$$M=QP$$

$Q$の推定値$\hat{Q}$を疑似逆行列$P^\dagger$を用いて求めると，$\hat{Q}(:,1)$が推定したい重力ベクトル$\hat{g}$に対応します。

$$\hat{Q}=MP^\dagger=MP^\text{T}(PP^\text{T})^{-1}$$

また，ボディの角速度$\omega_b$は6つのセンサの平均値を取って求めます。
推定した重力ベクトル$\hat{g}$とボディの角速度$\omega_b$から，[相補フィルタでオイラー角を推定する](https://teruru-52.github.io/post/2022-05-13-complimentary-filter-euler/)でオイラー角を推定します。

### 6. 実践する
今後追記します。