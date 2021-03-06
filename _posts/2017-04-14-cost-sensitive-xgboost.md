---
layout: post
title: "Cost Sensitive Learning with XGBoost"
tags: data_science xgboost
---

*We use a custom evaluation metric for our XGBoost model to predict returning customers with asymmetric costs.*

In a course at university, the professor proposed a challenge: Given customer data from an ecommerce company, we were tasked to predict which customers would return for another purchase on their own (and should not be incentivized additionally through a coupon). This binary prediction problem was posed with an asymmetric cost matrix, the argument being that false negatives are more costly than false positives. 

Here, a false negative implies that the company sends a coupon to someone who would have returned anyway. Thus, he would be given a discount for no reason leading to a loss of €10. On the other hand, whenever the company sends a coupon to someone who is not returning on his own, the customer will now return with a probability of 0.2 and place an order of an average value of €25. Thus, the expected value from sending a coupon to a non-returning customer is €3. This gives the profit for true negatives.

The setting described above differs from the traditional case in which both false positives and false negatives lead to a cost of 1, while the cost of correct predictions is zero. In this business context, however, it is important to take the costs of the business into account, so that not some error rate, but the actual profits are optimized. This is what is called cost-sensitive learning.

A good introduction to cost-sensitive learning is given in [Elkan (2001)](http://cseweb.ucsd.edu/~elkan/rescale.pdf). Most supervised algorithms return some kind of posterior probability `p=P(Y=1|X=x)`. In the standard case of symmetric costs, this probability is turned into binary predictions using `Y_hat=1 if p >= 0.5`, that is, if `Y=1` is more likely than `Y=0`, then predict 1. The catch with cost sensitive learning is that the threshold of 0.5 might no longer be optimal. Instead, if false negatives are more costly than false positives, we might want to lower this threshold so that we predict `Y=1` already when `P(Y=1|X=x)=0.25`, for example.

Thus, there are two things we'd like to do with our supervised learning algorithms:

1. Instead of using, for example, the error rate as evaluation metric to compare models, we want to use a metric that takes the asymmetric costs into account
2. Find the optimal threshold that transforms the posterior probabilities into binary predictions given the cost matrix



## Cost Sensitive Evaluation Metric

One of the many advantages of the gradient boosting implementation in XGBoost is the possibility of using a custom evaluation metric during model training. That's helpful because we might want to use the evaluation metric not only *after* training to compare different trained models, but also *while* training for early stopping (so that we don't have to train the full `nrounds` if the performance on the test set does not improve).

Now, given the costs described above, it is straightforward to define a profit function (in contrast to a cost function) that takes the asymmetric costs into account. This is implemented in the function `getProfit()` below. However, given a smaller share of returning customers in the data set (about 20% of the observations), models that are not tuned to the costs would often predict all observations to be non returning in an effort to optimize the misclassification rate (then being around 0.2). 

In our case, the result of predicting non-returning for every customer would actually be the action of a manager who sends coupons to every customer in his database given no support through a model. Thus, it is interesting to not consider the absolute profit (which is also difficult when comparing training sets of different sizes), but the increase in profit in comparison to the naive solution of sending coupons to everyone (predicting 0 for everyone). This is what the two functions `getBenchmarkProfit()` and `getLift()` implement.

```{r}
# Revenue function given expected revenue of 3 for true negatives,
# and -10 for false negatives. Revenue/cost of 0 for true and false positives.
getProfit <- function(obs, pred, threshold = 0.23) {
  prob <- pred
  pred <- ifelse(pred >= threshold, 1, 0)
  tp_count <- sum(obs == 1 & obs == pred)
  tn_count <- sum(obs == 0 & obs == pred)
  fp_count <- sum(obs == 0 & obs != pred)
  fn_count <- sum(obs == 1 & obs != pred)
  profit <- tn_count * 3 - fn_count * 10
  
  return(profit)
}

# get profit if we would predict every observation as "non-returning"
getBenchmarkProfit <- function(obs) { # predict non-returning for everyone
  n <- length(obs)
  getProfit(obs, rep(0, times = n))
}

# get the lift of our predictions over the benchmark profit;
# defined as getProfit/getBenchmarkProfit
getLift <- function(probs, labels, thresh) {
  pred_profit <- as.numeric(getProfit(obs = labels,
                                      pred = probs,
                                      threshold = thresh))
  naive_profit <- as.numeric(getBenchmarkProfit(labels))
  profit_lift <- pred_profit/naive_profit
  return(profit_lift)
}
```

So far so good. But if you have paid attention, you'll have noticed that one still needs to provide a threshold to the `getLift()` function. The threshold is used to take the probabilities returned by XGBoost and to turn them into the binary predictions. Elkan (2001) provides descriptions on what the optimal theoretical threshold is given the cost matrix. But thanks to our new `profitLift()` function, we can also use cross validation to compare the profits that we get using different thresholds.

## Optimal Threshold through Cross Validation

Performing proper optimization of the threshold by treating it as a hyperparameter which is tuned in nested cross validation can become a computationally expensive calculation. After all, XGBoost already has a large number of paramaters that want to be tuned.

A simpler way would be to train the model using one threshold, and choosing the optimal threshold only afterwards by applying all of them on the returned probabilities. But then the performance might be biased towards the threshold used in training, as for example the optimal number of rounds was determined using it.

Here, we consider a simple case where we fix all paramaters in our XGBoost model and only perform cross validation on the thresholds. This is easily done using the `xgb.cv()` function in the `xgboost` package. Additionally, we pass a set of parameters, `xgb_params`, as well as our evaluation metric to `xgb.cv()`. Notice that it's necessary to wrap the function we had defined before into the standardized wrapper accepted by `xgb.cv()` as an argument: `xgb.getLift()`. Also important: Set `maximize = TRUE`, since we want to maximize our profits. We then perform 5-fold cross validation for each threshold between 0.1 and 0.6 (leaving out some thresholds as they are irrelevant).

```{r}
library(xgboost)

# train features contains the training data, train_label the dependent
# variable; test features contains the test data, test_label the dependent
# variable of the test set.
dtrain <- xgb.DMatrix(as.matrix(train_features), label = train_label)
dtest <- xgb.DMatrix(as.matrix(test_features), label = test_label)

xgb_params <- list(objective = "binary:logistic",
                   eta = 0.03,
                   max_depth = 4,
                   colsample_bytree = 1,
                   subsample = 0.75,
                   min_child_weight = 1)

thresholds <- seq(0.1, 0.69, by = 0.01)
performance <- vector(length = 60)

for(i in 1:9) {
  # define the function every iteration to use a new threshold
  xgb.getLift <- function(preds, dtrain) {
    labels <- getinfo(dtrain, "label")
    lift <- getLift(preds, labels, thresholds[i])
    return(list(metric = "Lift", value = lift))
  }
  
  set.seed(512)
  # train the model again using the current iteration's threshold 
  xgb_fit <- xgb.cv(params = xgb_params, data = dtrain, nfold = 5,
                    feval = xgb.getLift, maximize = TRUE, 
                    nrounds = 250, early_stopping_rounds = 50,
                    verbose = TRUE, print_every_n = 100)
  
  # store the results
  performance[i] <- as.data.frame(xgb_fit$evaluation_log)[
    xgb_fit$best_iteration,4]
}

# print the optimal threshold
thresholds[which.max(performance)]
```

Given the results of the cross validation, we can plot the value of the `profitLift()` function against the threshold to see which threshold has lead to the optimal profit.

```{r}
# plot the evaluation metric against the thresholds
performance_df <- data.frame(threshold = thresholds,
                             performance = performance)
ggplot(performance_df, aes(x = threshold, y = performance)) +
  geom_line() +
  labs(title = "Profit lift over naive predictions using XGBoost",
          x = "Threshold", y = "Lift over naive profit")
```

![profit lift]({{ site.url }}/img/profit_lift_graph.png)

The plot shows clearly that for the standard threshold of 0.5 the XGBoost model would predict nearly every observation as non returning and would thus lead to profits that can be achieved without any model. However, by using the custom evaluation metric, we achieve a 50% increase in profits in this example as we move the optimal threshold to 0.23.