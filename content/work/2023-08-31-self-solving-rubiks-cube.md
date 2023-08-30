---
title: "Self-Solving Rubik's Cube Robot"
date: 2023-08-31T03:16:39+09:00
tags:
  - "Rubik's Cube"
categories:
  - "Works"
---

<!--more-->

{{< rawhtml >}}
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
    <script type="text/x-mathjax-config">
    MathJax.Hub.Config({tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}});
</script>
{{< /rawhtml >}}
---

### version 1
[ルービックキューブを揃えるロボットの製作](https://teruru-52.github.io/post/2022-07-16-self-solving-rubiks-cube/)に述べています。
{{< figure src="/works/self-solving-rubiks-cube-robot/machine_v1.jpg" width="30%">}}


### version 2
{{< figure src="/posts/2022-07-16-self-solving-rubiks-cube/self_solving_robot_v2.png" width="30%">}} 
{{< figure src="/works/self-solving-rubiks-cube-robot/machine_v2.jpg" width="30%">}}
version2ではルービックキューブの取り外しを容易にしました。
カメラを取り付けて各ブロックの色を取得しようとしましたが，外光の影響を受けたので失敗。

### version 3
{{< figure src="/posts/2022-07-16-self-solving-rubiks-cube/circuit_v3.jpg" width="30%">}} 
version3では基板を発注して作成しました。
カメラ用のLEDの明るさやモータをPWMで制御できるようにしています。

{{< figure src="/posts/2022-07-16-self-solving-rubiks-cube/self_solving_robot_v3.png" width="30%">}} 
外光の反射の影響を低減するために，カメラの取り付け位置を変更し，表面がマット加工されているGANのルービックキューブにしました。
作成中。