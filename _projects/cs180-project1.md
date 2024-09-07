---
layout: page
title: "Project 1: Images of the Russian Empire"
description: Colorizing the Prokudin-Gorskii photo collection
img: assets/img/180-project1/results/emir.jpg
importance: 1
category: cs180
related_publications: false
---

## Inroduction
In this project, I implemented a simple script that attempts to colorize the pictures taken by Prokudin-Gorskii as early as 1907, prior to the invention of color photography. These pictures are taken three in a group, each recorded with either a red, green or blue filter. Our goal is to recombine these pictures into one and produce a vivid color picture.

## Single Scale Implementation
Firstly, we need to find a plausible way of overlaying these pictures. Since these pictures are not taken digitally, they are essentially three pictures stick together. Our first step is to chuck them into three channels, R, G, and B. This can be easily done by dividing the picture evenly in vertical direction:
<div class="row">
    <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/data/cathedral.jpg" title="Original cathedral" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/data/monastery.jpg" title="Original monastery" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/data/tobolsk.jpg" title="Original tobolsk" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
After getting the three images, we convert them into `float` so that it is easier to process. In my algorithm, there is one parameter called `crop-r` that is core of my design. `crop-r` is the crop ratio that will be applied to the R channel, which takes the center `crop-r` portion of the image. The two other channels are aligned to the R channel.

To align the two other channels, the signle-scale implementation applies a sliding window of the same size as the cropped R channel images to the G and B channels. The difference in size decides the size of the search gird. More formally, it will become a `(W * (1 - crop-r), H * (1 - crop-r))` grid. The crop ratio used here is `0.85` to avoid boundaries. A larger value such as `0.9` also works but would include considerably more boundary portion and damage the aesthetics of the final product. Once the grid is created, the similarity between the sliding window and the R channel is calculated as (Here, $$\mathrm{img}_1$$ refers to the sliding window and the $$\mathrm{img}_2$$ refers to the R channel):

$$
\sum_{i, j} \frac{ {\mathrm{img}_{1}}_{i, j} - \mu_1}{\sigma_1} \times {\mathrm{img}_{2}}_{i, j}
$$

In other words, we calculate the dot product between the **normalized** sliding window and the R channel. The idea is that we do not care about the absolute scale of different channels. The only requirement we have here is that the relatively bright/dark areas in two images must match. In fact, I tried different metrics such as cosine similarities and Frobenius distances. They did not produce as high quality image as the current version. This is because there are images that have very vibrant colors, which leads to a significant difference between channels. As a side comment, the reason why $$\mathrm{img}_2$$ does not have to be normalized is because it is a constant and normalization is a global affine mapping to the similarities and won't change the maximum.

After that, the channels are overlaid to produce the final colored pictures:
<div class="row">
    <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/results/cathedral.jpg" title="Colorized cathedral" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/results/monastery.jpg" title="Colorized monastery" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/results/tobolsk.jpg" title="Colorized tobolsk" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

The offsets presented are relative to the R channel.

| &nbsp; &nbsp; &nbsp; channel &nbsp; &nbsp; &nbsp; | &nbsp; &nbsp; cathedral &nbsp; &nbsp; | &nbsp; &nbsp; monastery &nbsp; &nbsp; | &nbsp; &nbsp; &nbsp; tobolsk  &nbsp; &nbsp; &nbsp;  |
| :----------: | :-------: | :-------: | :-----: | 
| G offset | (7, 1) | (6, 1) | (4, 1) | 
| B offset | (12, 3) | (3, 2) | (6, 3) | 

## Multi-scale Implementation
However, as you may have noticed, this method scales as $$O(W^2H^2)$$ if crop ratio is constant: one $$HW$$ from the grid size, and another $$HW$$ from the similarity score calculation. This is prohibitively slow. To address this issue, I implemented a multi-scale version of the aligning algorithm. The idea is very simple: we downsample the image, find the optimal offsets, and perform a fine-grained search near the optimal offsets found in downsampled version. 

To be more precise, each time, the image is downsampled by a factor of `scale`, this recursive call ends as the `depth` parameter is reached. We then perform a full grid search downsampled version (the top of the pyramid). The optimal offset is then divided by the scale to go up by one layer, and the grid of the upper layer is no longer a full search but rather a `scale` portion of the full grid centerred around the optimal offset found by the recursive call. This design keeps the grid size constant, which after experiment balance well between quality and speed. The runtime to process these pictures is about 20 seconds.

<div class="row">
    <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/results/church.jpg" title="Colorized church" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/results/emir.jpg" title="Colorized emir" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/results/harvesters.jpg" title="Colorized harvesters" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="row">
    <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/results/icon.jpg" title="Colorized icon" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/results/lady.jpg" title="Colorized lady" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/results/melons.jpg" title="Colorized melons" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="row">
    <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/results/onion_church.jpg" title="Colorized onion_church" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/results/sculpture.jpg" title="Colorized sculpture" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/results/self_portrait.jpg" title="Colorized self_portrait" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="row">
    <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/results/three_generations.jpg" title="Colorized three_generations" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/results/train.jpg" title="Colorized train" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
    </div>
</div>

| &nbsp; &nbsp; &nbsp; channel &nbsp; &nbsp; &nbsp; | &nbsp; &nbsp; emir &nbsp; &nbsp; | &nbsp; &nbsp; church &nbsp; &nbsp; | &nbsp; &nbsp; &nbsp; icon  &nbsp; &nbsp; &nbsp;  |
| :----------: | :-------: | :-------: | :-----: | 
| G offset | (57, 17) | (33, -8) | (48, 5) | 
| B offset | (88, 44) | (58, -4) | (89, 23) | 

## Auto Contrast (Histogram Equalization)
For the Bells & Whistles, I chose to implement the histogram equalization. This method attempts to enhance the contrast by making a histogram of brightness, flattening the histogram to make sure the best spread across all brightness levels. By doing so, details in the picture become more visible.
Original:
<div class="row">
    <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/results/cathedral.jpg" title="Original cathedral" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/results/monastery.jpg" title="Original monastery" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/results/tobolsk.jpg" title="Original tobolsk" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
Contrasted:
<div class="row">
    <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/auto-contranst-results/cathedral.jpg" title="Contrasted cathedral" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/auto-contranst-results/monastery.jpg" title="Contrasted monastery" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/auto-contranst-results/tobolsk.jpg" title="Contrasted tobolsk" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
Original:
<div class="row">
    <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/results/church.jpg" title="Original church" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/results/emir.jpg" title="Original emir" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/results/icon.jpg" title="Original icon" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
Contrasted:
<div class="row">
    <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/auto-contranst-results/church.jpg" title="Contrasted church" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/auto-contranst-results/emir.jpg" title="Contrasted emir" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/auto-contranst-results/icon.jpg" title="Contrasted icon" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

After auto contrast, we can observe a lot more details, especially the sky. The color also becomes slightly more vivid.

## Auto Color Balance
I also performed auto color balance through forcing the brightest pixel of each channel to be 1.

Original:
<div class="row">
    <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/results/cathedral.jpg" title="Original cathedral" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/results/monastery.jpg" title="Original monastery" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/results/tobolsk.jpg" title="Original tobolsk" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
Balanced:
<div class="row">
    <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/auto-color-balance-results/cathedral.jpg" title="Balanced cathedral" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/auto-color-balance-results/monastery.jpg" title="Balanced monastery" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/auto-color-balance-results/tobolsk.jpg" title="Balanced tobolsk" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
Original:
<div class="row">
    <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/results/church.jpg" title="Original church" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/results/emir.jpg" title="Original emir" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/results/icon.jpg" title="Original icon" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
Balanced:
<div class="row">
    <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/auto-color-balance-results/church.jpg" title="Balanced church" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/auto-color-balance-results/emir.jpg" title="Balanced emir" class="img-fluid rounded z-depth-1" %}
    </div>
        <div class="col-sm">
        {% include figure.liquid loading="eager" path="assets/img/180-project1/auto-color-balance-results/icon.jpg" title="Balanced icon" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

Since the original picture is already pretty balanced, this does not make a big difference.

## References
Wikipedia contributors. (2024, August 13). "Histogram equalization". *Wikipedia*. https://en.wikipedia.org/wiki/Histogram_equalization