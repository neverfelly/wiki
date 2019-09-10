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

### Learning to see in the dark

- See-in-the-Dark (SID) dataset contains low-light images with paired ground truth.
- what's on: video capture, denosing, complexity compared to burst image pipeline(graphics TOG 16')
- Adopt pure FCN: based on recent works that pure FCN can effectively represent many image processing algorithms.
- process sensor data directly
- Context aggregation network and U-Net, not resnet
- Metrics: A/B tests with amazon workers , PSNR/SSIM in controlled experiments for image quality
- Loss, network architecture and data arrangement are tested in controlled experiments: shown that default pipeline outperforme than others

### Kindling the Darkness: A Practical Low-light image enhancer(KinD)

- Three challenges in low-light image enhancement:

  - Estimate the illumination component from a single image and adjust light levels
  - Remove degradations after light up dark regions
  - Train a model without well-defined ground-truth light conditions

- Contributions: 

  - trained with paired images captured under different light/exposure conditions, instead of using any ground-truth reflectance and illumination information.
  - flexibly adjust light levels
  - a module for denosing
  - State-of-the-art

- Layer decomposition: 

  - learn a mapping function from real data for light level adjustment
  - mutual consistency(edge in $\mathbf{I}$, the penalty on $\mathbf{L}$ is small, for a location in a flat, the penalty turns to be large, so it's edge awared ): $$\mathcal{L_{mc}}=||\mathbf{M}\circ \exp(-c \cdot \mathbf{M})||_1，\mathbf{M}=|\nabla\mathbf{L}_l|+|\nabla \mathbf{L}_h|$$

- Reflectance restoration

  $$\mathcal{L}=||\hat{\mathbf{R}}-\mathbf{R}||_2^2-\text{SSIM}(\hat{\mathbf{R}},\mathbf{R_h})+||\nabla\mathbf{\hat{R}}-\nabla\mathbf{R}_h||_2^2$$

  (question: why can do it? how about previous equation $$\mathbf{I}=\mathbf{R\circ I}+\mathbf{\hat{E}\circ I}$$)

- Illumination adjustment

  Two paramaters for adjustment $\alpha=\mathbf{L_t\over L_s}$,$\gamma=\mathbf{||\log{\hat{L}}||_1\over||\log{L_s}||_1}$ for gamma correction

- Metrics: PSNR,SSIM,LOE,NIQE

# References

- Guo, Xiaojie. 《LIME: A Method for Low-light IMage Enhancement》. *arXiv:1605.05034 [cs]*, 2016年5月17日. http://arxiv.org/abs/1605.05034.