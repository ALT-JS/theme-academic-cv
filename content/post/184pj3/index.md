---
title: 'COMPSCI 184 Project3: PathTracer'
date: '2025-03-04'
draft: false
math: true
subtitle: 'implementations of the core routines of a physically-based renderer using a pathtracing algorithm'
---

![dragon_64_32.png](https://s2.loli.net/2025/03/18/eHbOGFcswjBzxDa.png)

## Part 0: Project overview
In this project, I implemented some fundamental functions and features of a physically-based renderer using path tracing algorithm. First, I implemented ray generation and the intersection between ray and objects/scene. Next, I implemented the BVH splitting and intersection detection. Then, I implemented direct illumination(zero/one bounce lights) and global illumination(multi-bounce lights w/ w/o russian roullete) to actually render the whole scene with path tracing. Finally, I implemented adaptive sampling to reduce the noise of monte carlo path tracing.

## Part 1: Ray Generation and Scene Intersection

### Ray generation walkthrough & primitive intersection pipeline
I first calculate the bottom left and the top right corner of the sensor/image space coordinates, which helps me to determine the coordinates in the camera space with left multiplcation of the `c2w` matrix. Then with the origin `pos` and the previously calculated coord as `direction` vector(since the origin is [0,0,0] and it's numerically the same for the coords and the vector), we can generate the correct ray.

Primitive is the bridge between geometry processing and the shading subsystem. It contains all the things including `bbox`, `intersect`, `has_intersection` and `bsdf`. For intersections, we use `has_intersection` to determine whether it's intersected or not and use `intersect` to actually change the intersection distance, the normal vector, and the bsdf.

### Triangle intersection algorithm walkthrough
For the `intersect` and `has_intersection`, I used the Moller-Trumbore algorithm to compute them. We compute two edges of the triangle. Compute the determinant `det` to check if the ray is nearly parallel. Compute u and v(barycentric coordinates) to ensure the intersection lies within the triangle. Compute t, the intersection distance along the ray. And finally check if t is within the ray's valid range (min_t to max_t). The function `intersect` basically determine whether `has_intersection` is true or not and change the primitives and the properties mentioned above(intersection distance, normal vector, bsdf).

### Some results
| ![1-CBspheres.png](https://s2.loli.net/2025/03/17/PSxngs9r1DGBQvh.png) | ![1-cow.png](https://s2.loli.net/2025/03/17/ahoUQ67VMJ8Hnm5.png) |
|:--------------------:|:--------------------:|
| CB spheres | cow |

## Part 2: Bounding Volume Hierarchy

### BVH construction algorithm walkthrough
First, compute the bounding box (BBox) of all primitives in the given range [start, end). Then, I choose the best axis to split: compute the bounding box centroid for each primitive. Select the axis with the largest extent (X, Y, or Z) and split at the centroid median to achieve balanced partitions. The primitives are sorted first and partitioned into left and right subsets. Finally, recursively call function and construct BVH for left and right subsets and return the newly created BVHNode.

### Some results
| ![2-cblucy.png](https://s2.loli.net/2025/03/17/hsbaMc4D8Z1iKzq.png) | ![2-max.png](https://s2.loli.net/2025/03/17/h2aK4RpOl9yntiw.png) |
|:--------------------:|:--------------------:|
| CB Lucy | Max Planck |

### More on BVH acceleration
First, here's more result on the BVH rendering(left) and the comparison between the non-BVH. You can see that there's basically no difference between them.
| ![2-peter.png](https://s2.loli.net/2025/03/17/L7pVYQdUeayEkjW.png) | ![2slow-peter.png](https://s2.loli.net/2025/03/17/sKw7S8VnQ9bU3k2.png) |
|:--------------------:|:--------------------:|
| Peter w/ BVH | Peter w/o BVH |

But the rendering time, as shown in the picture below, have significant difference. One is 0.0653 second, and the other takes 166.5520 seconds to finish. Nearly 300x faster.

![2-peterspeed.png](https://s2.loli.net/2025/03/17/mGnP5FgaLyU4qWM.png)

## Part 3: Direct Illumination

### Direct Lighting functions walkthrough

#### Uniform Hemisphere Sampling
In the loop of `num_sample` samples, we sample from `hemisphereSampler`, then we create a new ray with in point `hit_p` and the epsilon constant for avoiding numerical presision issues and direction sampled earlier. If the ray intersects we calculate the `L_out` of the ray. Finally we divide the sum of all the things above with pdf and the `num_samples`.

#### Importance Sampling Lights
FOr importance sampling, we sample all the lights directly After we calculate the hit point `hit_p`, we directly sample a light ray from it. If we cast a ray in this direction and there is no other object between the hit point and the light source, then we know that this light source does cast light onto the hit point. After the judge, we still do the same thing as above(divide the sum of all the things above with pdf and the `num_samples`).

### Some results
| ![3H-bunny_64_32.png](https://s2.loli.net/2025/03/17/dcW1BYZT6Jpvswl.png) | ![3-bunny_64_32.png](https://s2.loli.net/2025/03/17/uhZSVJynjMpz6UK.png) |
|:--------------------:|:--------------------:|
| Hemisphere uniform sampling | Direct importance sampling |

| ![3H-dragon_32_16.png](https://s2.loli.net/2025/03/17/sbQr1iKAhxGZSg7.png) | ![3-dragon_32_16.png](https://s2.loli.net/2025/03/17/olEq7FRBQMzStYb.png) |
|:--------------------:|:--------------------:|
| Hemisphere uniform sampling | Direct importance sampling |

### Direct comparison
| ![3-1-bunny_64_32.png](https://s2.loli.net/2025/03/17/pd3KXycOaBsR8Eo.png) | ![3-4-bunny_64_32.png](https://s2.loli.net/2025/03/17/ORMdsz56EyBwQFS.png) |
|:--------------------:|:--------------------:|
| ![3-16-bunny_64_32.png](https://s2.loli.net/2025/03/17/wuqkp8aj7XcZ3PR.png) | ![3-64-bunny_64_32.png](https://s2.loli.net/2025/03/17/5exniCTFvOcGjm9.png) |
| Up 1 Down 16 | Up 4 Down 64 |

### Conclusion
Uniform hemisphere sampling distributes samples evenly over the hemisphere, leading to significant variance in Monte Carlo integration when evaluating lighting contributions, especially in scenes with strong directional lighting. This results in more noise and slower convergence. In contrast, importance sampling strategically places more samples in directions where the light source contributes most to the radiance, reducing variance and improving efficiency. This method converges faster, producing smoother and more accurate results with fewer samples. 

## Part 4: Global Illumination

### Indirect lighting function walkthough
We get the one bounce radiance first, then, we sample a new f using the outgoing direction of the last ray. We create new ray and increase its depth by 1. When the depth reaches the max depth, just return the one bounce result earlier(this is the last one being calculated), otherwise recursively calling the function.

As for russian roulette, the difference is that every new ray generated each turn has to pass the russian roulette coin flip check in order to not be eliminated and thrown away. The probability I'm using here is 70% continue, 30% delete each turn.

### Some global illumination results
| ![4-bunny_1024_8_2.png](https://s2.loli.net/2025/03/17/D4Ye7gfVZy5TcUq.png) | ![4-spheres_1024_8_4.png](https://s2.loli.net/2025/03/17/bfBkY6LQGyCnXAZ.png) |
|:--------------------:|:--------------------:|
| Bunny (maxdepth=2) | Spheres (maxdepth=4) |

### Only direct illumination vs. only indirect illumination
| ![4-spheres_1024_8_1.png](https://s2.loli.net/2025/03/17/QzaAcpTOljU9CMK.png) | ![4-spheres_1024_8_2onlyi.png](https://s2.loli.net/2025/03/18/4fCLUGcaYm8dwqI.png) |
|:--------------------:|:--------------------:|
| only direct illumination | only indirect illumination (maxdepth=2) |

### Renders of `max_ray_depth` from 0 to 5
specs using: `-t 12 -s 1024 -l 8 -r 480 360`, click the image to zoom in on the website.

| isAccumBounces | m = 0 | m = 1 | m = 2 | m = 3 | m = 4 | m = 5 |
| -------------- | ----- | ----- | ----- | ----- | ----- | ----- |
| false | ![CB_Bunny_0.png](https://s2.loli.net/2025/03/18/cGERHpAguQeCFLI.png) | ![CB_Bunny_1.png](https://s2.loli.net/2025/03/18/IYqgA2rjBJsduiV.png) | ![CB_Bunny_2.png](https://s2.loli.net/2025/03/18/OnDZbmFq9UihCgL.png) | ![CB_Bunny_3.png](https://s2.loli.net/2025/03/18/sqfh6ZUnmKetLET.png) | ![CB_Bunny_4.png](https://s2.loli.net/2025/03/18/r1iAbHDxV48nmp3.png) | ![CB_Bunny_5.png](https://s2.loli.net/2025/03/18/8X7aYmyHVGTpEQs.png) |
| true | ![CB_Bunny_cumsum_0.png](https://s2.loli.net/2025/03/18/bvizX7BDAYJN89T.png) | ![CB_Bunny_cumsum_1.png](https://s2.loli.net/2025/03/18/VjzShtWMLb7gIQy.png) | ![CB_Bunny_cumsum_2.png](https://s2.loli.net/2025/03/18/3WxsAPvbGEq6HNL.png) | ![CB_Bunny_cumsum_3.png](https://s2.loli.net/2025/03/18/LeQf9sqC3vta2hG.png) | ![CB_Bunny_cumsum_4.png](https://s2.loli.net/2025/03/18/epknVAIzZwdN2Ly.png) | ![CB_Bunny_cumsum_5.png](https://s2.loli.net/2025/03/18/ts5jVlDz1NhGSZF.png) |

#### the 2nd and 3rd bounce of light
For the 2nd bounce of the light, its main contribution is that it illuminates the ceiling, making the overall rasterization way more realistic. For the third bounce of light, it slightly illuminates the ground around the bunny, mimicing the lights reflected away from the bunny surface, making the overall rasterization more realistic.

### Russian roulette rendering results
specs using: `-s 1024 -l 4`. Along with using russian soulette, you may not be able to see that much difference after 2 bounce.
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Flexbox</title>
    <style>
        .container1 {
            display: flex;
            justify-content: space-between; /* 图片之间有间距 */
            align-items: center;
            gap: 10px; /* 图片间距 */
        }
        .container1 img {
            width: 100px; /* 设定图片宽度 */
            height: auto; /* 保持宽高比 */
        }
    </style>
</head>
<body>
    <div class="container1">
        <img src="https://s2.loli.net/2025/03/18/ter3DLVmPMNxOHU.png" alt="图片1">
        <img src="https://s2.loli.net/2025/03/18/4bXiQIOSD6k2h9G.png" alt="图片2">
        <img src="https://s2.loli.net/2025/03/18/f2LITNAPtaVv6QO.png" alt="图片3">
        <img src="https://s2.loli.net/2025/03/18/y4ZuJkjoRhrLM5e.png" alt="图片4">
        <img src="https://s2.loli.net/2025/03/18/8PTfcBJsKSzVAYa.png" alt="图片5">
        <img src="https://s2.loli.net/2025/03/18/ys6JOa3PBA4E7Zo.png" alt="图片5">
    </div>
</body>
</html>

### Various sample-per-pixel rates results
From left to right, from top to bottom: 1 2 4 8 16 64 1024.
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Flexbox</title>
    <style>
        .container2 {
            display: flex;
            justify-content: space-between; /* 图片之间有间距 */
            align-items: center;
            gap: 10px; /* 图片间距 */
        }
        .container2 img {
            width: 150px; /* 设定图片宽度 */
            height: auto; /* 保持宽高比 */
        }
    </style>
</head>
<body>
    <div class="container2">
        <img src="https://s2.loli.net/2025/03/18/fHB89v74NbV1RAe.png" alt="图片1">
        <img src="https://s2.loli.net/2025/03/18/a7iHhyqIOwczQ3C.png" alt="图片2">
        <img src="https://s2.loli.net/2025/03/18/IZNlfs51XEhi8Pe.png" alt="图片3">
        <img src="https://s2.loli.net/2025/03/18/DGeFtJuWlV6I9Sd.png" alt="图片4">
    </div>
</body>

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Flexbox</title>
    <style>
        .container3 {
            display: flex;
            justify-content: space-between; /* 图片之间有间距 */
            align-items: center;
            gap: 10px; /* 图片间距 */
        }
        .container3 img {
            width: 200px; /* 设定图片宽度 */
            height: auto; /* 保持宽高比 */
        }
    </style>
</head>
<body>
    <div class="container3">
        <img src="https://s2.loli.net/2025/03/18/tdysICTwuG6kzU1.png" alt="图片1">
        <img src="https://s2.loli.net/2025/03/18/gn7KfalOB6sDwiJ.png" alt="图片2">
        <img src="https://s2.loli.net/2025/03/18/nhL8oIT5qeFlKXN.png" alt="图片3">
    </div>
</body>

## Part 5: Adaptive Sampling

### Adaptive sampling walkthrough
Adaptive sampling is a method used in ray tracing to reduce the number of samples per pixel while maintaining image quality. Instead of taking a fixed number of samples (ns_aa), it dynamically decides when to stop sampling based on statistical analysis.

(If only one sample is requested, the algorithm traces a single ray and directly stores the computed radiance.) We loop over `ns_aa` samples, and only check after every `samplesPerBatch` samples. Then we calculate and check confidence interval `I` is smaller or larger than a threshold, if smaller then it stops early. otherwise just like ordinary calculation. Once sampling is complete (either because the pixel converged or the maximum samples were taken), the final radiance is computed as the average of all collected samples.

### Some results
| ![5-bunny.png](https://s2.loli.net/2025/03/18/SIEatGOiRCykVZY.png) | ![5-bunnyrate.png](https://s2.loli.net/2025/03/18/Jy58qPUbscnmpvN.png) |
|:--------------------:|:--------------------:|
| bunny | bunnyrate |

| ![5-ball.png](https://s2.loli.net/2025/03/18/hwvjBs8Wyx4Dk2G.png) | ![5-ballrate.png](https://s2.loli.net/2025/03/18/3SGViozB81c6rlH.png) |
|:--------------------:|:--------------------:|
| ball | ballrate |