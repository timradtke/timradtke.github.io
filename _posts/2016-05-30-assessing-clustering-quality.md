---
layout: post
title:  "Assessing the Quality of a Clustering Solution"
tags: statistics clustering reproducible
---

![not the clusters you think of]({{ site.url }}/img/cornflakes-clusters.png)

During one of the talks at PyData Berlin, a presenter quickly mentioned a k-means clustering used to group similar clothing brands. She commented that it wasn’t perfect, but good enough and the result you would expect from a k-means clustering.

There remains the question, however, how one can assess whether a clustering is “good enough”. In above case, the number of brands is rather small, and simply by looking at the groups one is able to assess whether the combination of Tommy Hilfiger and Marc O’Polo is sensible. This becomes more difficult, though, if you cluster hundreds or thousands of observations. Additionally, consider several clustering methods besides k-means from which to choose. In supervised learning, you always have labels to tell you the truth (for your training data), but in clustering, there is no objectively correct solution. How can we assess a cluster’s quality?

In his [notebooks](http://bactra.org/notebooks/clustering.html), Cosma Shalizi shares the frustration with assessing clustering quality:
>This [clustering] is an important subject, but one of the topics I most dislike teaching in data mining, because the students' natural question is always "how do I know when my clustering algorithm is giving me a good solution?", and it's very hard to give them a reasonable answer. I think this is because most other data-mining problems are basically predictive, and so one can ask how good the prediction is; what's the best way to turn clustering into a prediction problem? 

While Christian Hennig doesn't answer how clustering can be turned into a prediction problem, he did propose methods for assessing the clustering quality in a recent talk at PyData London. His presentation is [available on Youtube](https://www.youtube.com/watch?v=Mf6MqIS2ql4).

For me, the most important takeaway from Hennig’s presentation is the fact that due to the lack of a “true” clustering, the researcher has to choose from different clustering goals for which to optimize. For example, he might optimize for between-cluster separation, within-cluster homogeneity, or uniform cluster sizes. Since some of the goals may be in conflict with each other, the goals have to be scored by importance, and the different measures of quality have to be weighted. As usual, it is important to know the question you would like to answer; only then you can argue whether your clustering delivers a valuable solution.

In his presentation, Hennig refers to [a paper he co-authored](http://onlinelibrary.wiley.com/doi/10.1111/j.1467-9876.2012.01066.x/abstract) in which he describes diligently how he chooses between two different clustering methods in order to cluster individuals in a social stratification data set, and how he is able to compare the resulting clusters’ validity. The paper includes a chapter on the clustering philosophy he and Tim Liao applied in this specific case. They write:

>In most applications including social stratification the researcher may not be able to justify all the required decisions in detail; sometimes different aspects of what the researcher is interested in are in conflict, and there is feedback between the desired cluster concept and the data (the researcher may want very homogeneous but not too small clusters, or may have **unrealistic expectations about existing separation between clusters**). Furthermore, it is hardly possible to anticipate the implications of all required tuning decisions for the given data. As mentioned before, researchers often want to check whether the found clusters carry more meaning than an optimal partition of homogeneous random data. Therefore it is not enough to specify a cluster concept and to select a method. Clustering results need to be validated further.

The highlighted sentence expresses the frustration that clustering will entail if you start out with unrealistic expextations (as is so often the case). For example, if you set out to perform a clustering on a given data set for your thesis, and your clustering doesn’t bring up important clusters, you want to be smart about the communication of the result. Why didn’t it happen, which goals can still be achieved, which goals were unreasonable given the data. On the other hand, if your straightforward k-means returned interesting groupings of observations, you’d better be sure to communicate results which are valid and generalize. Else, you might find yourself making business decisions based on random allocations of brands.

To that end, I recommend Hennig’s [R package `fpc`](https://cran.r-project.org/web/packages/fpc/fpc.pdf) in which he has prepared several measures to support you with cluster validation.