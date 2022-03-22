---
title: "IIRフィルタの設計"
date: 2022-03-22T03:48:06+09:00
draft: false
tags:
  - "ディジタル信号処理"
categories:
  - "技術資料"
---

低域のIIRフィルタを設計し、ジャイロセンサに適用します。ここではまずプロトタイプフィルタを考え、双1次z変換によって目的のフィルタを求める手法を用います。

### 1. 設計仕様を与える
カットオフ周波数を100Hz、阻止域端周波数を300Hz、阻止域減衰量を30dBとします。
### 2. 正規化角周波数を求める
サンプリング周期を1msとして正規化します。
### 3. プロトタイプフィルタの次数を決定する

$$t_m$$

### 4. 双1次z変換で目的のフィルタを求める

<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>

<script type="text/x-mathjax-config">
    MathJax.Hub.Config({tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}});
</script>
