---
title: "T-Test to evaluate drafting from shoot-out games"
author: "yi-hui-wang"
date: "August 7, 2021"
output: 
  html_document: 
    keep_md: yes
---



# Topic: Does drafting players from shoot-out games increase the chance of winning in a daily league?

Predicted points are not perfect, so it is challenging to create a winning lineup from hundreds of players based on these predictions. Since it is difficult to make an accurate prediction, it may be easier to predict shoot-out games. If we can predict shoot-out games, we could narrow down the pool to the players in those games, who are more likely to outperform many players. In this topic, we aim to investigate whether drafting players from shoot-out games leads to a higher total score than drafting players from all games using a common hypothesis testing.

## Statistical Concepts
To answer this question, we will use a common hypothesis testing, t-test, to compare the mean total score of lineups from players in shoot-out games only with that from players in all games. The t-test is an objective approach to determine whether the mean of one group is different (or better/less) than the mean of the other group. 

### T-test
It is easy to use R to apply a t-test. Using the function, t.test, all we need are the data from two samples: the total actual score of each lineup drafted from players in shoot-out games and that from players in all games. The function outputs t statistic, degrees of freedom, p-value, alternative hypothesis, 95% confidence interval, and the mean of each sample.

The general idea of a hypothesis test is to examine a claim about a conjecture. The claim is often expressed as an alternative hypothesis as opposed to a null hypothesis, which assumes no difference between two groups. In this case, our conjecture expressed in the alternative hypothesis is whether the mean total score of lineups from shoot-out games is better than that from all games. To determine whether the data collected support the claim, one key output is the p-value. P-value represents the chance of getting the result from collected data if the null hypothesis is true. Here the p-value is the chance of getting the difference in the mean total score between two sample data, assuming one group is not better than the other. The lower the p-value, the more unlikely to get what collected data show if the null hypothesis is true. In other words, a very small p-value implies a rejection of the null hypothesis, and thus a support for the alternative hypothesis. In this example, a very small p-value would be indicative of a significant difference between two groups, meaning that the difference observed from the sample data is unlikely to be due to random chance when there is no difference.  

Another output that can be used for drawing a conclusion is a confidence interval. A confidence interval and hypothesis test are like two sides of a story. Their results must be consistent to reach a same conclusion. If the alternative hypothesis is supported by sample data, the confidence interval of the difference between the mean from shoot-out games and the mean from all games must be greater than 0.

Note that the default of the t.test function is to perform a two-sided test. That is to examine whether the means of two groups are different or not, meaning the difference between the means can be greater than 0 or smaller than 0. To perform a one-sided test like our case, we need to add ‘alternative = greater’. On the contrary, if you want to investigate a conjecture of whether the mean from Group 1 is less than that from Group 2, you need to use ‘alternative = less’.

Another default in the function is to use Welch’s t-test, assuming two groups have difference variance. If the two groups you use appear to have same variance, you can add ‘var.equal = TRUE’ to convert to a two-sample t-test. For a two-sample t-test, it uses pooled degrees of freedom that could lead to a slightly different t-distribution, and thus a slightly different p-value.

# Exploring the data


```r
library(dplyr)
library(ggplot2)
```
We used Week 15 in 2020 season for demonstration. As the p-value using different samples would be different, the conclusion may change if choosing another week. But the concept and analysis are applicable to other weeks or seasons. 

Here shoot-out games are defined as the top four games that had most combined points from two teams. Four games are a quarter of games for a typical week without any bye. In Week 15, the shoot-out games are: Buffalo Bills, Detroit Lions, New Orleans Saints, San Francisco 49ers, Tennessee Titans, Denver Broncos, Kansas City Chiefs, Dallas Cowboys.

To perform the t-test, the first step is to load 70 lineups based on predictions, optimized by the Genetic Algorithm (topic link) from the dataset with players in shoot-out games only and the dataset with all players, respectively. As a small sample size does not meet the assumption of data normality in t-test, we chose 70 to balance sample size and running time. If you only have a small sample of data, you can use a nonparametric test like bootstrap instead of t-test.


```r
### load GA between shoot-out games and all games
ga.shootout.wk15 <- read.csv("D:/fantasyfootball/data2020/GA_2020week15_fantasypro_totalscores.csv")
ga.all.wk15 <- read.csv("D:/fantasyfootball/data2020/GA_2020week15_fantasypro_allgames.csv")

# eliminate unwanted columns for a better snapshot
ga.shootout.wk15 <- ga.shootout.wk15 %>% select(-X.1, -Unnamed..0, -X, -h.a, -Oppt)
```

Here is the snapshot of the file, which shows the information of each lineup, including players, positions, team, salary, actual point (i.e. DK.points), and predicted point (i.e. Points). 


```r
# snapshot of the file
print(ga.shootout.wk15[1:20,])
```

```
##    Week Year             Name Position Team DK.points Salary      Points
## 1    15 2020       Josh Allen       QB  buf     40.66   7200        21.1
## 2    15 2020    D'Andre Swift       RB  det      23.2   6400        16.0
## 3    15 2020     Alvin Kamara       RB  nor      18.4   7400 25.81060643
## 4    15 2020    Brandon Aiyuk       WR  sfo      22.3   6300        16.3
## 5    15 2020       A.J. Brown       WR  ten      15.4   7600        17.6
## 6    15 2020 Emmanuel Sanders       WR  nor      11.6   4200        13.4
## 7    15 2020   Danny Amendola       WR  det       5.0   4200        10.0
## 8    15 2020       Jared Cook       TE  nor       4.9   3400        10.1
## 9    15 2020        Seahawks       DST  sea       9.0   3100         8.5
## 10                                                                      
## 11 Week Year             Name Position Team DK.points Salary      Points
## 12   15 2020       Drew Brees       QB  nor     20.36   5900        17.6
## 13   15 2020     Alvin Kamara       RB  nor      18.4   7400 25.81060643
## 14   15 2020 Devin Singletary       RB  buf      17.4   4100         9.5
## 15   15 2020      Corey Davis       WR  ten      24.0   5800        14.6
## 16   15 2020    Brandon Aiyuk       WR  sfo      22.3   6300        16.3
## 17   15 2020 Emmanuel Sanders       WR  nor      11.6   4200        13.4
## 18   15 2020      Tim Patrick       WR  den       4.4   4300        11.3
## 19   15 2020     Travis Kelce       TE  kan      22.8   8000        18.9
## 20   15 2020        Steelers       DST  pit       2.0   3900        13.7
```

To prepare data for t-test, we created a data frame that combines the total actual point of each lineup in the same group. We also included the total salary and total predicted point so now there are three columns in the data frame. 


```r
### document total scores and salary of each lineup
# keep columns needed
ga.shootout.small <- ga.shootout.wk15 %>% select(DK.points, Salary, Points)
ga.all.small <- ga.all.wk15 %>% select(DK.points, Salary, Points)

ist <- seq(1, nrow(ga.shootout.wk15), by = 11)
shootout.info <- data.frame()
all.info <- data.frame()

# keep total scores and salary only
for (i in 1:length(ist)){
  # lineups from shoot-out games
  shootouti <- ga.shootout.small[ist[i]:(ist[i]+8),] 
  shootouti <- sapply(shootouti, as.numeric)
  shootout.info <- bind_rows(shootout.info, colSums(shootouti))
  
  # lineups from all games
  alli <- ga.all.small[ist[i]:(ist[i]+8),] 
  alli <- sapply(alli, as.numeric)
  all.info <- bind_rows(all.info, colSums(alli))
}

head(shootout.info)
```

```
##   DK.points Salary   Points
## 1    150.46  49800 138.8106
## 2    143.26  49900 141.1106
## 3    168.86  49700 137.5106
## 4    137.46  49800 138.8106
## 5    174.26  49900 135.8106
## 6    156.82  49900 140.0106
```

Use the data frame for the t-test.


```r
ttest <- t.test(shootout.info$DK.points, all.info$DK.points, alternative = "greater")
print(ttest)
```

```
## 
## 	Welch Two Sample t-test
## 
## data:  shootout.info$DK.points and all.info$DK.points
## t = 7.0071, df = 137.87, p-value = 4.923e-11
## alternative hypothesis: true difference in means is greater than 0
## 95 percent confidence interval:
##  15.57563      Inf
## sample estimates:
## mean of x mean of y 
##  150.0554  129.6597
```

The output shows that the p-value is nearly 0, indicating the rejection of the null hypothesis in favor of the alternative hypothesis or our conjecture. In Week 15, the mean total actual score using players in shoot-out games is about 150 points and that in all games is about 130 points. A 20-point difference can for sure make a positive impact on your winning record in a daily league!

The result makes sense. But you may wonder if there is a week showing no significant difference between the two groups. We can repeat the t-test to lineups from Week 1 to Week 15. 


```r
### loop through all weeks
diffmean_wk <- c()
p_wk <- c()

for (iwk in seq(1,15)){
  ga.shootout <- read.csv(paste0('D:/fantasyfootball/data2020/GA_2020week',iwk,'_fantasypro_totalscores.csv')) %>% 
    select(DK.points, Salary, Points)
  ga.all <- read.csv(paste0('D:/fantasyfootball/data2020/GA_2020week',iwk,'_fantasypro_allgames.csv')) %>%
    select(DK.points, Salary, Points)
  
  ist <- seq(1, nrow(ga.shootout), by = 11)
  shootout.info <- data.frame()
  all.info <- data.frame()
  
  # keep total scores and salary only
  for (i in 1:length(ist)){
    # lineups from shoot-out games
    shootouti <- ga.shootout[ist[i]:(ist[i]+8),] 
    shootouti <- sapply(shootouti, as.numeric)
    shootout.info <- bind_rows(shootout.info, colSums(shootouti))
    
    # lineups from all games
    alli <- ga.all[ist[i]:(ist[i]+8),] 
    alli <- sapply(alli, as.numeric)
    all.info <- bind_rows(all.info, colSums(alli))
  }
  ttest <- t.test(shootout.info$DK.points, all.info$DK.points, alternative = "greater")
  
  # difference in means
  diffmean_wk[iwk] <- ttest$estimate[1]-ttest$estimate[2]
  
  # p-value
  p_wk[iwk] <- ttest$p.value
}

# combine data for plot
comp.df <- data.frame(weeki = seq(1,15), difference_mean = diffmean_wk, p_value = p_wk) %>%
  mutate(Color = ifelse(p_value <=0.05, "p <= 0.05", "p > 0.05"))
```

Here is the difference between two means and their p-values in each week. Among 15 weeks, 12 of them support our conjecture using the significance level, 0.05. 


```r
print(comp.df)
```

```
##    weeki difference_mean      p_value     Color
## 1      1       42.407429 1.641925e-18 p <= 0.05
## 2      2        9.292857 1.787811e-02 p <= 0.05
## 3      3       30.575714 1.826601e-15 p <= 0.05
## 4      4       20.186571 2.799241e-08 p <= 0.05
## 5      5        5.348286 3.749798e-02 p <= 0.05
## 6      6       29.704286 1.887363e-16 p <= 0.05
## 7      7       25.778857 1.384226e-09 p <= 0.05
## 8      8      -17.572000 9.999997e-01  p > 0.05
## 9      9        5.124286 8.242504e-02  p > 0.05
## 10    10       34.341714 8.176947e-20 p <= 0.05
## 11    11       32.769143 1.179018e-20 p <= 0.05
## 12    12       14.022857 9.779276e-04 p <= 0.05
## 13    13      -18.776857 9.999976e-01  p > 0.05
## 14    14       12.290286 9.672682e-07 p <= 0.05
## 15    15       20.395714 4.922687e-11 p <= 0.05
```

We can also visualize the change of the difference over weeks. Based on the results of the t-test, we are confident that there is a better chance to create a winning lineup drafting from players in shoot-out games. 


```r
### plot of difference in means over weeks
ggplot(data = comp.df, aes(x = weeki, y = difference_mean)) +
  geom_line() +
  geom_point(data = comp.df, aes(x = weeki, y = difference_mean, color = Color), size = 3) +
  xlab("week number") +
  ylab("mean(shootout) - mean(all)")
```

![](TTest_shootoutgames_files/figure-html/unnamed-chunk-8-1.png)<!-- -->


# Summary
In this topic, we used the t-test to investigate whether drafting players from shoot-out games only improves our daily leagues performance compared to drafting from all games. In real life, we wouldn’t know which games are shoot-out games until games are played. Therefore, our next effort is to predict shoot-out games with uncertainty so our drafting can focus on the players in those games. 
