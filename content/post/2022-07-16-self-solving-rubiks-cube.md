---
title: "自動でルービックキューブを揃えるロボットの製作"
date: 2022-07-16T19:09:07+09:00
draft: false
description: Two-Phase-Algorithmを用いて自動でルービックキューブを揃えるロボットを製作します。
tags:
  - "アルゴリズム"
  - "RASPBERRY PI"
categories:
  - "技術資料"
thumbnail:
  src: "posts/2022-07-16-self-solving-rubiks-cube/self_solving_robot.jpg"
---

Two-Phase-Algorithmを用いて自動でルービックキューブを揃えるロボットを製作します。

<!--more-->

{{< rawhtml >}}
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config">
    MathJax.Hub.Config({tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}});
</script>
{{< /rawhtml >}}

### 1. 機体設計
Fusion360で設計し，3Dプリンタで印刷しました。

![](https://i.imgur.com/hFCp8qg.png)

ルービックキューブの各面の中心ブロックを設計したものに取り替えています。
### 2. 回路
|    |       |
| ---- |----|
|  マイコン  | Raspberry Pi4B (8GB RAM) |
|  ステッピングモータドライバ  |  A4988 × 6 |
|  バイポーラステッピングモータ　|   SM-42BYG011 × 6　|
|  電源  |  ACアダプタ (12V，5A)  |

![](https://i.imgur.com/gV5mnZE.jpg)

### 3. アルゴリズム
Two-Phase-Algorithmを用いています。以下を参考にしています。

[ルービックキューブを解くプログラムを書いてみよう(前編:キューブを操る実装)](https://qiita.com/7y2n/items/a840e44dba77b1859352)


[ルービックキューブを解くプログラムを書いてみよう(中編:IDA*探索)](https://qiita.com/7y2n/items/24785b985e9c30862014)

[ルービックキューブを解くプログラムを書いてみよう(後編:状態のindex化, Two-Phase-Algorithm)](https://qiita.com/7y2n/items/55abb991a45ade2afa28)

### 4. プログラム
Pythonで書いています。動作は以下のようにしています。

1. ランダムに数手回す
1. Two-Phase-Algorithmで解を求める
1. 求めた解に従って回す

### 5. 結果
RaspberryPiにSSH接続して実行します。この動画ではランダム18手，解21手となっています。

{{< youtube ko1t7ebo6Wc >}}