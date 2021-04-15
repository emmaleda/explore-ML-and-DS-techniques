Case Study: k-NN and palmerpenguins
================
Emma Grossman
4/8/2021

## Import data

``` r
penguins <- penguins %>%
  filter(!is.na(bill_length_mm))
```

## Normalize the continuous data

``` r
normalize <- function(x){
  return ((x - min(x)) / (max(x) - min(x)))
}
```

``` r
penguins_n <- as.data.frame(lapply(penguins[3:6], normalize))
```

## Create a testing and training data set

``` r
set.seed(1223)
smp_size <- floor(0.75 * nrow(penguins))
train_id <- sample(1:nrow(penguins), smp_size)
penguins_train <- penguins_n[train_id,]
penguins_test  <- penguins_n[-train_id,]
```

Create label vectors of response categories:

``` r
penguins_train_labels <- penguins[train_id, 1]
penguins_test_labels  <- penguins[-train_id, 1]
```

``` r
sqrt(256)
```

    ## [1] 16

Let’s chose k = sqrt(256) = 16, since we have three levels of our
response, a k of 16 will not result in a tie.

``` r
penguins_test_pred <- knn(train = penguins_train, test = penguins_test,
                           cl = penguins_train_labels[,1, drop = TRUE], 
                          k = 16) # wouldn't work without including [,1, drop = TRUE]
penguins_test_pred
```

    ##  [1] Adelie    Adelie    Adelie    Adelie    Adelie    Adelie    Adelie   
    ##  [8] Adelie    Adelie    Adelie    Adelie    Adelie    Adelie    Adelie   
    ## [15] Adelie    Adelie    Adelie    Adelie    Adelie    Adelie    Adelie   
    ## [22] Adelie    Adelie    Adelie    Adelie    Adelie    Adelie    Adelie   
    ## [29] Adelie    Adelie    Adelie    Adelie    Adelie    Adelie    Adelie   
    ## [36] Adelie    Adelie    Gentoo    Gentoo    Gentoo    Gentoo    Gentoo   
    ## [43] Gentoo    Gentoo    Gentoo    Gentoo    Gentoo    Gentoo    Gentoo   
    ## [50] Gentoo    Gentoo    Gentoo    Gentoo    Gentoo    Gentoo    Gentoo   
    ## [57] Gentoo    Gentoo    Gentoo    Gentoo    Gentoo    Gentoo    Gentoo   
    ## [64] Gentoo    Gentoo    Gentoo    Gentoo    Gentoo    Gentoo    Gentoo   
    ## [71] Chinstrap Chinstrap Chinstrap Chinstrap Chinstrap Adelie    Chinstrap
    ## [78] Adelie    Chinstrap Chinstrap Chinstrap Chinstrap Chinstrap Chinstrap
    ## [85] Chinstrap Chinstrap
    ## Levels: Adelie Chinstrap Gentoo

Now to look at the classification table:

``` r
CrossTable(x = penguins_test_labels[,1, drop = TRUE], y = penguins_test_pred,
           prop.chisq = FALSE) # wouldn't work without including [,1, drop = TRUE]
```

    ## 
    ##  
    ##    Cell Contents
    ## |-------------------------|
    ## |                       N |
    ## |           N / Row Total |
    ## |           N / Col Total |
    ## |         N / Table Total |
    ## |-------------------------|
    ## 
    ##  
    ## Total Observations in Table:  86 
    ## 
    ##  
    ##                                        | penguins_test_pred 
    ## penguins_test_labels[, 1, drop = TRUE] |    Adelie | Chinstrap |    Gentoo | Row Total | 
    ## ---------------------------------------|-----------|-----------|-----------|-----------|
    ##                                 Adelie |        37 |         0 |         0 |        37 | 
    ##                                        |     1.000 |     0.000 |     0.000 |     0.430 | 
    ##                                        |     0.949 |     0.000 |     0.000 |           | 
    ##                                        |     0.430 |     0.000 |     0.000 |           | 
    ## ---------------------------------------|-----------|-----------|-----------|-----------|
    ##                              Chinstrap |         2 |        14 |         0 |        16 | 
    ##                                        |     0.125 |     0.875 |     0.000 |     0.186 | 
    ##                                        |     0.051 |     1.000 |     0.000 |           | 
    ##                                        |     0.023 |     0.163 |     0.000 |           | 
    ## ---------------------------------------|-----------|-----------|-----------|-----------|
    ##                                 Gentoo |         0 |         0 |        33 |        33 | 
    ##                                        |     0.000 |     0.000 |     1.000 |     0.384 | 
    ##                                        |     0.000 |     0.000 |     1.000 |           | 
    ##                                        |     0.000 |     0.000 |     0.384 |           | 
    ## ---------------------------------------|-----------|-----------|-----------|-----------|
    ##                           Column Total |        39 |        14 |        33 |        86 | 
    ##                                        |     0.453 |     0.163 |     0.384 |           | 
    ## ---------------------------------------|-----------|-----------|-----------|-----------|
    ## 
    ## 

Only two penguins were misclassified\!

## Try various k

Create a function to do this because why not? why shouldn’t I?

``` r
k_NN_dif_k <- function(k, seed){
  set.seed(seed)
  smp_size <- floor(0.75 * nrow(penguins))
  
  train_id <- sample(1:nrow(penguins), smp_size)
  penguins_train <- penguins_n[train_id,]
  penguins_test  <- penguins_n[-train_id,]
  
  penguins_train_labels <- penguins[train_id, 1]
  penguins_test_labels  <- penguins[-train_id, 1]
  
  penguins_test_pred <- knn(train = penguins_train, test = penguins_test,
                           cl = penguins_train_labels[,1, drop = TRUE], 
                          k = k)
  
  return(CrossTable(x = penguins_test_labels[,1, drop = TRUE], y = penguins_test_pred,
           prop.chisq = FALSE))
}
```

``` r
k_NN_dif_k(k = 1, seed = 208)
```

    ## 
    ##  
    ##    Cell Contents
    ## |-------------------------|
    ## |                       N |
    ## |           N / Row Total |
    ## |           N / Col Total |
    ## |         N / Table Total |
    ## |-------------------------|
    ## 
    ##  
    ## Total Observations in Table:  86 
    ## 
    ##  
    ##                                        | penguins_test_pred 
    ## penguins_test_labels[, 1, drop = TRUE] |    Adelie | Chinstrap |    Gentoo | Row Total | 
    ## ---------------------------------------|-----------|-----------|-----------|-----------|
    ##                                 Adelie |        40 |         0 |         0 |        40 | 
    ##                                        |     1.000 |     0.000 |     0.000 |     0.465 | 
    ##                                        |     0.952 |     0.000 |     0.000 |           | 
    ##                                        |     0.465 |     0.000 |     0.000 |           | 
    ## ---------------------------------------|-----------|-----------|-----------|-----------|
    ##                              Chinstrap |         2 |        16 |         0 |        18 | 
    ##                                        |     0.111 |     0.889 |     0.000 |     0.209 | 
    ##                                        |     0.048 |     1.000 |     0.000 |           | 
    ##                                        |     0.023 |     0.186 |     0.000 |           | 
    ## ---------------------------------------|-----------|-----------|-----------|-----------|
    ##                                 Gentoo |         0 |         0 |        28 |        28 | 
    ##                                        |     0.000 |     0.000 |     1.000 |     0.326 | 
    ##                                        |     0.000 |     0.000 |     1.000 |           | 
    ##                                        |     0.000 |     0.000 |     0.326 |           | 
    ## ---------------------------------------|-----------|-----------|-----------|-----------|
    ##                           Column Total |        42 |        16 |        28 |        86 | 
    ##                                        |     0.488 |     0.186 |     0.326 |           | 
    ## ---------------------------------------|-----------|-----------|-----------|-----------|
    ## 
    ## 

So with k = 1, there were again, 2 penguins misclassified.

``` r
k_NN_dif_k(k = 5, seed = 21035)
```

    ## 
    ##  
    ##    Cell Contents
    ## |-------------------------|
    ## |                       N |
    ## |           N / Row Total |
    ## |           N / Col Total |
    ## |         N / Table Total |
    ## |-------------------------|
    ## 
    ##  
    ## Total Observations in Table:  86 
    ## 
    ##  
    ##                                        | penguins_test_pred 
    ## penguins_test_labels[, 1, drop = TRUE] |    Adelie | Chinstrap |    Gentoo | Row Total | 
    ## ---------------------------------------|-----------|-----------|-----------|-----------|
    ##                                 Adelie |        40 |         0 |         0 |        40 | 
    ##                                        |     1.000 |     0.000 |     0.000 |     0.465 | 
    ##                                        |     0.976 |     0.000 |     0.000 |           | 
    ##                                        |     0.465 |     0.000 |     0.000 |           | 
    ## ---------------------------------------|-----------|-----------|-----------|-----------|
    ##                              Chinstrap |         1 |        20 |         0 |        21 | 
    ##                                        |     0.048 |     0.952 |     0.000 |     0.244 | 
    ##                                        |     0.024 |     1.000 |     0.000 |           | 
    ##                                        |     0.012 |     0.233 |     0.000 |           | 
    ## ---------------------------------------|-----------|-----------|-----------|-----------|
    ##                                 Gentoo |         0 |         0 |        25 |        25 | 
    ##                                        |     0.000 |     0.000 |     1.000 |     0.291 | 
    ##                                        |     0.000 |     0.000 |     1.000 |           | 
    ##                                        |     0.000 |     0.000 |     0.291 |           | 
    ## ---------------------------------------|-----------|-----------|-----------|-----------|
    ##                           Column Total |        41 |        20 |        25 |        86 | 
    ##                                        |     0.477 |     0.233 |     0.291 |           | 
    ## ---------------------------------------|-----------|-----------|-----------|-----------|
    ## 
    ## 

With k = 5, only 1 penguins was misclassified.

``` r
k_NN_dif_k(k = 11, seed = 211)
```

    ## 
    ##  
    ##    Cell Contents
    ## |-------------------------|
    ## |                       N |
    ## |           N / Row Total |
    ## |           N / Col Total |
    ## |         N / Table Total |
    ## |-------------------------|
    ## 
    ##  
    ## Total Observations in Table:  86 
    ## 
    ##  
    ##                                        | penguins_test_pred 
    ## penguins_test_labels[, 1, drop = TRUE] |    Adelie | Chinstrap |    Gentoo | Row Total | 
    ## ---------------------------------------|-----------|-----------|-----------|-----------|
    ##                                 Adelie |        35 |         0 |         0 |        35 | 
    ##                                        |     1.000 |     0.000 |     0.000 |     0.407 | 
    ##                                        |     0.972 |     0.000 |     0.000 |           | 
    ##                                        |     0.407 |     0.000 |     0.000 |           | 
    ## ---------------------------------------|-----------|-----------|-----------|-----------|
    ##                              Chinstrap |         1 |        22 |         0 |        23 | 
    ##                                        |     0.043 |     0.957 |     0.000 |     0.267 | 
    ##                                        |     0.028 |     1.000 |     0.000 |           | 
    ##                                        |     0.012 |     0.256 |     0.000 |           | 
    ## ---------------------------------------|-----------|-----------|-----------|-----------|
    ##                                 Gentoo |         0 |         0 |        28 |        28 | 
    ##                                        |     0.000 |     0.000 |     1.000 |     0.326 | 
    ##                                        |     0.000 |     0.000 |     1.000 |           | 
    ##                                        |     0.000 |     0.000 |     0.326 |           | 
    ## ---------------------------------------|-----------|-----------|-----------|-----------|
    ##                           Column Total |        36 |        22 |        28 |        86 | 
    ##                                        |     0.419 |     0.256 |     0.326 |           | 
    ## ---------------------------------------|-----------|-----------|-----------|-----------|
    ## 
    ## 

Again, with k = 10 only 1 penguin was misclassified.
