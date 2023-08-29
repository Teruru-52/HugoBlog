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

|    |       |
| ---- |----|
|  マイコン  | ESP32 |
|  IMU  |  MPU-6050 |
|  モータ |   BLDC × 3　|
|  電源  |  ACアダプタ 24V 5A  |

BLDCはAmazonで購入したものです。入力のPWMにおよそ比例して速度が出力されます。
プログラムも販売されていますが，一部のピン設定を除きC++で1から書きました。

### 2. 機体
[ReM-RC](https://www.youtube.com/watch?v=AJQZFHJzwt4&list=LL&index=10&t=263s)さんのものを使わせていただいています。

### 3. 制御
#### 3.1 PD制御
倒立状態からの$x$軸周りの傾き角とその角速度を$\theta_x,\omega_x$とし，$y$軸についても同様に$\theta_y,\omega_y$とします。

HomeMadeGarbageさんとReM-RCさんの倒立振子とはセンサーの位置を変えて，頂点で倒立時にセンサの$xy$平面が地面と水平($\theta_x=0$，$\theta_y=0$)になるようにしています。

まずは，このセンサ座標系における入力$u$を考えます。
PD制御に加え，入力の積分値をフィードバックします。
入力の積分値を考慮することで，ホイールの速度が大きくなりすぎないようにしています。

$$u_x(k)=k_1\hat\theta_x(k)+k_2\hat\omega_x(k)+k_3\sum_{i=0}^{k-1}u_x(i)$$

$$u_y(k)=k_1\hat\theta_y(k)+k_2\hat\omega_y(k)+k_3\sum_{i=0}^{k-1}u_y(i)$$

$$u_z(k) = 0$$

$k_1,k_2,k_3$はゲインです。姿勢角$\hat\theta_x,\hat\theta_y$とその角速度$\hat\omega_x,\hat\omega_y$は定常カルマンフィルタによって推定しています。とりあえず倒立できればいいので，$u_z=0$としています。

#### 3.2 入力分配
入力$u_x,u_y,u_z$を3つのホイールに分配します。センサ座標系$xyz$での入力$u_x,u_y,u_z$をホイール座標$x'y'z'$での各ホイールの入力$u^\prime_{x},u^\prime_{y},u^\prime_{z}$に分配します。

まず，センサ座標系$xyz$とホイール座標系$x'y'z'$を一致させる回転行列$R$を考えます。

{{< rawhtml >}}
$$
\left[
\begin{matrix}
    x^\prime\\
    y^\prime\\
    -z^\prime\\
\end{matrix}
\right]=R
\left[
\begin{matrix}
    x\\
    y\\
    z\\
\end{matrix}
\right]
$$
{{< \rawhtml >}}

$-z^\prime$となっているのは，ホイール座標が左手座標系になっていたためです。考えやすいように$z$軸を反転して，右手座標系で統一します。

この回転は，$y$軸周りに$\beta=\dfrac{\pi}{2}-\cos^{-1}\dfrac{1}{\sqrt{3}}$回転した後，
回転後の$x^\prime$軸周りに$\alpha=-\dfrac{\pi}{4}$回転させるものになります。

$$
R=R_xR_y
$$

{{< rawhtml >}}
$$
R_x=\left[
\begin{matrix}
    1 & 0 & 0\\
    0 & \cos\alpha &\sin\alpha\\
    0 & -\sin\alpha& \cos\alpha
\end{matrix}
\right],\quad
R_y=\left[
\begin{matrix}
    \cos\beta& 0 &-\sin\alpha\\
    0 & 1 & 0\\
    \sin\beta & 0 & \cos\beta
\end{matrix}
\right]
$$
{{< /rawhtml >}}

{{< figure src="/posts/2022-07-16-3d-inverted-pendulum/matlab_rotation.jpg">}}

そして，$u^\prime_{x},u^\prime_{y},u^\prime_{z}$を求めます。

{{< rawhtml >}}
$$
\left[
\begin{matrix}
    u^\prime_{x}\\
    u^\prime_{y}\\
    -u^\prime_{z}
\end{matrix}
\right]=R
\left[
\begin{matrix}
    u_x\\
    u_y\\
    u_z\\
\end{matrix}
\right]
$$
{{< \rawhtml >}}

<!-- $$
u_xx+u_yy+u_zz=u^\prime_{x}x'+u^\prime_{y}y'+u^\prime_{z}z'
$$

$$
x = \dfrac{-y'-z'+2x'}{\sqrt6}
$$

$$
y = \dfrac{y'-z'}{\sqrt2}
$$

$$
z = -\dfrac{x'+y'+z'}{\sqrt3}
$$ -->

より，

$$
u_{x'} = \dfrac{2}{\sqrt6}u_x-\dfrac{1}{\sqrt3}u_z
$$

$$
u^\prime_{y} = -\dfrac{1}{\sqrt6}u_x+\dfrac{1}{\sqrt2}u_y-\dfrac{1}{\sqrt3}u_z
$$

$$
u^\prime_{z} = -\dfrac{1}{\sqrt6}u_x-\dfrac{1}{\sqrt2}u_y-\dfrac{1}{\sqrt3}u_z
$$

<!-- としてホイールに入力します。 -->
この回転入力$u'$となるように各ホイールに割り当てます。
制御周期$10\ \text{ms}$程度で制御しています。

### 4. Kalman Filter
用いたIMUであるMPU-6050は6軸センサなので，加速度と角速度を取得できます。

まず，加速度から直接$\theta_x,\ \theta_y$を求めます。

{{< rawhtml >}}
$$
\theta_x=\tan^{-1}\dfrac{a_y}{\sqrt{a_x^2 + a_z^2}},\quad \theta_y= \tan^{-1}\dfrac{a_x}{\sqrt{a_y^2 + a_z^2}}
$$
{{< /rawhtml >}}

差分方程式は，角速度と組み合わせて以下のようになります。

$$
\theta_{k+1}=\theta_k+(\omega_k-\omega_{\text{bias},k})\Delta{t},\quad \omega_{\text{bias},k+1}=\omega_{\text{bias},k}
$$

状態方程式で表現すると，

{{< rawhtml >}}
$$
\left[
\begin{matrix}
    \theta_{k+1}\\
    \omega_{\text{bias},k+1}
\end{matrix}
\right]=
\left[
\begin{matrix}
    1 & -\Delta{t}\\
    0 & 1
\end{matrix}
\right]
\left[
\begin{matrix}
    \theta_{k}\\
    \omega_{\text{bias},k}
\end{matrix}
\right]+
\left[
\begin{matrix}
    \Delta{t}\\
    0
\end{matrix}
\right]\omega_k
$$
{{< /rawhtml >}}

{{< rawhtml >}}
$$
\theta_k=
\left[
\begin{matrix}
    1 & 0
\end{matrix}
\right]
\left[
\begin{matrix}
    \theta_{k}\\
    \omega_{\text{bias},k}
\end{matrix}
\right]
$$
{{< /rawhtml >}}

となり，以下のように表現します。

$$
x_{k+1}=Ax_k+Bu_k+v_k
$$

$$
y_k=Cx_k+w_k
$$

$v_k,\ w_k$はそれぞれシステム雑音と観測雑音です。
このシステムに対してKalman Filterを適用します。

予測ステップで事前状態推定値$\hat x_{k}^-$と事前誤差共分散行列$P_k^{-}$を更新します。$P_{k-1}$は事後誤差共分散行列，$V$はシステム雑音の共分散行列です。

{{< rawhtml >}}
$$
\hat x_{k}^- = A\hat x_{k-1}+Bu_{k-1}
$$
{{< /rawhtml >}}

{{< rawhtml >}}
$$ 
P^{-}_k = AP_{k-1}A^\text{T}+V
$$
{{< /rawhtml >}}

フィルタリングステップではカルマンゲイン$G_k$，状態推定値$\hat x_k$，事後誤差共分散行列$P_k$を更新します。
$W$は観測雑音の共分散行列です。

{{< rawhtml >}}
$$
G_k = \dfrac{P^-_kC^\text{T}}{CP^{-}_kC^\text{T} + W}
$$
{{< /rawhtml >}}

{{< rawhtml >}}
$$
\hat x_k = \hat x_{k}^- + G_k\{y_k - C\hat x_{k}^-\}
$$
{{< /rawhtml >}}

{{< rawhtml >}}
$$
P_k = (I - G_kC)P^-_k
$$
{{< /rawhtml >}}

推定した$\hat x_k$から$\hat\theta_k$と$\hat\omega_k=\omega_k-\hat\omega_{\text{bias},k}$が求まります。

### 5. 実験結果
{{< youtube ZBS4nwo4Kak >}}

ゲインの調整がなかなか難しく，ふらつきがみられます。
ダイナミクスを考えずに瞬間的な入力を考えたので，その影響があるかもしれません。
<!-- $z$軸周りの入力$u_z$を制御に加えるのも，ふらつきを収える方法だと考えています。 -->

