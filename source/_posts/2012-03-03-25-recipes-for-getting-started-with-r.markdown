---
layout: post
title: "25 Recipes for Getting Started with R"
date: 2012-03-03 20:42
comments: true
categories: Book Reading
---

## 1. Downloading and Installing R ##

*Noting special*

## 2. Getting Help on a Function ##

`help()`, `args()`, `example()`, `?func`

## 3. Viewing the Supplied Documentation ##

`help.start()`

## 4. Searching the Web for Help ##

`RSiteSearch("hey phrase")`

## 5. Reading Tabular Datafiles ##

    read.table("target.txt", stringsAsFactor=FALSE)

*will prevent `read.table()` from interpreting character string as factor*.

Other options: *`na.strings="."`*, *`header=T`*, etc.

## 6. Reading from CSV Files ##

    read.csv("filename", header=T, as.is=T)

`as.is=T` indicates that **R** should not interpret nonnumeric data as a *factor*.

## 7. Creating a Vector ##

`c(...)`

## 8. Computing Basic Statistics ##

- `na.rm=T` ignore *NA* values in data, otherwise result would be invalidated;
- `lapply()` is magic.


## 9. Initializing a Data Frame from Column Data ##

    dfrm <- data.frame(v1, v2, v3, v4)
    dfrm <- as.data.frame(list.of.vectors)

## 10. Selecting Data Frame Column by Position ##

- `dfrm[[n]]`, `dfrm[, n]`: returns *n*th column;
- `dfrm[n]`: returns a data frame of *n*th column;
- `dfrm[c(n1, n2, ..., nk)]`, `dfrm[, c(n1, n2, ..., nk)]`: returns a data frame of *k* columns.

## 11. Selecting Data Frame column by Name ##

Similar to the previous section.

- `dfrm[["name"]]`, `dfrm$name`: returns *one column*, called *name*;
- `dfrm["name"]`, `dfrm[c("name1", "name2", ..., "name3")]`, `dfrm[, "name"]`, `dfrm[, c(...)]`.

## 12. Forming a Confidence Interval for a Mean ##

- `t.test(x)`: apply to sample *x*, to determine a confidence interval;
- `conf.level` argument: see intervals at other levels.

## 13. Forming a Confidence Interval for a Proportion ##

- `prop.test(n, x)`: sample size *n*, *x* successes;
- use `conf.level` argument for other confidence levels.

## 14. Comparing the Means of Two Samples ##

By default, `t.test()` assumes data is not paired. Test with two sample *x*, *y*:
    
    t.test(x, y, paired=T)

## 15. Testing a Correlation for Significance ##

- `cor.test(x, y, method="Spearman")` for nonnormally data; default is *Pearson* method.
- the function returns several values, including *p*-value from the test of significance. <u>*p* < 0.05</u> indicates that the correlation is likely significant; otherwise, not.

## 16. Creating a Scatter Plot ##

- `plot(x, y)`: two parallel vectors *x* and *y*;
- `plot(dfrm)`: for data frame.

## 17. Creating a Bar Chart ##

- `barplot(vector)`;
- ref `tapply()`;
- `barchart()` from *lattice* package.

## 18. Creating a Box Plot ##

- `boxplot(vector)`

## 19. Creating a Histogram ##

- `hist(vector, number.of.bins)`: 7 bins by default;
- `histogram()` from *lattice* package.

## 20. Performing Simple Linear Regression ##

- `lm(y ~ x)`: `y ~ x` is a *model formula*

## 21. Performing Multiple Linear Regression ##

- `lm(y ~ u + v + w)`

## 22. Getting Regression Statistics ##

- `m <- lm(y ~ u + v + w)`

        anova(m)                        ANOVA table
        coef(m), coefficients(m)        Model coefficients
        confint(m)                      Confidence intervals for the regression coefficients
        deviance(m)                     Residual sum of squares
        effects(m)                      Vector of orthogonal effects
        fitted(m)                       Vector of fitted *y* values
        resid(m), residuals(m)          Model residuals
        summary(m)                      Key statistics
        vcov(m)                         Variance-covariance matrix of the main parameters

## 23. Diagnosing a Linear Regression ##

``` R
m <- lm(y ~ x)
plot(m)

library(car)
outlier.test(m)
```

## 24. Predicting New Values ##

``` R
m <- lm(y ~ u + v + w)
preds <- data.frame(u=3.1, v=4.0, w=5.5)
predict(m, newdata=preds)

preds <- data.frame(
            u=c(...),
            v=c(...),
            w=c(...))
predict(m, newdata=preds)
```

- use `interval="prediction"` argument of `predict()` to obtain the confidence intervals.

## 25. Accessing the Functions in a Package ##

- `library(packagename)`;
- `detach(package:name)`;
- higher function **masks** the lower function.

---
#†#
