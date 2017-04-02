---
layout: post
title:  "Setting Hyper-Parameters of a Beta Prior Distribution"
tags: statistics data_science beta_distribution
---

Suppose you're implementing Bayesian A/B testing in your company. To encourage your colleagues to use sensible prior distributions in testing, you would like to make it easy for them to choose the parameters of the Beta prior. Sure, they could play around with the parameters until the curve looks fine; but there is a better way.

When setting the prior, one would like the prior to cover a certain interval with some probability (`0.95`, for example). There should also exist a conversion rate that one expects the most. Then, given the interval and the mean of the beta distribution, it is easy to calculate the parameters `a` and `b` of the Beta distribution which define the prior.

The same problem is an exercise in chapter 3 of Murphy's *[Machine Learning: A Probabilistic Perspective](https://www.cs.ubc.ca/~murphyk/MLbook/)*:

> Suppose `theta ~ Beta(a,b)` and we believe that `E[theta]=m` and `p(l<theta<u) = 0.95`. Write a program that can solve for `a` and `b` in terms of `m`, `l,` and `u`. Hint: write `b` as a function of `a` and `m` so the pdf only has one unknown; then write down the probability mass contained in the interval as an integral, and minimize its squared discrepancy from `0.95`. What values do you get if `m=0.15`, `l=0.05` and `u=0.3`? What is the equivalent sample size of this prior?

The following R code offers one solution to the problem.

```
optimHyperpars <- function(m,l,u, prob = 0.95, maxESS = 300, maxAlpha = 100) {
  maxAlpha <- round(ifelse(m*maxESS < maxAlpha, m*maxESS, maxAlpha))
  
  computeDeviation <- function(a) {
    b <- round(a*(1-m)/m)
    betaIntegral <- pbeta(u, a, b) - pbeta(l, a, b)
    deviation <- (prob - betaIntegral)^2
    return(c(beta = b, deviation = deviation, coverage = betaIntegral))
  }
  
  optimSet <- 1:maxAlpha
  parameterSet <- cbind(alpha = optimSet,
                        plyr::ldply(optimSet, computeDeviation))
  
  optimalParameters <- parameterSet[which.min(parameterSet$deviation),1:2]
  optimalDeviation <- min(parameterSet$deviation)
  uniqueMinimum <- min(
    parameterSet$deviation[-which.min(parameterSet$deviation)]) != 
    optimalDeviation
  
  if (!uniqueMinimum) message("The minimum is not unique.")
  
  return(list(optimalParameters = optimalParameters,
              optimalDeviation = optimalDeviation,
              uniqueMinimum = uniqueMinimum,
              parameterSet = parameterSet))
}
```

The function `optimHyperpars` takes as input the values described in Murphy’s problem, which describe the information one might have prior to an A/B test. Additionally, a maximum value for the equivalent sample size given by `(a+b)` can be specified. The equivalent sample size has priority over the maximum value for `a`, which is `100` by default. 

If the equivalent sample size is specified, then the maximum value for `a` will be given through `m*maxESS`, where `m` is the mean of the distribution we look for.

Notice that we take a brute-force approach over `1:maxAlpha` to find the parameters minimizing the deviation from the given probability coverage. We only take into account integer values to limit the set of potential parameters. But this also fits the idea of an equivalent sample size given by the parameters of the prior distribution, where `a` can be interpreted as conversions and `(a+b)` as the number of observations we had before starting the A/B test.

The best way to make sure that the resulting beta prior makes sense, is to plot the density function with the interval specified by `l=0.05` and `u=0.3`.

![beta density]({{ site.url }}/img/beta-hyperparameter/beta-density.png)

The following code to plot the results was easy because of [David Robinson's example](https://github.com/dgrtwo/dgrtwo.github.com/blob/master/_R/2016-10-12-hierarchical_bayes_baseball.Rmd). He wrote an excellent [series of blog posts](http://varianceexplained.org/r/hierarchical_bayes_baseball/) on Bayesian methods.

```
optimResults <- optimHyperpars(m = 0.15, 0.05, 0.3)

optimResults$optimalParameters %>%
  do(data_frame(x = seq(0, 1, .0002),
                density = dbeta(x, .$alpha, .$beta))) %>%
  ggplot(aes(x, density)) +
  geom_line() +
  geom_ribbon(aes(ymin = density * (x < .05), ymax = density * (x < .3)),
              alpha = .1, fill = "red") +
  geom_vline(color = "red", lty = 2, xintercept = .05) +
  geom_vline(color = "red", lty = 2, xintercept = .3) +
  theme_bw() +
  labs(y = "Prior density",
       x = "theta",
       title = paste("Beta prior with effective sample size of (a+b) =",
                     sum(optimResults[[1]])))
```