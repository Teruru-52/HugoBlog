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
最初に記述した参考文献では，機体重心にかかる力と重心周りのトルクのL2ノルムが最大になるようにプロペラの方向が最適化されています。

今回は倒立振子を考えるので，回転中心は重心ではなく立方体の頂点です。
また，目的は倒立なので頂点にかかる力は考える必要がありません。
そのため，この方向最適化は倒立には不適切ですが，倒立に要するトルクは十分足りているので良しとしています。


### 2. 機体作成
|    |       |
| ---- |----|
|  マイコン  | STM32F446RE |
|  IMU  |  MPU-9250 |
|  モータ |   小型ドローン用DCモータ × 8　|
|  電源  |  Lipo 2S 240mAh  |

カーボンロッドを繋いで立方体としています。
回路基板は4枚で，組み合わせて小さな立方体になるようにしました。


### 3. 姿勢推定
以下の2つを実装して，どちらかを使うようにしています。
- Madgwick Filter
- Extended Kalman Filter (EKF)

[Madgwickフィルタでクォータニオンを推定する](Madgwickフィルタでクォータニオンを推定する)・
[EKFでクォータニオンを推定する](https://teruru-52.github.io/post/2023-07-19-ekf-quaternion/)
に記述しています。

### 4. 姿勢制御
- Nonlinear Attitude Control

参考文献に載っていたもので，クォータニオンを用いた制御です。
角速度$\omega_{\text{des}}$の求め方ですが，Lyapnov安定となるように制御設計されています。

- Linear Quadratic Regulator (LQR) 

実際にはLQRを用いています。
辺倒立では，状態をピッチ角$\theta$とその速度$\dot\theta$として状態方程式を立て，倒立点近傍で線形化しました。

### 5. 推力配分
姿勢制御で求めたトルク入力を満たすように8つのプロペラ推力に配分します。
[CVXGEN](https://cvxgen.com/docs/index.html)を用いて2次計画問題を解いています。

### 6. 実験結果
{{< youtube 2SDSoaEJgWo >}}
