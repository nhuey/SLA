---
title: "Bryophyte Specific Leaf Area"
author: "Nathan Huey, Harvard T.H. Chan School of Public Health, Boston, MA"
date: "09/04/2018"
output:
  html_document: 
    keep_md: yes
  pdf_document: default
  github_document:
    pandoc_args: --webtex
---


# SLA Analysis {.tabset .tabset-fade}
This is an R Markdown document. It contains both text and code with the purpose of allowing the user to follow the analysis exactly as done.

## Taking a look at SLA$_{\text{leaf}}$




This is a plot of the SLA$_{\text{leaf}}$, which is the "correct"/gold standard functional measure of SLA. Note that the thallose liverworts have been removed from analysis since they are morphologically distinct from the rest of the collected bryophytes.

<img src="some_analysis_files/figure-html/first-1.png" style="display: block; margin: auto;" />

Here we see that the variation in SLA leaf doesn't appear to be constant for all bryophytes. Pleurocarpous mosses seem to have more variation overall than the other species, which may be explained by their large numbers of leaves, allowing for more measurement error to enter.


<img src="some_analysis_files/figure-html/second-1.png" style="display: block; margin: auto;" />
Although the sample sizes are relatively low (n = 10 for the majority), the histograms of SLA leaf indicate that some of the distributions are likely non-normal (e.g. Tha_alo and Dip_alb appear to exhibit some right skew).

## Comparing SLA$_{\text{leaf}}$ and SLA$_{\text{shoot}}$

One of the key statements we'd like to make in the paper is that SLA$_{\text{shoot}}$ is an inadequate measure of SLA. My interpretation of this statement is that there isn't a reasonably simple way to predict the biologically relevant value of SLA$_{\text{leaf}}$ from the easier-to-obtain value of SLA$_{\text{shoot}}$. 

First we take a look at the relationship of the two quantities by plotting them against each other: 
<img src="some_analysis_files/figure-html/third-1.png" style="display: block; margin: auto;" />

There is certainly a positive relationship as would be expected between the two quantities, but nothing that would suggest a simple and reliable predictor. If the data are stratified by growth pattern:

<img src="some_analysis_files/figure-html/fourth-1.png" style="display: block; margin: auto;" />

The groups appear to display distinct relationships and one may ask if, given the growth pattern of a particular bryophyte, it is possible to reasonably predict SLA$_{\text{leaf}}$. 

To this end, we fit a multiple regression model allowing each growth pattern group to have a distinct intercept and slope coefficient:

```r
multiple_reg <- lm(SLAleaf ~ growth * SLAshoot, data = SLA)
summary(multiple_reg)
```

```
## 
## Call:
## lm(formula = SLAleaf ~ growth * SLAshoot, data = SLA)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -60.562 -15.342  -5.136   9.145  98.156 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)    
## (Intercept)               42.3234     7.7015   5.496 2.12e-07 ***
## growthfol_liver           38.6319    30.1499   1.281 0.202470    
## growthpleuro              33.8024    14.2950   2.365 0.019600 *  
## SLAshoot                   1.2267     0.2549   4.812 4.25e-06 ***
## growthfol_liver:SLAshoot  -0.6601     0.9785  -0.675 0.501169    
## growthpleuro:SLAshoot      1.8541     0.5319   3.486 0.000679 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 28.7 on 124 degrees of freedom
## Multiple R-squared:  0.6298,	Adjusted R-squared:  0.6149 
## F-statistic:  42.2 on 5 and 124 DF,  p-value: < 2.2e-16
```
<img src="some_analysis_files/figure-html/naive_mul_reg_plot-1.png" style="display: block; margin: auto;" />

A common way of checking the assumptions of a regression model is to look at the plot of residuals (i.e. difference between actual and fitted values) against the fitted values. The standard multiple regression model assumes that the error terms all come from a single normal distribution with mean 0 and constant variance and that the mean of the response is related in a linear way to the predictor. This would result in a band of residuals scattered around 0 with a constant width.

<img src="some_analysis_files/figure-html/unnamed-chunk-1-1.png" style="display: block; margin: auto;" />

From this plot, one can conclude that the linear model is inadequate to describe the relationship between SLA$_{\text{shoot}}$ and SLA$_{\text{leaf}}$ for the acrocarpous mosses in general and that while it may be more appropriate for pleurocarpous and foliose liverworts, the assumption of constant variance is clearly violated. 

While it may be possible to use more complicated statistical models that allow for heteroskedasticity and non-linear relationships, we believe it is reasonable from the presented analysis that no reasonable ``simple" relationship exists between SLA$_{\text{shoot}}$ and SLA$_{\text{leaf}}$ such that one could predict SLA$_{\text{leaf}}$ with a high level of confidence from SLA$_{\text{shoot}}$.

### Why no formal test that SLA$_{\text{leaf}}$ and SLA$_{\text{shoot}}$ are different?

I think the differences between SLA$_{\text{leaf}}$ and SLA$_{\text{shoot}}$ are so apparent that a formal statistical test that the two are not identical is superfluous. SLA$_{\text{shoot}}$ is consistently smaller than SLA$_{\text{leaf}}$ and we have shown that a simple and accurate transformation between the two does not exist in general.

## Differences in SLA$_{\text{leaf}}$ between growth pattern groups

It may be of interest whether significant differences exist between the SLA$_{\text{leaf}}$ values of the 4 growth patterns:



<img src="some_analysis_files/figure-html/growth_boxplots-1.png" style="display: block; margin: auto;" />

<img src="some_analysis_files/figure-html/growth_histo-1.png" style="display: block; margin: auto;" />

An informal look at the distributions seems to suggest that differences do indeed exist between the growth groups, but we might want to try to test for this formally somehow. A simple one-way ANOVA isn't going to be the ideal way of doing this however, since the data in the groups may not be normally distributed and the variances between the groups (especially for foliose liverworts) are very different from each other.


```r
SLA %>% ddply(.(growth),summarize, variance =var(SLAleaf))
```

```
##       growth   variance
## 1       acro  901.95626
## 2  fol_liver   64.91973
## 3     pleuro 2797.89848
## 4 thal_liver 1150.45690
```

In this case the non-parametric Kruskal-Wallis Test results in a highly-significant p-value, indicating that these samples do not originate from identical distributions.


```r
kruskal.test(SLAleaf~as.factor(growth), data = SLA)
```

```
## 
## 	Kruskal-Wallis rank sum test
## 
## data:  SLAleaf by as.factor(growth)
## Kruskal-Wallis chi-squared = 70.652, df = 3, p-value = 3.095e-15
```
To look at pairwise differences, we used the Conover-Iman Test, correcting for multiple testing with the Benjamini-Hochberg and Bonferroni methods:

```r
conover.test(SLA$SLAleaf, g = SLA$growth, method = "bh")
```

```
##   Kruskal-Wallis rank sum test
## 
## data: x and group
## Kruskal-Wallis chi-squared = 70.652, df = 3, p-value = 0
## 
## 
##                            Comparison of x by group                            
##                              (Benjamini-Hochberg)                              
## Col Mean-|
## Row Mean |       acro   fol_live     pleuro
## ---------+---------------------------------
## fol_live |  -3.971665
##          |    0.0001*
##          |
##   pleuro |  -9.330519  -3.379462
##          |    0.0000*    0.0006*
##          |
## thal_liv |   2.590905   5.376544   9.897866
##          |    0.0052*    0.0000*    0.0000*
## 
## alpha = 0.05
## Reject Ho if p <= alpha/2
```

```r
conover.test(SLA$SLAleaf, g = SLA$growth, method = "bonferroni")
```

```
##   Kruskal-Wallis rank sum test
## 
## data: x and group
## Kruskal-Wallis chi-squared = 70.652, df = 3, p-value = 0
## 
## 
##                            Comparison of x by group                            
##                                  (Bonferroni)                                  
## Col Mean-|
## Row Mean |       acro   fol_live     pleuro
## ---------+---------------------------------
## fol_live |  -3.971665
##          |    0.0003*
##          |
##   pleuro |  -9.330519  -3.379462
##          |    0.0000*    0.0028*
##          |
## thal_liv |   2.590905   5.376544   9.897866
##          |     0.0314    0.0000*    0.0000*
## 
## alpha = 0.05
## Reject Ho if p <= alpha/2
```

While Benjamini-Hochberg rejects the null hypothesis for all pairwise comparisons, the more stringent Bonferroni correction only fails to reject the null for thallose liverworts and acrocarpous mosses. Again, it is wise to keep in mind that only 3 species of thallose liverworts were used and that it this result may not be respresentative of the broad growth class.

## Differences in SLA$_{\text{leaf}}$ within growth pattern groups

It may also be of interest whether a single SLA value can be used for each growth pattern:

### Acrocarpous
<img src="some_analysis_files/figure-html/acro-1.png" style="display: block; margin: auto;" />

### Pleurocarpous
<img src="some_analysis_files/figure-html/pleuro-1.png" style="display: block; margin: auto;" />

### Foliose Liverworts
<img src="some_analysis_files/figure-html/fol-1.png" style="display: block; margin: auto;" />

### Thalose Liverworts
<img src="some_analysis_files/figure-html/thal-1.png" style="display: block; margin: auto;" />

We believe it suffices to show that statistically significant differences exist between species within a growth group. The main message is that a single SLA value cannot represent an entire growth pattern. A visual inspection confirms that this is indeed the case for 3 of the 4 groups, with the exception of the 2 foliose liverworts. Again, we use the Kruskall-Wallis Test as a formal test:


```r
SLA %>% group_by(growth) %>% do(test = kruskal.test(SLAleaf~as.factor(species), data = .)) %>% glance(test)
```

```
## # A tibble: 4 x 5
## # Groups:   growth [4]
##   growth     statistic  p.value parameter method                      
##   <chr>          <dbl>    <dbl>     <int> <chr>                       
## 1 acro          69.4   1.98e-12         7 Kruskal-Wallis rank sum test
## 2 fol_liver      0.691 4.06e- 1         1 Kruskal-Wallis rank sum test
## 3 pleuro        21.1   9.82e- 5         3 Kruskal-Wallis rank sum test
## 4 thal_liver    20.0   4.50e- 5         2 Kruskal-Wallis rank sum test
```

Significant differences are observed for all growth groups with the exception of the 2 foliose liverworts. Again, only using 2 species severely limits the applicablity of this result to all foliose liverworts.

## Can we use a subset of the leaves to generate SLA$_{\text{leaf}}$?

Some bryophytes have many tiny leaves on the first 1cm of a shoot. This can make it quite time consuming to accurately measure SLA.

<center>

![H. splendens](./splendens.jpg)

</center>


To this end, for a subset of the sample, an incomplete set of the leaves on the first 1cm of the shoot were used to calculate SLA. While a clearly defined method for choosing the number of leaves to use (for example based on a classification system of the number of leaves typically found on the first 1cm - "low"/"medium"/"high") is essential for the validity of the conclusions of the following analysis, we assume that the method employed here fulfills that criterium.

### Is SLA$_{\text{leaf}}$ =  SLA$_{\text{subsetleaf}}$?

The natural first question is whether or not we can just use the SLA value obtained from a subset of the leaves instead of the value that would be obtained from using all of the leaves.

One of the key assumptions of the paired t-test is a normal distribution of the differences of the two variables:

<img src="some_analysis_files/figure-html/diff_dist-1.png" style="display: block; margin: auto;" />

Although the differences do appear visually to be approximately normally distributed, a Wilcoxon-signed rank test was also applied as a more conservative non-parametric test:


```r
t.test(SLA_sub$SLAleaf, SLA_sub$SLAsubset_leaf, paired = TRUE )
```

```
## 
## 	Paired t-test
## 
## data:  SLA_sub$SLAleaf and SLA_sub$SLAsubset_leaf
## t = -4.0883, df = 129, p-value = 7.596e-05
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##  -15.862731  -5.516327
## sample estimates:
## mean of the differences 
##               -10.68953
```

```r
wilcox.test(SLA_sub$SLAleaf, SLA_sub$SLAsubset_leaf, paired = TRUE)
```

```
## 
## 	Wilcoxon signed rank test with continuity correction
## 
## data:  SLA_sub$SLAleaf and SLA_sub$SLAsubset_leaf
## V = 2227, p-value = 3.855e-06
## alternative hypothesis: true location shift is not equal to 0
```

We conclude that there is reasonable evidence that it is not generally the case that SLA$_{\text{leaf}}$ =  SLA$_{\text{subsetleaf}}$.

### Are there any particular growth groups for which this holds?

Even though it may not be generally true, it would still be very useful if SLA$_{\text{leaf}}$ =  SLA$_{\text{subsetleaf}}$ were to hold for some growth patterns. Taking a look at the distributions of the differences for each group:

<img src="some_analysis_files/figure-html/growth_histog-1.png" style="display: block; margin: auto;" />

It is difficult to determine how well these distributions approximate a normal curve. Therefore we applied the Wilcoxon-signed rank test to each of the 3 growth pattern groups individually:


```r
wilcox.test(SLA_sub$SLAleaf[(SLA_sub$growth == "acro")], SLA_sub$SLAsubset_leaf[(SLA_sub$growth == "acro")], paired = TRUE)
```

```
## 
## 	Wilcoxon signed rank test with continuity correction
## 
## data:  SLA_sub$SLAleaf[(SLA_sub$growth == "acro")] and SLA_sub$SLAsubset_leaf[(SLA_sub$growth == "acro")]
## V = 1002, p-value = 0.00737
## alternative hypothesis: true location shift is not equal to 0
```

```r
wilcox.test(SLA_sub$SLAleaf[(SLA_sub$growth == "pleuro")], SLA_sub$SLAsubset_leaf[(SLA_sub$growth == "pleuro")], paired = TRUE)
```

```
## Warning in wilcox.test.default(SLA_sub$SLAleaf[(SLA_sub$growth ==
## "pleuro")], : cannot compute exact p-value with zeroes
```

```
## 
## 	Wilcoxon signed rank test with continuity correction
## 
## data:  SLA_sub$SLAleaf[(SLA_sub$growth == "pleuro")] and SLA_sub$SLAsubset_leaf[(SLA_sub$growth == "pleuro")]
## V = 102, p-value = 0.004354
## alternative hypothesis: true location shift is not equal to 0
```

```r
wilcox.test(SLA_sub$SLAleaf[(SLA_sub$growth == "fol_liver")], SLA_sub$SLAsubset_leaf[(SLA_sub$growth == "fol_liver")], paired = TRUE)
```

```
## 
## 	Wilcoxon signed rank test
## 
## data:  SLA_sub$SLAleaf[(SLA_sub$growth == "fol_liver")] and SLA_sub$SLAsubset_leaf[(SLA_sub$growth == "fol_liver")]
## V = 50, p-value = 0.03999
## alternative hypothesis: true location shift is not equal to 0
```

Before adjusting for multiple testing, each of these tests indicates nominal evidence that SLA$_{\text{leaf}}$ and SLA$_{\text{subsetleaf}}$ are not generally interchangeable within each growth group. 

Note: in the course of the test for pleurocarpous mosses, it was found that a sample of Pleurozium schreberi had identical values for SLA$_{\text{leaf}}$ and SLA$_{\text{subsetleaf}}$, a very unlikely event that suggests a potential data entry error.

### Is there a reasonably simple way to predict SLA$_{\text{leaf}}$ from SLA$_{\text{subsetleaf}}$?

If SLA$_{\text{subsetleaf}}$ can be used to accurately predict SLA$_{\text{leaf}}$, it still may be desirable to save resources and time by only collecting the first.

<img src="some_analysis_files/figure-html/subset_leaf_plot-1.png" style="display: block; margin: auto;" />

It appears that applying logarithmic transformations may improve the assumptions of the linear model:

<img src="some_analysis_files/figure-html/log_plot-1.png" style="display: block; margin: auto;" />

Taking a look at the residuals v.s. the fitted values:

<img src="some_analysis_files/figure-html/log_model_resid-1.png" style="display: block; margin: auto;" />
we can identify potentially problematic patterns (a "w" shape in residuals, distinct patterns within growth groups). We can nonetheless get an idea of the predictive accuracy of this simple linear model as follows:

* Randomly remove approximately 20$%$ of the dataset as a test set (i.e. ~20$%$ of each species removed)
* Train the linear model on the remaining data
* Generate predictions for the test set
* Calculate Root Mean Squared Error

<img src="some_analysis_files/figure-html/RMSE-1.png" style="display: block; margin: auto;" />
The points above are the testing dataset and the line is linear model fitted on the training dataset.


```r
RMSE <- sqrt((sum((exp(predict(subset80_lm, SLA_subtest)) -(SLA_subtest$SLAleaf))^2))/nrow(SLA_subtest))
```

The RMSE for this partition of the dataset is approximately 20.6. With the potential systematic prediction errors by growth group in mind, it is also a question for the bryophyte specialist whether an "average" prediction error of this magnitude could be tolerated in applications of SLA. 

## Is SLA calculated from the first 2cm equal to that from the first 1 cm?

I'm not really certain why this is of great utility. Maybe it is biologically interesting or maybe for some species it is easier to take 2 cm instead of 1 cm? The following analysis will be almost identical to the above analysis for SLA$_{\text{subsetleaf}}$, but I've included the thallose liverworts again.

<img src="some_analysis_files/figure-html/two_diff_hist-1.png" style="display: block; margin: auto;" />

While the distribution of the differences is perhaps appropriate for a matched t-test, again, a Wilcoxon signed rank test was also applied:


```r
t.test(SLA2$SLA2cmleaf, SLA2$SLAleaf, paired = TRUE )
```

```
## 
## 	Paired t-test
## 
## data:  SLA2$SLA2cmleaf and SLA2$SLAleaf
## t = 2.7587, df = 69, p-value = 0.007423
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##   1.924556 11.978393
## sample estimates:
## mean of the differences 
##                6.951474
```

```r
wilcox.test(SLA2$SLAleaf, SLA2$SLA2cmleaf, paired = TRUE)
```

```
## 
## 	Wilcoxon signed rank test with continuity correction
## 
## data:  SLA2$SLAleaf and SLA2$SLA2cmleaf
## V = 769, p-value = 0.005639
## alternative hypothesis: true location shift is not equal to 0
```

While in general, there appear to be differences between the SLA values for 1 and 2 cm shoots, when stratified by growth pattern:

<img src="some_analysis_files/figure-html/two_diff_hist_growth-1.png" style="display: block; margin: auto;" />


```r
t.test(SLA2$SLAleaf[(SLA2$growth == "acro")], SLA2$SLA2cmleaf[(SLA2$growth == "acro")], paired = TRUE)
```

```
## 
## 	Paired t-test
## 
## data:  SLA2$SLAleaf[(SLA2$growth == "acro")] and SLA2$SLA2cmleaf[(SLA2$growth == "acro")]
## t = -0.21028, df = 19, p-value = 0.8357
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##  -6.351006  5.191352
## sample estimates:
## mean of the differences 
##              -0.5798272
```

```r
wilcox.test(SLA2$SLAleaf[(SLA2$growth == "pleuro")], SLA2$SLA2cmleaf[(SLA2$growth == "pleuro")], paired = TRUE)
```

```
## 
## 	Wilcoxon signed rank test
## 
## data:  SLA2$SLAleaf[(SLA2$growth == "pleuro")] and SLA2$SLA2cmleaf[(SLA2$growth == "pleuro")]
## V = 16, p-value = 0.2754
## alternative hypothesis: true location shift is not equal to 0
```

```r
wilcox.test(SLA2$SLAleaf[(SLA2$growth == "fol_liver")], SLA2$SLA2cmleaf[(SLA2$growth == "fol_liver")], paired = TRUE)
```

```
## 
## 	Wilcoxon signed rank test
## 
## data:  SLA2$SLAleaf[(SLA2$growth == "fol_liver")] and SLA2$SLA2cmleaf[(SLA2$growth == "fol_liver")]
## V = 37, p-value = 0.009436
## alternative hypothesis: true location shift is not equal to 0
```

```r
wilcox.test(SLA2$SLAleaf[(SLA2$growth == "thal_liver")], SLA2$SLA2cmleaf[(SLA2$growth == "thal_liver")], paired = TRUE)
```

```
## 
## 	Wilcoxon signed rank test
## 
## data:  SLA2$SLAleaf[(SLA2$growth == "thal_liver")] and SLA2$SLA2cmleaf[(SLA2$growth == "thal_liver")]
## V = 49, p-value = 0.03623
## alternative hypothesis: true location shift is not equal to 0
```

We find that the SLA values from 2 cm shoots do not show significant differences for the mosses, but that the liverworts both have nominally significant results. This is maybe a consequence of their contrasting morphologies.

Note that a matched pair t-test was used for acrocarpous mosses based on the reasonable normal-looking distribution of differences.
