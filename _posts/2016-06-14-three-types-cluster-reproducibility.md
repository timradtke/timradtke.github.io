---
layout: post
title:  "Three Types of Cluster Reproducibility"
tags: statistics clustering reproducible
---

Christian Hennig provides a function called `clusterboot()` in his R package `fpc` which [I mentioned before](http://www.timradtke.net/blog/assessing-clustering-quality-hennig) when talking about assessing the quality of a clustering. The function runs the same cluster algorithm on several bootstrapped samples of the data to make sure that clusters are reproduced in different samples; it validates the cluster stability.

In a similar vein, the reproducibility of clusterings with subsequent use for marketing segmentation is discussed in [this paper](https://epub.ub.uni-muenchen.de/10960/1/tr063.pdf) by Dolnicar and Leisch. They describe three types of cluster solutions distinguished by their level of reproducibility. The authors argue that this kind of assessment is necessary so as to communicate the correct value of the results to the management. The marketing team is then able to work with the results to create an appropriate customer segmentation.

## Natural Clustering

The first and optimal type of clustering is what the authors call *natural clustering/segmentation*. There exist (obvious) groups (“density clusters”) that are easily picked up by algorithms and persist over different samples. Basically, there exists some natural cause segmenting the observations. Imagine a 2D-plot where two very clearly different groups of observations are shown.

## Reproducible Clustering

The second type of clustering they call *reproducible clustering/segmentation*:
>Typically, consumer data contain some structure, but not density clusters. … In such situations true density clusters cannot be revealed but data structure can be used to derive stable, reproducible market segments.

I'd describe the natural clustering as an idealized type of solution that usually doesn't occur, and so a strongly reproducible clustering is the outcome one is looking for. There does not necessarily exist a natural distinction of segments, but the data may still be very structured. Alas, this is not always the case and leads to the third type.

## Constructive Clustering

*Constructive clustering/segmentation* is what the authors call cluster solutions which are not reproducible due to unstructured data, but which may inform market segmentation in a valuable manner anyways. The authors argue that an artificial segmentation can still be better than no segmentation, and so “while the cluster algorithm cannot reveal true groups of consumers, it can still help to create managerially useful subgroups of customers”.

Working with this scaffolding, Dolnicar and Leisch go on to show how two questions about clusters can be examined. First, “Does the data set contain natural clusters?” and second, “Does the data set allow reproduction of similar segmentation solutions?”. To perform the explanatory analysis on articificial and real data sets, they have used the packages `flexmix` and [`flexclust`](https://cran.r-project.org/web/packages/flexclust/flexclust.pdf) by Leisch himself. The function `bootFlexclust()` they use is similar to `clusterboot()` in that it implements a repeated clustering on bootstrapped data. 

With regard to the bootstrapping for stability assessment, the authors seem to overlap with Christian Hennig's recommendations. But I draw value from the way in which they interpret cluster solutions and prepare them for management (in their theoretical examples). This is something I don’t come across in my courses but as time goes on it will become increasingly important for me. Take for example this paragraph (which is sadly based on their aritificial data; also, they never explicitly go through a thought experiment in which they present a constructive clustering/segmentation, the weakest type, to management):

>To continue the marketing management interpretation of the artificial example, we would suggest the following consequences for the two data sets: the square data would be classified as stable segmentation case with four market segments. In addition to the two segments mentioned above, this data set would include also highly price-elastic techno buffs and – every manager’s dream segment – the modest price-inelastic mobile phone users in the top right-hand corner who seemingly do not mind to pay a lot of money for a basic mobile phone with few features only. In the case of the circular data set, management has to be informed that natural segments do not exist and that the data set does not enable the researcher to determine stable segments of consumers. In this constructive segmentation case a number of solutions have to be computed, visualized, described and presented to management for comparative evaluation.