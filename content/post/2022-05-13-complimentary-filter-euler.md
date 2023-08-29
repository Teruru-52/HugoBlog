---
title: "相補フィルタでオイラー角を推定する"
date: 2022-05-13T03:28:34+09:00
draft: false
description: 6軸IMUと相補フィルタを用いてオイラー角を推定します。
tags:
  - "制御工学"
  - "姿勢推定"
categories:
  - "技術資料"
---

6軸IMUと相補フィルタを用いてオイラー角を推定します。

相補フィルタはカルマンフィルタと同様の働きがあり，簡単で実用性が高いです。

<!--more-->

{{< rawhtml >}}
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config">
    MathJax.Hub.Config({tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}});
</script>
{{< /rawhtml >}}

### 1. 加速度からオイラー角を求める
<!-- [EKFでオイラー角を推定する](https://teruru-52.github.io/post/2022-05-10-ekf-euler/)と同様です。 -->

ロール角$\phi$とピッチ角$\theta$は

$$
\phi = \text{tan}^{-1}\dfrac{a_y}{a_z},\quad \theta = \text{tan}^{-1}\dfrac{a_x}{\sqrt{a_y^2 + a_z^2}}
$$

となります。

### 2. ジャイロからオイラー角速度を求める
<!-- [EKFでオイラー角を推定する](https://teruru-52.github.io/post/2022-05-10-ekf-euler/)と同様です。 -->

オイラー角の運動方程式は次のようになります。

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

### 3. 相補フィルタで推定する
加速度はジャイロに比べて低周波はノイズが小さく，高周波はノイズが大きくなります。ジャイロはその逆です。
相補フィルタでは，低周波は加速度に重みをつけ，高周波はジャイロに重みをつけることでノイズの影響を低減します。

推定値$\hat\phi$と$\hat\theta$は重みパラメータ$\kappa$を用いて次のように表されます。

$$
\hat\phi_k = \kappa \phi_k + (1 - \kappa)(\hat\phi_{k-1} + \dot\phi_k\Delta t)
$$
$$
\hat\theta_k = \kappa \theta_k + (1 - \kappa)(\hat\theta_{k-1} + \dot\theta_k\Delta t)
$$

ここで，

{{< rawhtml >}}
$$
\left[
\begin{matrix}
    \dot\phi_k \\
    \dot\theta_k \\
    \dot\psi_k
\end{matrix}
\right]=
\left.\left[
\begin{matrix}
    1 & \text{sin}\phi\text{tan}\theta & \text{cos}\phi\text{tan}\theta \\
    0 & \text{cos}\phi & -\text{sin}\phi \\
    0 & \text{sin}\phi\text{sec}\theta & \text{cos}\phi\text{sec}\theta
\end{matrix}
\right]\right|_{\phi=\hat\phi_k,\;\theta=\hat\theta_k}
\left[
\begin{matrix}
    \omega_{x_k} \\
    \omega_{y_k} \\
    \omega_{z_k}
\end{matrix}
\right]
$$
{{< /rawhtml >}}

です。

この相補フィルタでは，加速度に1次LPF，ジャイロに1次HPFを通しています。
重みパラメータ$\kappa$についてはサンプリング周期$\Delta t$に加えてLPFとHPFの時定数$\tau$に依存し，次のようになります。

$$
\kappa = \dfrac{\Delta t}{\tau + \Delta t}
$$

相補フィルタではLPFとHPFのゲインは足しても常に1となり，加速度とジャイロの良い部分を取っています。

ジャイロについてはHPFを通しているので直流オフセットが除去されます。

### 4. 実践する
今後追記します。