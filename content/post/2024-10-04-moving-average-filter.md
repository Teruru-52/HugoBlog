---
title: "移動平均フィルタの設計"
date: 2024-10-04T13:11:53+09:00
draft: false
description: 所望のフィルタ特性に合わせて，何回分の移動平均を取ればよいかを求めます。
tags:
  - "ディジタル信号処理"
categories:
  - "技術資料"
---

所望のフィルタ特性に合わせて，何回分の移動平均を取ればよいかを求めます。

<!--more-->

移動平均フィルタはその理解のし易さ，実装のし易さから幅広く用いられていますが，何回分の移動平均を取るかは実験によって求めている事例が多くみられる（所感）ため，設計手法を記述していきます。

[What is the cut-off frequency of a moving average filter?](https://dsp.stackexchange.com/questions/9966/what-is-the-cut-off-frequency-of-a-moving-average-filter)

[3dB-Cut off frequency of moving average](https://dsp.stackexchange.com/questions/28169/3db-cut-off-frequency-of-moving-average/28186#28186)

を参考にしています．

{{< rawhtml >}}
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config">
    MathJax.Hub.Config({tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}});
</script>
{{< /rawhtml >}}

### 1. 移動平均フィルタの伝達関数
時刻$k$における移動平均フィルタの出力は，

{{< rawhtml >}}
\begin{align}
y(k)&=\dfrac{1}{N}\sum_{i=k-N+1}^kx(i)\\
&=\dfrac{1}{N}\big\{x(k-N+1)+x(k-N+2)+\cdots+x(k-1)+x(k)\big\}
\end{align}
{{< /rawhtml >}}

となります。$z$変換により伝達関数$G(z)$は，

{{< rawhtml >}}
\begin{align}
Y(z)&=\dfrac{1}{N}(z^{-N+1}+z^{-N+2}+\cdots+z^{-1}+1)X(z)
\end{align}
{{< /rawhtml >}}

{{< rawhtml >}}
\begin{align}
G(z)=\dfrac{Y(z)}{X(z)}&=\dfrac{1}{N}(z^{-N+1}+z^{-N+2}+\cdots+z^{-1}+1)
\end{align}
{{< /rawhtml >}}

と求まります。例として2回，4回平均の例を以下に挙げます。

- 2回平均
{{< rawhtml >}}
\begin{align}
G(z)&=\dfrac{1}{２}(z^{-1}+1)
\end{align}
{{< /rawhtml >}}

- 4回平均
{{< rawhtml >}}
\begin{align}
G(z)&=\dfrac{1}{4}(z^{-3}+z^{-2}+z^{-1}+1)
\end{align}
{{< /rawhtml >}}

$N$が大きくなるほど項数が増えて伝達関数の計算が面倒に感じますが，非常に簡潔に整理することができます。

{{< rawhtml >}}
\begin{align}
G(z)&=\dfrac{1}{N}(z^{-N+1}+z^{-N+2}+\cdots+z^{-1}+1)\\
&=\dfrac{1}{N}\dfrac{(1-z^{-1})(z^{-N+1}+z^{-N+2}+\cdots+z^{-1}+1)}{1-z^{-1}}\\
&=\dfrac{1}{N}\dfrac{1-z^{-N}}{1-z^{-1}}
\end{align}
{{< /rawhtml >}}

### 2. 周波数特性
#### 2.1 ゲイン特性
周波数特性を調べるために，$z=e^{j\omega}$の変換を施します。
$\omega$は正規化角周波数です。

{{< rawhtml >}}
\begin{align}
G(j\omega)&=\dfrac{1}{N}\dfrac{1-e^{-j\omega N}}{1-e^{-j\omega}}\\
&=\dfrac{1}{N}\dfrac{e^{-\frac{j\omega N}{2}}}{e^{-\frac{j\omega}{2}}}\frac{e^{\frac{j\omega N}{2}}-e^{\frac{-j\omega N}{2}}}{e^{\frac{j\omega}{2}}-e^{\frac{-j\omega}{2}}}\\
&=\dfrac{1}{N}\dfrac{e^{-\frac{j\omega N}{2}}}{e^{-\frac{j\omega}{2}}}\frac{\sin\big(\frac{\omega N}{2}\big)}{\sin\big(\frac{\omega}{2}\big)}
\end{align}
{{< /rawhtml >}}

{{< rawhtml >}}
\begin{align}
|G(j\omega)|&=\dfrac{1}{N}\bigg|\frac{\sin\big(\frac{\omega N}{2}\big)}{\sin\big(\frac{\omega}{2}\big)}\bigg|
\end{align}
{{< /rawhtml >}}

よって，移動平均フィルタのゲイン$g(\omega)$は
{{< rawhtml >}}
\begin{align}
g(\omega)=20\log_{10}|G(j\omega)|
\end{align}
{{< /rawhtml >}}

となり，$\omega_c$をカットオフ角周波数とすると，$g(\omega_c)=-3$ [dB]近傍となるように$N$を求めればよいことになります。

#### 2.1 位相特性
位相遅れ$\phi(\omega)$は，伝達関数において

{{< rawhtml >}}
\begin{align}
\dfrac{e^{-\frac{j\omega N}{2}}}{e^{-\frac{j\omega}{2}}}=e^{-\frac{j\omega(N-1)}{2}}
\end{align}
{{< /rawhtml >}}

であるから，

{{< rawhtml >}}
\begin{align}
\phi(\omega)=-\dfrac{\omega(N-1)}{2}
\end{align}
{{< /rawhtml >}}

となります。

### 3. 設計
$g(\omega_c)=-3$ [dB]近傍となるように$N$を求めていきます。
{{< rawhtml >}}
\begin{align}
g(\omega_c)=20\log_{10}|G(j\omega_c)|=-3
\end{align}
{{< /rawhtml >}}

となる$N$を求めればよいのですが，計算が容易ではありません。
そのため，近似を使って$N$を求めていきます。

<!-- まず，
{{< rawhtml >}}
\begin{align}
g(\omega_c)=20\log_{10}\dfrac{1}{\sqrt{2}}=-3.010299...\approx-3
\end{align}
{{< /rawhtml >}}
となることから，

{{< rawhtml >}}
\begin{align}
|G(j\omega_c)|\approx\dfrac{1}{\sqrt{2}}
\end{align}
{{< /rawhtml >}}
とおきます。 -->

ここでは1つの手法として，$|G(j\omega)|$を$\omega=0$まわりのテイラー展開で2次近似します（$\omega$が高周波となる場合には注意  ）。
{{< rawhtml >}}
\begin{align}
h(\omega)=\frac{\sin\big(\frac{\omega N}{2}\big)}{\sin\big(\frac{\omega}{2}\big)}
\end{align}
{{< /rawhtml >}}
とおき，導関数を求めると
{{< rawhtml >}}
\begin{align}
h'(\omega)=\frac{N\sin\big(\frac{\omega}{2}\big)\cos\big(\frac{\omega N}{2}\big)-\cos\big(\frac{\omega}{2}\big)\sin\big(\frac{\omega N}{2}\big)}{2\sin^2\big(\frac{\omega}{2}\big)}
\end{align}
{{< /rawhtml >}}

{{< rawhtml >}}
\begin{align}
h''(\omega)=\cdots\ (\text{omission})
\end{align}
{{< /rawhtml >}}

となり，$\omega=0$において
{{< rawhtml >}}
\begin{align}
h(0)&=N\\
h'(0)&=0\\
h''(0)&=\dfrac{N(1-N^2)}{12}
\end{align}
{{< /rawhtml >}}

であるから，2次近似は
{{< rawhtml >}}
\begin{align}
|G(j\omega)|&\approx \dfrac{1}{N}\big(h(0)+h'(0)\omega+\dfrac{h''(0)}{2!}\omega^2\big)\\
&=1+\dfrac{1-N^2}{24}\omega^2
\end{align}
{{< /rawhtml >}}

となります。よって，

{{< rawhtml >}}
\begin{align}
|G(j\omega_c)|\approx1+\dfrac{1-N^2}{24}\omega_c^2=10^{-\frac{3}{20}}
\end{align}
{{< /rawhtml >}}

{{< rawhtml >}}
\begin{align}
\omega_c=\sqrt{\dfrac{24\times(1-10^{-\frac{3}{20}})}{N^2-1}}
\end{align}
{{< /rawhtml >}}

{{< rawhtml >}}
\begin{align}
N=\sqrt{1+\dfrac{24\times(1-10^{-\frac{3}{20}})}{\omega_c^2}}
\end{align}
{{< /rawhtml >}}

となり，$N$において小数点以下は切り上げすればよいことになります。

### 6. 例
カットオフ周波数$f_c=50$ [Hz]，サンプリング周波数$f_s=1000$ [Hz]について考えてみます。
正規化角周波数は

{{< rawhtml >}}
\begin{align}
\omega_c=2\pi\dfrac{f_c}{f_s}=0.3141...
\end{align}
{{< /rawhtml >}}

より，

{{< rawhtml >}}
\begin{align}
N=\sqrt{1+\dfrac{24\times(1-10^{-\frac{3}{20}})}{\omega_c^2}}=8.4864...
\end{align}
{{< /rawhtml >}}

となり，切り上げによって$N=9$と求まります。

### 5. まとめ
参考にした記事では，近似の精度をあげるために様々な手法が挙げられています。
上述した設計法は2次近似した$\omega=0$から大きく離れる場合には注意が必要で，あくまで目安として使うべきだいうのが個人の見解です。
近似の精度をあげるか，結局はボード線図をみて設計をするのが良いと思います。
