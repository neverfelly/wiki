---
title: LightEnhancement
toc: false
date: 2019-08-25 11:30:16
tags: [DeepLearning, Visions]
---
# Methods

## Retinex

- tranditional vision
- Based on equation $S(x, y)=R(x,y)L(x,y)$
- $r(x,y)=\log S(x,y)-\log F(x,y)\cdot S(x,y)$ ,$\cdot$denote convolution，$F(x,y)=\lambda e^{-(x^2+y^2)\over c^2}$, which must statisify $\int \int F(x,y)dxdy=1$
- - SSR 算法流程：![image-20190905214238128](ssr.png)
  - MSR:$r(x,y)=\sum_{k}w_k(\log S(x,y)-\log F_k(x,y)\cdot S(x,y))$ 
- Retinex based methods suffer from the limitation in model capacity of the decomposition for reflectance and illumination

## LIME

- tranditional

## Retinex-Net

- deep learning based
- end-to-end