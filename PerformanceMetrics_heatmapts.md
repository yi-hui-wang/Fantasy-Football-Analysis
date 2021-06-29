---
title: "Performance Metrics"
author: "yi-hui-wang"
date: "June 28, 2021"
output: 
  html_document: 
    keep_md: yes
---



# Topic: Evaluate the performance of predicted scores

Some online platforms such as FantasyPros (insert link) provide predicted scores of each player before a game so fantasy football players can draft an optimal lineup within the salary cap based on these predictions. However, these predictions are not perfect because of flawed predicting methods and uncontrollable conditions in real life such as player injuries. Imperfect predictions could be costly in rewards as they can lead to suboptimal lineups. Despite drawbacks, predicted scores from online platforms can still be useful as they do positively correlate with actual points sometimes and can save users time and efforts to make their own predictions. In this topic, we will evaluate performance of predicted scores by measuring their deviations from actual scores in different ways. With these measures, we can better 1. understand how predictable of each player and each position and 2. investigate whether predictability changes over time.

In this topic, you will use R to 

1.	calculate several performance metrics.

2.	make a heat map and time series plot to visualize performance metrics.

## Statistical concepts
To evaluate the performance of predicted scores, we need to compare predicted scores to actual scores. There are a variety of metrics, which describe the comparison in one number from different perspectives (https://arxiv.org/abs/1809.03006). Which one to use depends on the context of questions. Here, you will learn several metrics that are commonly used and easily implemented to fantasy football data. 

## Performance Metrics
All following metrics contain the component of error, defined as the distance between an actual value and a predicted value. In the fantasy football context, error represents the difference between an actual point and a predicted point.  

### Bias
Bias is the mean error. The positive bias represents overestimated prediction on average, whereas the negative bias represents underestimated prediction. Because positive deviation can offset negative deviation, a close-to-zero bias does not imply precise predictions.

### Mean absolute error (MAE)
MAE is the mean of the absolute value of each error. The math form (https://www.statology.org/mean-absolute-error-in-r/). Unlike bias, MAE takes the absolute value of each error before averaging, which can avoid the offset between underestimates and overestimates of predictions. As a result, MAE is never a negative value.

### Mean Absolute Percentage Error (MAPE)
In MAPE, the absolute value of each error is scaled by the actual point, leading to a dimensionless unit. It is a percentage error that describes the magnitude of error relative to the actual point. Because of its dependence on both error and actual point, a MAPE can be high even when the error is low. Although MAPE is commonly used in many fields, it should be used with caution particularly for low-scoring players. Because of its definition, the MAPE is infinite when the actual point is zero.

### Mean Squared Error (MSE)
Evidence by its name, MSE is the mean of the square of errors. Its unit is the square of points. Unlike MAE, which describes error linearly, MSE uses the square to assign more weight to a prediction with a larger error. 

### Root Mean Squared Error (RMSE)
RMSE is the root of MAE.
Except for bias, the other four metrics are never negative values. When they are zero, predictions are identical to observations. The lower the value, the better the prediction. 




```r
summary(cars)
```

```
##      speed           dist       
##  Min.   : 4.0   Min.   :  2.00  
##  1st Qu.:12.0   1st Qu.: 26.00  
##  Median :15.0   Median : 36.00  
##  Mean   :15.4   Mean   : 42.98  
##  3rd Qu.:19.0   3rd Qu.: 56.00  
##  Max.   :25.0   Max.   :120.00
```

## Including Plots

You can also embed plots, for example:

![](PerformanceMetrics_heatmapts_files/figure-html/pressure-1.png)<!-- -->

Note that the `echo = FALSE` parameter was added to the code chunk to prevent printing of the R code that generated the plot.
