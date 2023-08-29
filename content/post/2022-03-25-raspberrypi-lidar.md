---
title: "Raspberry Pi 4でLiDARを使う"
date: 2022-03-25T01:06:56+09:00
draft: true
tags:
  - "Raspberry Pi"
  - "ROS"
  - "LiDAR"
categories:
  - "技術資料"
---

<!--more-->

{{< rawhtml >}}
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
    <script type="text/x-mathjax-config">
    MathJax.Hub.Config({tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}});
</script>
{{< /rawhtml >}}

Raspberry Pi 4, ROSでLiDARを使ってみました。

|  変数 |  a  |
| ---- |----|
|  Device  |  Raspberry Pi 4 (8GB RAM)|
|  OS  |  Raspberry Pi OS |
|  LiDAR  |  [OKDO LIDAR HAT](https://www.okdo.com/p/lidar-module-with-bracket/) |
|  ROS distribution  |  Melodic |
|  Visualization Tool  |  Rviz |

OKDO LIDAR HATは[RS Components](https://jp.rs-online.com/web/p/sensor-development-tools/2037609)から購入しました。

### 1. 環境構築
以下を参考にしました。

https://docs.rs-online.com/c099/A700000007824693.pdf

https://ma38su.hatenablog.com/entry/2021/11/19/182417

Raspberry Pi OSについては, bullseyeではなくbusterでなければなりませんでした。Raspberry Pi Imagerにおいて

Raspberry Pi OS (other)　>　Raspberry Pi OS Lite (Legacy)

を選択しました。

また, 最初の起動後にユーザー名をpiから変更しないことをおすすめします。シェルスクリプトにおいて /home/pi をパスとしている部分がありました。自分でシェルスクリプトを書き直せばユーザー名を変えた上でも環境構築はできると思います。あるいは環境構築後にユーザー名を変更してもいいです。

あとは説明資料に従って進めれば環境構築できます。

### 2. LiDARを動かしてRvizで可視化する
