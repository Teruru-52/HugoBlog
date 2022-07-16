---
title: "Kanamaya Control Methodを用いたスラローム走行"
date: 2022-05-08T08:59:53+09:00
draft: false
description: Kanayama Control Methodを用いてスラローム走行を設計します。
tags:
  - "クラシックマウス"
  - "制御工学"
categories:
  - "技術資料"
thumbnail:
#   src: "posts/2022-05-08-kanamaya-control-method/controller.PNG"
    src: "posts/2022-05-08-kanamaya-control-method/kanayama.PNG"
---

Kanayama Control Methodを用いてスラローム走行を設計します。

<!--more-->
{{< rawhtml >}}
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config">
    MathJax.Hub.Config({tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}});
</script>
{{< /rawhtml >}}

### 1. Kanayama Control Methodとは
Kanayama Control Methodは独立二輪車のような非ホロノミック系において，目標軌道に追従させる制御手法のことです。

マイクロマウスも非ホロノミック系であり，Kanayama Control Methodを適用していきます．

この記事では以下を参考にしています。

https://www03.core.ac.uk/download/pdf/36732512.pdf

### 2. 理論を理解する
#### 2.1 最低限必要な理解
絶対座標系での位置を$p=[x,y,\theta]^\text{T}$，その目標値を$p_r=[x_r,y_r,\theta_r]^\text{T}$とします。

並進速度$v$，回転速度$\omega$をまとめたベクトルを$q=[v, \omega]^\text{T}$，その目標値を$q_r=[v_r, \omega_r]^\text{T}$とします。$v,\omega$は絶対座標，機体座標系で共通となります。

絶対座標系での位置$p$の微分$\dot p$は

{{< rawhtml >}}
$$
\dot p=\left[
\begin{matrix}
    \dot x \\
    \dot y \\
    \dot\theta
\end{matrix}
\right]=
\left[
\begin{matrix}
    v\ \text{cos}\theta \\
    v\ \text{sin}\theta \\
    \omega
\end{matrix}
\right]
$$
{{< /rawhtml >}}

と表されます。

現在の状態を$p_c=[x_c,y_c,\theta_c]^\text{T}$，$\dot p_c=[\dot x_c,\dot y_c,\dot \theta_c]^\text{T}$，$q_c=[v_c, \omega_c]^\text{T}$とすると，
機体座標系での位置の誤差$p_e=[x_e,y_e,\theta_e]^\text{T}$は次のように表されます。

![](https://i.imgur.com/QZ0wGNx.png)

{{< rawhtml >}}
$$
p_e=\left[
\begin{matrix}
    x_e \\
    y_e \\
    \theta_e
\end{matrix}
\right]=
\left[
\begin{matrix}
    (x_r - x_c)\text{cos}\theta_c + (y_r - y_c) \text{sin}\theta_c \\
    -(x_r - x_c)\text{sin}\theta_c + (y_r - y_c) \text{cos}\theta_c \\
    \theta_r - \theta_c
\end{matrix}
\right]
$$
{{< /rawhtml >}}

機体座標系での位置の誤差の微分$\dot p_e=[\dot x_e,\dot y_e,\dot\theta_e]^\text{T}$は

{{< rawhtml >}}
$$
\dot p_e=\left[
\begin{matrix}
    \dot x_e \\
    \dot y_e \\
    \dot\theta_e
\end{matrix}
\right]=
\left[
\begin{matrix}
    y_e \omega_c - v_c + v_r\text{cos}\theta_e \\
    -x_e\omega_c + v_r\text{sin}\theta_e \\
    \omega_r - \omega_c
\end{matrix}
\right]
$$
{{< /rawhtml >}}

となります。導出については資料にあります。

Kanayama Control Methodでは，誤差$p_e$を$[0,0,0]^\text{T}$に漸近収束させる速度入力$q$には

{{< rawhtml >}}
$$
q=\left[
\begin{matrix}
    v  \\
    \omega
\end{matrix}
\right]=
\left[
\begin{matrix}
    v_c  \\
    \omega_c
\end{matrix}
\right]=
\left[
\begin{matrix}
    v_r\text{cos}\theta_e + K_x x_e \\
    \omega_r + v_r(K_y y_e + K_\theta \text{sin}\theta_e)
\end{matrix}
\right]
$$
{{< /rawhtml >}}

があると証明されています。$K_x,K_y,K_\theta >0$はパラメータです。

とりあえず入力をこのようにするのだと思えば利用できます。

#### 2.2 漸近安定の証明
$q$がLyapunov安定性を満たす入力であることが示されています。

まず，速度入力を$q$としたときの誤差の微分$\dot p_e$は

{{< rawhtml >}}
$$
\dot p_e=\left[
\begin{matrix}
    \dot x_e \\
    \dot y_e \\
    \dot\theta_e
\end{matrix}
\right]=
\left[
\begin{matrix}
    \{\omega_r + v_r(K_y y_e + K_\theta \text{sin}\theta_e)\}y_e - K_x x_e\\
    -\{\omega_r + v_r(K_y y_e + K_\theta \text{sin}\theta_e)\}x_e + v_r\text{sin}\theta_e \\
    -v_r(K_y y_e + K_\theta\text{sin}\theta)
\end{matrix}
\right]
$$
{{< /rawhtml >}}

となります。

Lyapunov関数$V$を

$$V = \dfrac{1}{2}(x_e^2 + y_e^2) + \dfrac{1}{K_y}(1 - \text{cos}\theta_e)$$

とすると，その微分は

$$\dot V = x_e\dot x_e + y_e \dot y_e + \dfrac{1}{K_y}\dot\theta_e\text{sin}\theta_e
= -K_x x_e^2 - \dfrac{K_\theta}{K_y}v_r\text{sin}^2\theta_e \leq 0$$

となり，$\dot V$が平衡点$q_e=[0,0,0]^\text{T}$以外で常に負となればシステムは平衡点に漸近しますが，この式からは
$q_e=[0,y_e,0]^\text{T}$においても$\dot V = 0$を満たす可能性があるため平衡点に漸近するとは言えません。

そこで，平衡点周りで線形近似したシステムに対してRouth-Hurbitsの安定判別法を適用し，システムが安定であれば平衡点に漸近すると言えます。テイラー展開で1次線形化したシステムは

{{< rawhtml >}}
$$
\dot p_e=\left[
\begin{matrix} \dot x_e \\ \dot y_e \\ \dot\theta_e \end{matrix}
\right]=
\left[
\begin{matrix} -K_x & \omega_r & 0 \\ -\omega_r & 0  & v_r \\ 0 & -v_rK_y & -v_rK_\theta \end{matrix}
\right]
\left[
\begin{matrix} x_e \\ y_e \\ \theta_e \end{matrix}
\right]\equiv Ap_e
$$
{{< /rawhtml >}}

であり，その特性方程式は

$$|Is-A|=0$$

より

$$a_3s^3 + a_2s^2 + a_1s + a_0=0$$

{{< rawhtml >}}
$$
\left\{
    \begin{array}{l}
    a_3 =& 1\\
    a_2 =& K_\theta v_r + K_x \\
    a_1 =& K_yv_r^2 + K_xK_\theta v_r + \omega_r^2\\
    a_0 =& K_xK_yv_r^2 + \omega_r^2K_\theta v_r
    \end{array}
\right.
$$
{{< /rawhtml >}}
となります。$a_3=1$となるように整理してあります。

Hurbits行列式$H$は
{{< rawhtml >}}
$$
H=\left|
\begin{matrix} a_1 & a_3\\ a_0 & a_2 \end{matrix}
\right|=a_1a_2 - a_0a_3 > 0
$$
{{< /rawhtml >}}

となり，$a_3,a_2,a_1,a_0 > 0$かつ$H>0$であるため，システムは安定となります。

よって，速度入力$q$はシステムを平衡点$q_e=[0,0,0]^\text{T}$に漸近させることが証明されました。

### 3. 全体的なシステム
Kanayama Control Methodで求めた速度入力$v,\omega$を目標値として2自由度制御に渡します。

![](https://i.imgur.com/BK9gYIH.png)

### 4. 目標軌道を生成する
[MATLABでスラローム軌道生成](https://www.kerislab.jp/posts/2017-09-04-matlab-trajectory/)を参考にします．

目標軌道はMATLABで生成し，マイコン内で更新をしない静的軌道とします．

![](https://i.imgur.com/KWwgbNv.jpg)
目標の並進速度$v_r=0.506\ \text{m/s}$とし，$x_r,y_r,\theta_r,\omega_r$を$1\ \text{ms}$ごとに更新します．

### 5. 実装する
ゲイン$K_x,K_y,K_\theta$の調整が難しく，Kanayama Controllerから生成される目標速度$q$が振動しているようになってしまいました．

ですが，ある程度は追従できているので，並進方向の速度追従とKanayama Controllerのゲインを修正すれば良くなりそうです．

![](https://i.imgur.com/cd346Ru.jpg)