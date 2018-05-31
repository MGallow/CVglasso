CVglasso
================

See [manual](https://github.com/MGallow/CVglasso/blob/master/CVglasso.pdf).

Overview
--------

<br>

<p align="center">
<img src="lik.gif">
</p>
<br>

`CVglasso` is an R package that estimates a penalized precision matrix via block-wise coordinate descent -- also known as the graphical lasso (glasso) algorithm. This package is a simple wrapper around the popular 'glasso' package and extends and enhances its capabilities. These enhancements include built-in cross validation and visualizations. A (possibly incomplete) list of functions contained in the package can be found below:

-   `CVglasso()` computes the estimated precision matrix

-   `plot.CVglasso()` produces a heat map or line graph for cross validation errors

Installation
------------

``` r
# The easiest way to install is from CRAN
install.packages("CVglasso")

# You can also install the development version from GitHub:
# install.packages("devtools")
devtools::install_github("MGallow/CVglasso")
```

If there are any issues/bugs, please let me know: [github](https://github.com/MGallow/CVglasso/issues). You can also contact me via my [website](http://users.stat.umn.edu/~gall0441/). Pull requests are welcome!

Usage
-----

``` r
library(CVglasso)

# generate data from a sparse matrix
# first compute covariance matrix
S = matrix(0.7, nrow = 5, ncol = 5)
for (i in 1:5){
  for (j in 1:5){
    S[i, j] = S[i, j]^abs(i - j)
  }
}

# print oracle precision matrix (shrinkage might be useful)
(Omega = qr.solve(S) %>% round(3))
```

    ##        [,1]   [,2]   [,3]   [,4]   [,5]
    ## [1,]  1.961 -1.373  0.000  0.000  0.000
    ## [2,] -1.373  2.922 -1.373  0.000  0.000
    ## [3,]  0.000 -1.373  2.922 -1.373  0.000
    ## [4,]  0.000  0.000 -1.373  2.922 -1.373
    ## [5,]  0.000  0.000  0.000 -1.373  1.961

``` r
# generate 1000 x 5 matrix with rows drawn from iid N_p(0, S)
Z = matrix(rnorm(100*5), nrow = 100, ncol = 5)
out = eigen(S, symmetric = TRUE)
S.sqrt = out$vectors %*% diag(out$values^0.5) %*% t(out$vectors)
X = Z %*% S.sqrt

# calculate sample covariance
Sample = (nrow(X) - 1)/nrow(X)*cov(X)

# print sample precision matrix (perhaps a bad estimate)
(qr.solve(cov(X)) %>% round(5))
```

    ##          [,1]     [,2]     [,3]     [,4]     [,5]
    ## [1,]  1.95623 -1.36048 -0.06276 -0.06776  0.09454
    ## [2,] -1.36048  4.04784 -2.06835 -0.01028  0.00739
    ## [3,] -0.06276 -2.06835  2.90974 -1.15438 -0.07357
    ## [4,] -0.06776 -0.01028 -1.15438  2.81550 -1.36917
    ## [5,]  0.09454  0.00739 -0.07357 -1.36917  2.05717

``` r
# GLASSO (lam = 0.5)
CVglasso(S = Sample, lam = 0.5)
```

    ## 
    ## 
    ## Call: CVglasso(S = Sample, lam = 0.5)
    ## 
    ## Iterations:
    ## [1] 3
    ## 
    ## Tuning parameter:
    ##       log10(lam)  lam
    ## [1,]      -0.301  0.5
    ## 
    ## Log-likelihood: -12.55568
    ## 
    ## Omega:
    ##          [,1]     [,2]     [,3]     [,4]     [,5]
    ## [1,]  1.03180 -0.17609 -0.10070  0.00000  0.00000
    ## [2,] -0.17609  1.24241 -0.38083  0.00000  0.00000
    ## [3,] -0.10071 -0.38083  0.93000 -0.28277 -0.02189
    ## [4,]  0.00000  0.00000 -0.28276  1.06893 -0.22377
    ## [5,]  0.00000  0.00000 -0.02189 -0.22377  1.07939

``` r
# GLASSO cross validation
CVglasso(X, trace = "none")
```

    ## 
    ## 
    ## Call: CVglasso(X = X, trace = "none")
    ## 
    ## Iterations:
    ## [1] 2
    ## 
    ## Tuning parameter:
    ##       log10(lam)    lam
    ## [1,]      -1.794  0.016
    ## 
    ## Log-likelihood: -116.60747
    ## 
    ## Omega:
    ##          [,1]     [,2]     [,3]     [,4]     [,5]
    ## [1,]  1.88864 -1.24863 -0.09935 -0.00366  0.00454
    ## [2,] -1.24857  3.75693 -1.87994 -0.03410  0.00000
    ## [3,] -0.09934 -1.88008  2.72611 -1.10388 -0.04351
    ## [4,] -0.00358 -0.03418 -1.10386  2.67005 -1.27430
    ## [5,]  0.00453  0.00000 -0.04348 -1.27430  1.98453

``` r
# produce CV heat map for GLASSO
CVGLASSO = CVglasso(X, trace = "none")
CVGLASSO %>% plot
```

![](README_files/figure-markdown_github/unnamed-chunk-2-1.png)

``` r
# produce line graph for CV errors for GLASSO
CVGLASSO %>% plot(type = "heatmap")
```

![](README_files/figure-markdown_github/unnamed-chunk-2-2.png)
