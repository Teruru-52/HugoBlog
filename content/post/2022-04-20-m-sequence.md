---
title: "M系列信号を用いたシステム同定"
date: 2022-04-20T01:04:53+09:00
draft: false
tags:
  - "制御工学"
  - "マイクロマウス"
categories:
  - "技術資料"
thumbnail:
  src: "posts/2022-04-20-m-sequence/identification.jpg"
---

M系列信号を用いて回転方向のシステム同定をします。

<!--more-->

[マイクロマウスの機体を同定する](http://idken.net/posts/2017-06-02-systemident/)を参考にしました。

{{< rawhtml >}}
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config">
    MathJax.Hub.Config({tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}});
</script>
{{< /rawhtml >}}

### 1. M系列信号を決定する
|    |    |   |   |   |
| ---- |----| ---- | ---- | ---- |
|  シフトレジスタの長さ  | 入力の長さ | シフトレジスタのクロック周期$[s]$ | M系列信号の振幅$[V]$ | サンプリング周期$[s]$ |
|  7  |  127 | 0.2 | 3.0 | 0.02 |

MATLABを用いてM系列信号を生成しました。シフトレジスタのクロック周期は$0.2[s]$です。
![](https://i.imgur.com/XUJEn4P.jpg)
### 2. M系列信号入力の応答を得る
M系列信号1周期分の応答を得ました。サンプリング周期は$0.02[s]$です。

{{< youtube duZe7dhA7FQ >}}

![](https://i.imgur.com/eswxLee.jpg)
### 3. システム同定をする
MATLABのSystem Identification Toolboxを使用しました。

極1個、零点0個の伝達関数として同定しました。
![](https://i.imgur.com/vhyWyq5.jpg)

同定した結果は次の通りです。推定データへの適合率は77.18%でした。

$$G(s)=\dfrac{105.1}{s+37.67}$$
### 4. 同定した結果を用いて超信地旋回する
[曲線加速を用いた超信地旋回](https://teruru-52.github.io/post/2022-03-22-rotation-curved-acceleration/)をします。2自由度制御で速度追従をします。ここでは180°旋回させます。

フィードバックにおけるPIDゲインは、MATLABのpidTunerで生成したゲインを元に調整します。
![](https://i.imgur.com/qCx7JUz.jpg)

追従ができていない点については，同定精度がそこまで良くないこととゲインの調整ができていないことが原因だと思われます。

同定精度についてはM系列信号や極・零点が大きく影響しています。

より良い結果が得られ次第記事を更新します。
