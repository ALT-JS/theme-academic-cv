---
title: 'COMPSCI 280 Project2: Flow Matching'
date: '2025-03-09'
math: true
draft: false
subtitle: 'an updated version of CS180 Project 5 part B with flow matching instead of DDPM diffusion'
---

## Overview
You will train your own flow matching model on MNIST.

## Part 1: Training a Single-Step Denoising UNet
### Unconditioned UNet architecture
![fig1](https://cal-cs180.github.io/fa24/hw/proj5/assets/unconditional_arch.png)
![fig2](https://cal-cs180.github.io/fa24/hw/proj5/assets/atomic_ops_new.png)


### Visualization of the noising process
![fig3.png](https://s2.loli.net/2025/03/14/VapRt2jWhLcIC5m.png)

### Training loss curve
![fig4.png](https://s2.loli.net/2025/03/14/Xo6LJGaAh3bKsNF.png)

### Sample results on the test set after the first and the 5-th epoch
![fig5.png](https://s2.loli.net/2025/03/14/1JgCnLkd5ZFaTeu.png)
![fig6.png](https://s2.loli.net/2025/03/14/7K12zJM4ox6Icm8.png)

### Sample results on the test set with out-of-distribution noise levels after the model is trained.
![fig7.png](https://s2.loli.net/2025/03/13/tzSuKb4QPysqL7A.png)

### Sample results on the test set with pure noise
![fig8.png](https://s2.loli.net/2025/03/13/2v94YdzxhlALSi7.png)

### Average image of the training set
![fig9.png](https://s2.loli.net/2025/03/13/kI5rCa3uQsLbUoR.png)
The average MNIST image is blurry and lacks clear digits, showing general intensity distributions. The denoised results, however, retain distinct shapes, indicating that it is doing denoising.

## Part 2: Training a Flow Matching Model

### Training loss curve plot for the time-conditioned UNet over the whole training process
![fig10_tunet.png](https://s2.loli.net/2025/03/14/HaQGujkX1vgocIb.png)

### Sampling results for the time-conditioned UNet for 5 and 10 epochs
![tunet_sample.png](https://s2.loli.net/2025/03/14/JPRa2lU9YkINCFo.png)

### Training loss curve plot for the class-conditioned UNet over the whole training process
![fig10_cunet.png](https://s2.loli.net/2025/03/14/dxosKPJYICX9Tki.png)

### Sampling results for the class-conditioned UNet for 5 and 10 epochs
![cunet_sample.png](https://s2.loli.net/2025/03/14/62mLjKZDs7nkih5.png)