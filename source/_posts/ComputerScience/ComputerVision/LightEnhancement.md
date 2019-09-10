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
- - SSR 算法流程：![image-20190905214238128](./ssr.png)
  
  - MSR:$r(x,y)=\sum_{k}w_k(\log S(x,y)-\log F_k(x,y)\cdot S(x,y))$ 
  
  - LIME：1.Initial Illumination map estimation $$\textbf{L}={\textbf{S}-1+a\over 1-{1\over a}+\max_{c}{\textbf{S}^c \over a}+\epsilon} + (1-a)$$  2. Illumination map refinement  $$ \min _{\mathbf{T}}\|\hat{\mathbf{T}}-\mathbf{T}\|_{F}^{2}+\alpha \sum_{x} \frac{\mathbf{W}_{h}(x)\left(\nabla_{h} \mathbf{T}(x)\right)^{2}}{\left|\nabla_{h} \hat{\mathbf{T}}(x)\right|+\epsilon}+\frac{\mathbf{W}_{v}(x)\left(\nabla_{v} \mathbf{T}(x)\right)^{2}}{\left|\nabla_{v} \hat{\mathbf{T}}(x)\right|+\epsilon} $$ conclusion: best traditional method
- Retinex based methods suffer from the limitation in model capacity of the decomposition for reflectance and illumination

## DeepMethods

### Retinex-Net

- deep learning based
- end-to-end
- Three stages: decomposition, enhancement, reconstruction
- decomposition: Data driven by nerual network， input: paired normal light and low light, key: low-light image and normal-light image share the same reflectance. 
- loss: $$\mathcal{L}=\mathcal{L_{recon}}+\mathcal{\lambda_{r}L_r}+\lambda_{s}\mathcal{L_s}$$, $$\mathcal{L_{recon}}=\sum_i\sum_j\lambda_{ij}\vert\vert R_i \circ I_j-S_j\vert\vert_1, i \in （low,normal), j \in (low,normal)$$, $$\mathcal{L_r}=||R_{low}-R_{normal}||_1$$ , **novel ** improved structure-blinding TV loss(Total variation minimizati) to Structure-Aware Smoothness Loss $$\mathcal{L_s}=\sum_i||\nabla I_i \circ \exp (-\lambda_g \nabla R_i))||$$ , compared with LIME: LIME is weighted by an inital illumination map estimation, rather it is weighted by reflectance.
- Enhancement: encoder-decoder architecture, **multi-scale concatenation** : each resize by nearest-neighbor interpolation to final scale, concatenate to $C \times M$ channel features map, then by convolutional layer reduced and reconstruct the illumination map $\tilde{I}$. Resized-convolutional layer in up-sampling block used for avoid checkerboard pattern of artifacts.
- Loss: $$\mathcal{L}=\mathcal{L_{recon}}+\mathcal{L_s}, \mathcal{L_{recon}}=||R_{low}\circ \hat{I}-S_{normal}||_1$$ 
- Denoising operated on reflectance

# References

- Guo, Xiaojie. 《LIME: A Method for Low-light IMage Enhancement》. *arXiv:1605.05034 [cs]*, 2016年5月17日. http://arxiv.org/abs/1605.05034.