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

  Two paramaters for adjustment $\alpha=\mathbf{L_t\over L_s}$,$\gamma=\mathbf{||\log{\hat{L}}||_1\over||\log{L_s}||_1}$ for gamma correction(How to use)

- Metrics: PSNR,SSIM,LOE,NIQE

### Low-light image enhancement algorithm based on retinex and generative adversarial network(Retinex-GAN)

- loss

  Regular loss: $$\mathcal{L_{reg}}={1\over mn}\sum_{i=0}^{m-1}\sum_{j=0}^{n-1}{1\over C-f(R(i,j))}$$, prevent the illumination image from approaching 1 or -1 to avoid that the network falls into a local optimal solution.

  multi-task loss: $$\mathcal{L}=\lambda_{rec}\mathcal{L_{rec}}+\lambda_{dec}\mathcal{L_{dec}}+\lambda_{com}\mathcal{L_{com}}+\lambda_{cGAN}\mathcal{L_{cGAN}}$$ 

  Smooth L1 loss: $$ \mathcal{L_{L_1}}=smooth_{\mathcal{L_1}}(x)=\begin{cases} 0.5x^2 & if \quad x < 1 \\ x-0.5 & otherwise \end{cases} $$
  
  reconstruction loss: $$\mathcal{L_{rec}}=\mathcal{L_{rec_x}}+\mathcal{L_{rec_y}}+\mathcal{L_{reg}}$$
  
  decomposition loss: $$\mathcal{L_{dec}}=\mathcal{L_{L_1}}(I_x,I_y)$$ make the image in different brightness is decomposed to the same illumination images.
  
  SSIM-MS loss: obtain the image details
  
  Composite loss: $$\mathcal{L_{com}}=\alpha \mathcal{L_{enh}}+(1-\alpha)\mathcal{L}_{ssim\_ms},\mathcal{L_{enh}}=\mathcal{L_{L_{1}}}(y,R_x\cdot I_x^{'})$$  smooth loss and SSIM to keep structure consistency
  
- Datasets: CSID, converted from see in the dark(SID) dataset

### EnlightenGAN

- without paired supervison

- Global-local discriminator

  Enhance local regions with randomly croped patches

  global loss: LSGAN loss

- Self feature preserving loss

  Preserve the image content features

  $$\mathcal{L}_{SFP}(I^L)={1\over W_{i,j}H_{i,j}}\sum_{x=1}^{W_{i,j}}\sum_{y=1}^{H_{i,j}}(\phi_{i,j}(I^L)-\phi_{i,j}(G(I^L)))^2$$ , $i$ for max pooling, $j$ for convolutional layer after max pooling

  local discriminator patches also need to be regularized.

- attention : enhance dart region more effectively

  Question: self labeled attention map?

- Metrics: NIQE

### Attention-guided low-light image enhancement

- solve the denosing and low-light enhancement simultaneously

- Datasets: propose a low-light image simulation pipeline to synthesize realistic low-light images

- Attention-net: estimate the illumination to guide the attention

  U-Net, retinex-based solution faces difficulties in handling black regions, ue-attention map could solve it.

- Ehancement-net

  decompose original problem into serveral sub-problems(noise removal, texture preserving, color correction)

  Five different network architectures(why???)

- Reinforce-net: improve contrast and details

- Loss: attention: L2. Noise: L1

  Enhancement:

  bright loss: $$\mathcal{L_{eb}}=||\mathcal{S}(\mathcal{F_e}(I, A', N')-\tilde{I})||^1, \mathcal{S}=\begin{cases} -\lambda x & x <0 \\x&x\ge 0 \end{cases}$$ , ensure sufficient brightness.

  structural loss: SSIM

  perceptual loss

  region loss: balances the degree of enhancement for different regions:$$\mathcal{L_{er}}=||I'\cdot A'-\tilde{I}\cdot A'||^1+1-\text{ssim}(I'\cdot A',\tilde{I}\cdot A')$$

  $A'$ Is the predicted ue-attention map, $N'$for noise map

  Reinforce:

  bright loss+structural loss + perceptual loss

- Metrics: PSNR, SSIM, AB, VIF, LOE, TMQI, LPIPS

- further: blocking artifacts, black regions without any texture, extremely strong noise

# References

- Guo, Xiaojie. 《LIME: A Method for Low-light IMage Enhancement》. *arXiv:1605.05034 [cs]*, 2016年5月17日. http://arxiv.org/abs/1605.05034.