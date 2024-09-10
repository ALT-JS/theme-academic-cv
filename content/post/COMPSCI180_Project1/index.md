---
title: 'COMPSCI 180 Project1'
date: '2024-09-03'
subtitle: 'Images of the Russian Empire: Colorizing the Prokudin-Gorskii photo collection'
image:
  caption: 'Left: Canny & ImagePyramid tuned. Right: stacked without alignment.'
  focal_point: ''
  placement: 1
  preview_only: false
---

## Part1: Background & Overview
The goal of this assignment is to take the digitized Prokudin-Gorskii glass plate images and, using image processing techniques, automatically produce a color image with as few visual artifacts as possible. First, I cropped the pictures into three parts, and treat them as blue channel, green channel and red channel. Then, the vanilla method is just to stack the three channels together, as shown below.

![12color_image.jpg](https://s2.loli.net/2024/09/04/6ByAnf7ZY3d2gV4.jpg)
<p style="text-align: center;"><em>Picture Selected: Cathedral</em></p>

The easiest attempt clearly has two drawbacks. 
One is that the three channels of pictures don't crop quite well, resulting in the huge colored bands on the edge of the stacked picture. 
The other is that the three channels of pictures don't align quite well, resulting in the blur and gleming of the stacked picture.

## Part2: Simple Image Alignment

The easiest way to align the parts is to exhaustively search over a window of possible displacements (say [-15,15] pixels), score each one using some image matching metric, and take the displacement with the best score. I choose the easier way of calculating L2 norm:
```python
# L2_norm calculation
def L2_norm(image1, image2):
    return np.sqrt(np.sum((image1 - image2) ** 2))
```
Using dual for loop, the simple attempt can exhaustively search over a window of possible displacements, compare L2 norm values 256 times, and apply the displacement with the smallest L2 norm, indicating the best alignment. With the simple alignment method, I am able to reconstruct low-res pictures like the cathedral well. **The "Best matching displacement(Using blue channel as base image)" applies to all results shown below.**

*For green channel: move -1 on x-axis, move 1 on y-axis.*

*For red channel: move -1 on x-axis, move 7 on y-axis.*
<table>
  <tr>
    <td align="center">
      <img src="https://s2.loli.net/2024/09/04/6ByAnf7ZY3d2gV4.jpg" alt="Stacked w/o simple alignment" width="400"><br>
      <em>Stacked w/o simple alignment</em>
    </td>
    <td align="center">
      <img src="https://s2.loli.net/2024/09/04/FPh2fRt3nWvjaAc.jpg" alt="Stacked w/ simple alignment" width="400"><br>
      <em>Stacked w/ simple alignment</em>
    </td>
  </tr>
</table>

Also, I noticed that the picture has black edges that only bear unnecessary informations and can cause poor results in the simple alignment method. So I revised the simple alignment method by adding one more step of cropping the black edges. With the preprocessing cropping used, the simple alignment algorithm can achieve fine results on other low-res pictures like the monastery and the tobolsk.

*For green channel: move 2 on x-axis, move -3 on y-axis.*

*For red channel: move 2 on x-axis, move 3 on y-axis.*
<table>
  <tr>
    <td align="center">
      <img src="https://s2.loli.net/2024/09/04/B8fsCXc61GmOx2k.jpg" alt="Stacked w/o simple alignment" width="400"><br>
      <em>Stacked w/ cropping w/o simple alignment</em>
    </td>
    <td align="center">
      <img src="https://s2.loli.net/2024/09/04/zghJ3mGdqcIenr5.jpg" alt="Stacked w/ simple alignment" width="400"><br>
      <em>Stacked w/ simple alignment</em>
    </td>
  </tr>
</table>

*For green channel: move 2 on x-axis, move 3 on y-axis.*

*For red channel: move 3 on x-axis, move 6 on y-axis.*
<table>
  <tr>
    <td align="center">
      <img src="https://s2.loli.net/2024/09/04/v1O5ZT8dEYtDAFy.jpg" alt="Stacked w/o simple alignment" width="400"><br>
      <em>Stacked w/ cropping w/o simple alignment</em>
    </td>
    <td align="center">
      <img src="https://s2.loli.net/2024/09/04/YNiUTPA69DmcLu2.jpg" alt="Stacked w/ simple alignment" width="400"><br>
      <em>Stacked w/ simple alignment</em>
    </td>
  </tr>
</table>

## Part3: Image alignment using image pyramid
Exhaustive search will become prohibitively expensive if the pixel displacement is too large (which will be the case for high-resolution glass plate scans). In this case, we can implement a faster search procedure such as an image pyramid. An image pyramid represents the image at multiple scales (usually scaled by a factor of 2) and the processing is done sequentially starting from the coarsest scale (smallest image) and going down the pyramid, updating your estimate as you go. It is very easy to implement by adding recursive calls to your original single-scale implementation.

My recursive part of the image pyramid consists of two major parts: when level reaches zero, it will call the original simple alignment function, otherwise, it will resize the images to half of its width and height and recursively call itself. In each call we can get a coarse displacement, accompanied by the fined displacement result derived at its level, we will get a correct result at the final output. Using the image pyramid, I am able to get nice results of icon and harvesters.

**You can click on the image to see a bigger picture for hi-res results.**

*For green channel: move 17 on x-axis, move 40 on y-axis.*

*For red channel: move 23 on x-axis, move 89 on y-axis.*
| ![3cropped_color_image_harvesters.jpg](https://s2.loli.net/2024/09/04/UcoevqmXN1Ay3Tk.jpg) | ![3aligned_color_image_pyramid_harvesters.jpg](https://s2.loli.net/2024/09/04/LsNMZjSBHqYfCiF.jpg) |
|------------------------|------------------------|

*For green channel: move 15 on x-axis, move 58 on y-axis.*

*For red channel: move 13 on x-axis, move 120 on y-axis.*

| ![3cropped_color_image_harvesters.jpg](https://s2.loli.net/2024/09/04/bfrcSv8QowgZaF6.jpg) | ![3aligned_color_image_pyramid_harvesters.jpg](https://s2.loli.net/2024/09/04/WcBdltGabp3EP8w.jpg) |
|------------------------|------------------------|

I thought the image pyramid alignment performs not that bad until I get the result of Emir:

*For green channel: move 24 on x-axis, move 49 on y-axis.*

*For red channel: move 32 on x-axis, move 0 on y-axis.*
| ![34cropped_color_image_emir.jpg](https://s2.loli.net/2024/09/04/KzjCstLyPvE43JS.jpg) | ![34aligned_color_image_pyramid_emir.jpg](https://s2.loli.net/2024/09/04/Ej1PRmh3LIYFnWM.jpg) |
|------------------------|------------------------|

After checking the original channel-splitted picture of emir, I discovered that the exposure and the brightness level of three channels vary dramatically, which can lead to a bad result demonstrated above.

How to solve it?

## Part4: Bells & Whistles: a try on canny
After examining the emir's picture, I discovered that Emir's burka bears lots of stripes and treads on it. And the stripes are so clear and sharp that under uneven exposure and brightness level they can be distinguished easily. So edge detection techique may achieve unbelievably good results.

After channel-split, I ran canny edge detection on three pictures and get the following result:

| ![emir canny b](https://s2.loli.net/2024/09/04/76JB1yWpO9vMxgN.jpg) | ![emir canny g](https://s2.loli.net/2024/09/04/aepMOS3NwnoFjzd.jpg) | ![emir canny r](https://s2.loli.net/2024/09/04/lWck5DOEBaVdjs6.jpg) |
|------------------------|------------------------|------------------------|
| Emir canny blue channel | Emir canny green channel | Emir canny red channel |

By feeding canny-processed pictures into the previous image pyramid algorithm, Emir's color image is sharp and the skin tone is also right. The left is Alignment result only using image pyramid, the right is Alignment result using canny.

| ![34aligned_color_image_pyramid_emir.jpg](https://s2.loli.net/2024/09/04/Ej1PRmh3LIYFnWM.jpg) | ![canny_aligned_color_image_emir.jpg](https://s2.loli.net/2024/09/04/t1dcLirhVN4opG2.jpg) |
|------------------------|------------------------|

<p style="text-align: center;"><em>Picture Selected: emir</em></p>

Best matching displacement for emir.tif (Using blue channel as base image):
For green channel: move 24 on x-axis, move 49 on y-axis.
For red channel: move 40 on x-axis, move 107 on y-axis.

##### More results using canny-processed image pyramid:

| ![canny_image_lady_b_edges.jpg](https://s2.loli.net/2024/09/04/zjBVpUH268Ekq3o.jpg) | ![canny_image_lady_g_edges.jpg](https://s2.loli.net/2024/09/04/KxdDG762WUE1AkP.jpg) | ![canny_image_lady_r_edges.jpg](https://s2.loli.net/2024/09/04/USnFCfIWucBoTmE.jpg) |
|------------------------|------------------------|------------------------|
| lady canny blue channel | lady canny green channel | lady canny red channel |

| ![cropped_color_image_lady.jpg](https://s2.loli.net/2024/09/04/Fbqp7j3ovhcnUwP.jpg) | ![canny_aligned_color_image_lady.jpg](https://s2.loli.net/2024/09/04/lXQ9H3KWprwZRO6.jpg) |
|------------------------|------------------------|

<p style="text-align: center;"><em>Picture Selected: lady</em></p>

Best matching displacement for lady.tif (Using blue channel as base image):
For green channel: move 10 on x-axis, move 56 on y-axis.
For red channel: move 13 on x-axis, move 120 on y-axis.


| ![canny_image_onion_church_b_edges.jpg](https://s2.loli.net/2024/09/04/MRukiF4olt2BDKQ.jpg) | ![canny_image_onion_church_g_edges.jpg](https://s2.loli.net/2024/09/04/syezjQ1pwISg3Z7.jpg) | ![canny_image_onion_church_r_edges.jpg](https://s2.loli.net/2024/09/04/zKM4GXlkSqsZyPN.jpg) |
|------------------------|------------------------|------------------------|
| onion church canny blue channel | onion church canny green channel | onion church canny red channel |

| ![cropped_color_image_onion_church.jpg](https://s2.loli.net/2024/09/04/JnMG4Roc7zguiUX.jpg) | ![canny_aligned_color_image_onion_church.jpg](https://s2.loli.net/2024/09/04/Urdn6zPO5EeIi2R.jpg) |
|------------------------|------------------------|

<p style="text-align: center;"><em>Picture Selected: onion church</em></p>

Best matching displacement for onion_church.tif (Using blue channel as base image):
For green channel: move 24 on x-axis, move 52 on y-axis.
For red channel: move 35 on x-axis, move 107 on y-axis.

| ![canny_image_melons_b_edges.jpg](https://s2.loli.net/2024/09/04/e2uJh97zpDCgKZq.jpg) | ![canny_image_melons_g_edges.jpg](https://s2.loli.net/2024/09/04/Jziqr4STZwxteKd.jpg) | ![canny_image_melons_r_edges.jpg](https://s2.loli.net/2024/09/04/D2nIwH5kCLlmWxs.jpg) |
| --------- | ---------- | ---------- |
| melons canny blue channel | melons canny green channel | melons canny red channel |

| ![cropped_color_image_melons.jpg](https://s2.loli.net/2024/09/04/MNBt5mgJLjZs238.jpg) | ![canny_aligned_color_image_melons.jpg](https://s2.loli.net/2024/09/04/8uahEeo2DB3KxJG.jpg) |
| -------- | -------- |

<p style="text-align: center;"><em>Picture Selected: melons</em></p>

Best matching displacement for melons.tif (Using blue channel as base image):
For green channel: move 10 on x-axis, move 80 on y-axis.
For red channel: move 14 on x-axis, move 176 on y-axis.

| ![canny_image_sculpture_b_edges.jpg](https://s2.loli.net/2024/09/04/h8pmArenc6KsaXu.jpg) | ![canny_image_sculpture_g_edges.jpg](https://s2.loli.net/2024/09/04/tNgc2XWILRUJBQo.jpg) | ![canny_image_sculpture_r_edges.jpg](https://s2.loli.net/2024/09/04/yDm51TLhgfzuNji.jpg) |
| ---- | ---- | ---- |
| sculpture canny blue channel | sculpture canny green channel | sculpture canny red channel |

| ![cropped_color_image_sculpture.jpg](https://s2.loli.net/2024/09/04/XoflpA3e5nxk8VH.jpg) | ![canny_aligned_color_image_sculpture.jpg](https://s2.loli.net/2024/09/04/cmjna6tr958sGyD.jpg) |
| ---- | ---- |

<p style="text-align: center;"><em>Picture Selected: sculpture</em></p>

Best matching displacement for sculpture.tif (Using blue channel as base image):
For green channel: move -11 on x-axis, move 33 on y-axis.
For red channel: move -27 on x-axis, move 140 on y-axis.

| ![canny_image_three_generations_b_edges.jpg](https://s2.loli.net/2024/09/04/lw4uhdBqOsUaCio.jpg) | ![canny_image_three_generations_g_edges.jpg](https://s2.loli.net/2024/09/04/emArC6vjUM5sbK1.jpg) | ![canny_image_three_generations_r_edges.jpg](https://s2.loli.net/2024/09/04/xcyl3Ieonf7uFLP.jpg) |
| ---- | ---- | ---- |
| three generations canny blue channel | three generations canny green channel | three generations canny red channel |

| ![cropped_color_image_three_generations.jpg](https://s2.loli.net/2024/09/04/2lDi1Y9WSwhTBu6.jpg) | ![canny_aligned_color_image_three_generations.jpg](https://s2.loli.net/2024/09/04/lpZMXiWe3koVhBc.jpg) |
| ---- | ---- |

<p style="text-align: center;"><em>Picture Selected: three generations</em></p>

Best matching displacement for three_generations.tif (Using blue channel as base image):
For green channel: move 12 on x-axis, move 55 on y-axis.
For red channel: move 8 on x-axis, move 111 on y-axis.

| ![canny_image_self_portrait_b_edges.jpg](https://s2.loli.net/2024/09/04/vs97olRMkLpwUPK.jpg) | ![canny_image_self_portrait_g_edges.jpg](https://s2.loli.net/2024/09/04/hkmz8fLysEYIRMb.jpg) | ![canny_image_self_portrait_r_edges.jpg](https://s2.loli.net/2024/09/04/Hbpv4fLQzl298md.jpg) |
| ---- | ---- | ---- |
| self portrait canny blue channel | self portrait canny green channel | self portrait canny red channel |

| ![cropped_color_image_self_portrait.jpg](https://s2.loli.net/2024/09/04/qyeD3P9a74irQgK.jpg) | ![canny_aligned_color_image_self_portrait.jpg](https://s2.loli.net/2024/09/04/kVlKI56qOjgyoFY.jpg) |
| ---- | ---- |

<p style="text-align: center;"><em>Picture Selected: self portrait</em></p>

Best matching displacement for self_portrait.tif (Using blue channel as base image):
For green channel: move 29 on x-axis, move 77 on y-axis.
For red channel: move 37 on x-axis, move 175 on y-axis.

| ![canny_image_train_b_edges.jpg](https://s2.loli.net/2024/09/04/AIaptbDWicO4TUr.jpg) | ![canny_image_train_g_edges.jpg](https://s2.loli.net/2024/09/04/325RqSIB97CrJte.jpg) | ![canny_image_train_r_edges.jpg](https://s2.loli.net/2024/09/04/17FojSVlYbdmGyD.jpg) |
| ---- | ---- | ---- |
| train canny blue channel | train canny green channel | train canny red channel |


| ![cropped_color_image_train.jpg](https://s2.loli.net/2024/09/04/3BrUogubDRKxIf4.jpg) | ![canny_aligned_color_image_train.jpg](https://s2.loli.net/2024/09/04/NzWx1HmY4oPjJ9h.jpg) |
| ---- | ---- |

<p style="text-align: center;"><em>Picture Selected: train</em></p>

Best matching displacement for train.tif (Using blue channel as base image):
For green channel: move 0 on x-axis, move 41 on y-axis.
For red channel: move 29 on x-axis, move 85 on y-axis.

##### Two result from Prokudin-Gorskii collection:

<table>
  <tr>
    <td align="center">
      <img src="https://s2.loli.net/2024/09/10/6y9vmReCZTaKJE3.jpg" alt="Stacked w/o simple alignment" width="400"><br>
      <em>Stacked w/o simple alignment</em>
    </td>
    <td align="center">
      <img src="https://s2.loli.net/2024/09/10/9wq5mNuAEvxgbc3.jpg" alt="Stacked w/ simple alignment" width="400"><br>
      <em>Stacked w/ simple alignment</em>
    </td>
  </tr>
</table>

Best matching displacement for more.jpg (Using blue channel as base image):
For green channel: move 2 on x-axis, move 4 on y-axis.
For red channel: move 4 on x-axis, move 7 on y-axis.

| ![canny_image_more2_b_edges.jpg](https://s2.loli.net/2024/09/10/qEjYsOtgfNWPJUx.jpg) | ![canny_image_more2_g_edges.jpg](https://s2.loli.net/2024/09/10/tWP1VDoXB52SNOl.jpg) | ![canny_image_more2_r_edges.jpg](https://s2.loli.net/2024/09/10/5WDsxVnoqHAc12j.jpg) |
| ---- | ---- | ---- |
| boy canny blue channel | boy canny green channel | boy canny red channel |


| ![cropped_color_image_more2.jpg](https://s2.loli.net/2024/09/10/lk7wC8hDJLcitO5.jpg) | ![canny_aligned_color_image_more2.jpg](https://s2.loli.net/2024/09/10/N2IoiV5e8wLYyca.jpg) |
| ---- | ---- |

<p style="text-align: center;"><em>Picture Selected: boy</em></p>

Best matching displacement for boy.tif (Using blue channel as base image):
For green channel: move -14 on x-axis, move 45 on y-axis.
For red channel: move -11 on x-axis, move 103 on y-axis.

## Part5: My debugging journey
Although it's nothing important or difficult, I got a bad result on image pyramid implementation due to the false understanding of `np.roll` at first. It was not until I viewed the previous post when I realized that it wasn't the pictures that are hard to align, it was my function wrong. My Bells&Whistles is originally an attempt to rotate the picture to achieve alignment. But obviously, if the base method is wrong, there's no way for this attempt to be right.

The way `np.roll` handles image translation is to move the section out of the border to the gap created at the other side. So if the original image has black edges on both left and the right side, rolling left means the black edge on the left transferred to the right of the original right black edge, which doesn't contribute to reducing the L2 norm and therefore pictures won't match in the end. In short, counteract the effect of `np.roll` in the recursion bit of the function, which will help reducing the L2 norm dramatically, and keep the effect of `np.roll` at the last level of the pixel calculation.