Lazy Learning - Classification Using Nearest Neighbors
================
Emma Grossman
3/31/2021

> \[…\] things that are alike are likely to have properties that are
> alike. Machine learning uses this principle to classify data by
> placing it in the same category as similar, or “nearest” neighbors.

Goals of this chapter: 1. understand key concepts that define nearest
neighbor classifiers 2. why they are considered “lazy” learners 3.
measuring similarity using distance 4. how to k-NN

#### Understanding the nearest neighbor classification

  - **nearest neighbor** classifiers take unlabeled observations and
    assign them a class based on similar labeled observations
  - the fundamental idea is simple but it is quite powerful,
    successfully used to
      - recognize faces in still images and in videos
      - recommend movies or songs
      - “identify patterns in genetic data to detect specific proteins
        or diseases”

> \[…\] if a concept is difficult to define, but you know it when you
> see it, then nearest neighbors might be appropriate. On the other
> hand, if the data is noisy and thus no clear distinction exists among
> the groups, nearest neighbor algorithms may struggle to identify the
> class boundaries

### The k-NN algorithm

Strengths

  - simple and effective
  - no underlying assumptions about distribution
  - fast training phase

Weaknesses:

  - no model produced: limits our understanding of how features are
    related to class
  - must know k
  - slow at classifying
  - nominal features and missing data require additional processing

The name, k-NN, comes from the fact that the algorithm uses information
about an examples *k* nearest neighbors to classify unlabeled examples;
*k* is the number of nearby variables to use.

A training dataset is required with examples that have already been
classified into categories, labeled by a nominal variable. “Then, for
each unlabeled record in the test dataset, k-NN identifies the *k*
records in the training data that are ‘nearest’ in similarity. The
unlabeled test instance is assigned the class representing the majority
of the *k* nearest neighbors.”

### Measuring similarity with distance

The algorithm treats the features as coordinates in a multidimensional
feature space. It uses the Euclidean distance to measure space.

### Choosing an appropriate k

The balance between overfitting and underfitting is the **bias-variance
tradeoff**: large *k* reduces the variance caused by noisy data but can
bias the learner to ignore smaller patterns.

Hypothetically, if we chose *k* = *n*, the number of observations in our
training data, the most common class would always be the majority class,
which disregards the actual nearest neighbors. In contrast, if we chose
*k* = 1, noisy data and outliers would strongly influence the
classification. Smaller *k* values allow more complex boundaries which
more carefully fit the training data.

In practice, we should chose *k* based on how difficult the concept that
needs to be learned is. It is common to begin with *k* = sqrt(n).
Another option is to try several different *k* on a variety of training
examples and chose the one that classifies most accurately. If we have a
large dataset, however, the choice of *k* isn’t as important. One last
thought is to use weighted voting, which weights points nearer to the
unlabeled observation as more important than points farther away.

### Preparting data for use with k-NN

Features should be transformed to a standard range before
implementation, otherwise features with a larger range than the others
will dominate the algorithm’s process. What is typically used is the
**min-max normalization**: all of the values will fall between 0 and 1.
\(X_{new} = \frac{\text{X}- \text{min(X)}}{\text{max(X)} - \text{min(X)}}\).
Another option is **z-score standardization**.

The same re-scaling must happen to both the testing and training
dataset. This can be tricky if we have different minimum and maximum
values, but if we use z-score standardization, and assume that we’ll
have similar means and SDs, it usually works out okay.

The Euclidean distance is not defined for nominal data, so we must use a
**dummy variable**, or an indicator variable, to change nominal data
into numeric data. This is also called **one-hot encoding**. It is also
convenient that because all values are 0s and 1s, they will be on the
same scale as min-max normalized data

### Why is the k-NN algorithm lazy?

The definition of a **learning algorithm** is that it abstracts and
generalizes processes but there is no abstraction that occurs in k-NN,
which is why it is considered lazy. So, the lazy learner is not actually
*learning* anything, it is just storing the training data verbatim; the
training phase is thus very quick. The downside is that the prediction
process is then quite slow by comparison. Since the reliance on the
training is to heavy, the lazy learning model is known as an
**instance-based learning** or **rote learning**.

Because no model is built, the method is **non-parametric** which limits
our ability to understand about the how the classifier is using the
data. It gives the learner the freedom, however, to identify natural
patterns that may not be well represented by a parametric model.

## Example - diagnosing breast cancer with the k-NN algorithm

### Step 1 - collecting data

### Step 2 - exploring and preparing the data

``` r
wbcd <- read.csv("~/Documents/School/OSU/6. Spring 2021/R&C/explore-ML-and-DS-techniques/MLwR/wdbc.data",
                 header=FALSE)
wbcd <- wbcd %>%
  mutate(id = V1,
         diagnosis = V2,
         radius_mean = V3,
         texture_mean = V4,
         perimeter_mean = V5,
         area_mean = V6,
         smoothness = V7,
         compactness = V8,
         concavity = V9,
         concave_points = V10,
         symmetry = V11,
         fractal_dim = V12) %>%
  select(-c(V1,V2,V3,V4,V5,V6,V7,V8,V9,V10,V11,V12))

wbcd <- wbcd[,c(21:32,1:20)]
```

We’re first going to drop the id column

``` r
wbcd <- wbcd[-1]
```

The second column, diagnosis, is the outcome we’re trying to predict.

``` r
table(wbcd$diagnosis)
```

    ## 
    ##   B   M 
    ## 357 212

It is already a factor variable, which is generally required for
learning classifiers, but let’s relabel the factor levels.

``` r
wbcd$diagnosis <- factor(wbcd$diagnosis, levels = c("B","M"), 
                         labels = c("Benign", "Malignant"))
```

Let’s now look at the proportion of each level of classification:

``` r
round(prop.table(table(wbcd$diagnosis))*100, digits = 1)
```

    ## 
    ##    Benign Malignant 
    ##      62.7      37.3

The other 30 features are all numeric and consist of three different
measurements of 10 characteristics.

``` r
summary(wbcd[c("radius_mean", "area_mean", "smoothness")])
```

    ##   radius_mean       area_mean        smoothness     
    ##  Min.   : 6.981   Min.   : 143.5   Min.   :0.05263  
    ##  1st Qu.:11.700   1st Qu.: 420.3   1st Qu.:0.08637  
    ##  Median :13.370   Median : 551.1   Median :0.09587  
    ##  Mean   :14.127   Mean   : 654.9   Mean   :0.09636  
    ##  3rd Qu.:15.780   3rd Qu.: 782.7   3rd Qu.:0.10530  
    ##  Max.   :28.110   Max.   :2501.0   Max.   :0.16340

Looking at the above summaries, all the variables have very different
means and sds, which is a problem with k-NN. We must standardize our
variables before continuing on.

Create a normalize function:

``` r
normalize <- function(x){
  return ((x - min(x)) / (max(x) - min(x)))
}
```

We can use `lapply()` to apply this function to all the column vectors
in our data set.

``` r
wbcd_n <-  as.data.frame(lapply(wbcd[2:31], normalize))
```

Let’s check one of the variables to ensure that our code worked
correctly:

``` r
summary(wbcd_n$area_mean)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##  0.0000  0.1174  0.1729  0.2169  0.2711  1.0000

We’ll now divide our data into two sections: a training set to build the
k-NN model and a testing set to test the predictive accuracy of the
model.

``` r
set.seed(1119)
train_id <- sample(1:569, 469)
wbcd_train <- wbcd_n[train_id,]
wbcd_test <- wbcd_n[-train_id,]
```

When normalizing the data, we didn’t include the response variable so we
should save these values as well:

``` r
wbcd_train_labels <- wbcd[train_id, 1]
wbcd_test_labels <- wbcd[-train_id, 1]
```

### Step 3 - training a model on the data

We’ll use a k-NN implementer from the `class` package.

``` r
# install.packages("class")
library(class)
```

This function is a classic approach to k-NN and when ties occur during
voting, they are broken at random. The `knn` function from the `class`
package requires four arguments: the training set, the testing set, a
vector of the true labels of the training set and the number of
neighbors to consider.

We can now proceed with our data.

``` r
sqrt(469)
```

    ## [1] 21.65641

We can choose k = 21 since it is odd and roughly equal to the square
root of 469, the number of data in our training set. Using an odd number
eliminates the possibility of a tie vote, since we have two categories
of our outcome variable.

``` r
wbcd_test_pred <- knn(train = wbcd_train, test = wbcd_test,
                      cl = wbcd_train_labels, k = 21)
```

What `knn` returns is a factor vector of predicted labels in the testing
data set.

### Step 4 - evaluating model performance

Now we want to know how well our model did at predicting the correct
outcome. We can use the `CrossTable()` function in the `gmodels` package
to do this.

``` r
# install.packages("gmodels")
library(gmodels)
```

``` r
CrossTable(x = wbcd_test_labels, y = wbcd_test_pred,
           prop.chisq = FALSE)
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
    ## Total Observations in Table:  100 
    ## 
    ##  
    ##                  | wbcd_test_pred 
    ## wbcd_test_labels |    Benign | Malignant | Row Total | 
    ## -----------------|-----------|-----------|-----------|
    ##           Benign |        63 |         0 |        63 | 
    ##                  |     1.000 |     0.000 |     0.630 | 
    ##                  |     0.969 |     0.000 |           | 
    ##                  |     0.630 |     0.000 |           | 
    ## -----------------|-----------|-----------|-----------|
    ##        Malignant |         2 |        35 |        37 | 
    ##                  |     0.054 |     0.946 |     0.370 | 
    ##                  |     0.031 |     1.000 |           | 
    ##                  |     0.020 |     0.350 |           | 
    ## -----------------|-----------|-----------|-----------|
    ##     Column Total |        65 |        35 |       100 | 
    ##                  |     0.650 |     0.350 |           | 
    ## -----------------|-----------|-----------|-----------|
    ## 
    ## 

Top-left: true negative 63/100 Bottom-right: true positive 35/100
Bottom-left: false negative 2/100 top-right: false positive 0/100

In total, 2 out of 100 masses were incorrectly classified; we could
improve this with another iteration of the model.

### Step 5: improving model performance

We’ll try a couple ways of improving model performance

1.  z-score standardization rather than min-max normalization
2.  different values for k

#### Transformation - z-score standardization

z-score standardization allows outliers to be seen in the data, min-max
compresses extreme observations toward each other. The `scale()`
function can be applied directly to a data frame, so there is no need to
use `lapply()`.

``` r
wbcd_z <- as.data.frame(scale(wbcd[-1]))

wbcd_train <- wbcd_z[train_id,]
wbcd_test <- wbcd_z[-train_id,]
wbcd_train_labels <- wbcd[train_id, 1]
wbcd_test_labels <- wbcd[-train_id, 1]
wbcd_test_pred <- knn(train = wbcd_train, test = wbcd_test,
                      cl = wbcd_train_labels, k = 21)
CrossTable(x = wbcd_test_labels, y = wbcd_test_pred,
           prop.chisq = FALSE)
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
    ## Total Observations in Table:  100 
    ## 
    ##  
    ##                  | wbcd_test_pred 
    ## wbcd_test_labels |    Benign | Malignant | Row Total | 
    ## -----------------|-----------|-----------|-----------|
    ##           Benign |        63 |         0 |        63 | 
    ##                  |     1.000 |     0.000 |     0.630 | 
    ##                  |     0.969 |     0.000 |           | 
    ##                  |     0.630 |     0.000 |           | 
    ## -----------------|-----------|-----------|-----------|
    ##        Malignant |         2 |        35 |        37 | 
    ##                  |     0.054 |     0.946 |     0.370 | 
    ##                  |     0.031 |     1.000 |           | 
    ##                  |     0.020 |     0.350 |           | 
    ## -----------------|-----------|-----------|-----------|
    ##     Column Total |        65 |        35 |       100 | 
    ##                  |     0.650 |     0.350 |           | 
    ## -----------------|-----------|-----------|-----------|
    ## 
    ## 

Standardizing with z-score produced the same accuracy as with min-max
normalization.

#### Testing alternative values of k

The result of testing several k values with different test data sets is
below:

``` r
data.frame(k_value = c(1, 5, 11, 15, 21, 27),
           false_neg = c(1, 2, 3, 3, 2, 4),
           false_pos = c(3, 0, 0, 0, 0, 0),
           perc_inc_class = c("4 percent", "2 percent", "3 percent",
                              "3 percent", "2 percent", "4 percent")
             )
```

    ##   k_value false_neg false_pos perc_inc_class
    ## 1       1         1         3      4 percent
    ## 2       5         2         0      2 percent
    ## 3      11         3         0      3 percent
    ## 4      15         3         0      3 percent
    ## 5      21         2         0      2 percent
    ## 6      27         4         0      4 percent

A k of 1 was about to avoid false negatives at the expense of false
positives. We do not want to overfit our data, and tailor our approach
too closely with our test data.
