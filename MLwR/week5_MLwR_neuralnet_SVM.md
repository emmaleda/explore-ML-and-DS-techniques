Black Box Methods - Neural Networks and Support Vector Machines
================
Emma Grossman
4/27/2021

**Black box** methods are called such because the model created to
obtain results is obscure and difficult to understand because of complex
mathematics.

# Understanding neural networks

A relationship between a set of input signals and an output signal can
be modeled with an **artificial neural network (ANN)**, which is derived
from mimicing how biological brains respond to stimuli from sensory
inputs.

## From biological to artificial neurons

There are several important characteristics of ANN:

  - **activation funtion** - transforms a neuron’s net input signal into
    a single output signal to be broadcasted further in the network
  - **netowrk topology** - describes the number of neurons in the model,
    as well as the number of layers and manner in which they are
    connected
  - **training algorithm**, specifies how connection weights are set in
    order to inhibit or excite neurons in proportion to the input signal

### Activation functions

A certain activation threshold must be obtained before the activation
function will fire its signal, and thus, it is called a **threshold
activation function**. Though this has parellels in biology, it is
rarely used in the real world.

A more commonly used activation function is called the **sigmoid
activation function**, which is logistic and differentiable. Other
activations functions are linear, saturatedlinear, hyperbolic tangent,
and gaussian. The differences between these functions are their support.

### Network topology

> The capacity of a neural network to learnis rooted in its
> **topolocy**, or the patters and structures of inteconnected neurons.

Key features are: - number of layers - whether information in the
network is allowed to travel backward - number of nodes within each
layer of the network

More complex networks can identify more subtle patterns.

**Multilayer networks** add hidden layers to the structure of ANN and
are generally **fully connected**, meaning that each node is connected
to the node in the next layer. A network with more than one hidden layer
is called a **deep neural network (DNN)** and training shuch models is
called **deep learning**.

If input signals are only allowed one direction, then they are called
**feedforward networks**. In contrast, if a signal can moves forward and
backward, it is called a **recurrent network** (or **feedback
network**). A **delay** can also be added to the structure, which is
short term memory that increases the power of recurrent networks a lot.
The inclusion of a delay allows the model to surmise events over time.
With all of these potential modifications, extremely complex patterns
can be learned.

Despite all of the benefits to recurrent models, multilayer feedforward
networks, also known as **multilayer perceptron (MLP)** are the de facto
standard.

Nodes can also vary. Input nodes and output nodes are predetermined by
the number of explanatory variables and the number of response
variables, but the number of hidden nodes can be determined by the user.
Some aspects that inform how many hidden nodes are (1) number of input
nodes, (2) amount of training data, (3) amount of noisy data, and (4)
complexity of the task.

More nodes means more complexity but the risk is overfitting the
training data. Common (and best) practice is to use the least amount of
nodes that results in an adequate model.

### Training neural networks with backpropagation

**Backpropagation** is the algorithm used to train an ANN model and it
uses a strategy of back-propagating errors.

Strengths:

  - can be used for classification or numeric prediction
  - models complex patterns
  - few assumptions about underlying relationships

Weaknesses:

  - slow to train and computationally intensive
  - prone to overfitting
  - can be nearly impossible to interpret

The backpropagation algorithm is an iterative process. Random weights
are chosen for each explanatory variable, a model is estimated, then the
weights are recalculated using **gradient descent** until a stopping
criteron is reached.

# Example - modeling the strength of concrete with ANNs

``` r
concrete <- read.csv("https://raw.githubusercontent.com/stedy/Machine-Learning-with-R-datasets/master/concrete.csv", header = TRUE)
str(concrete)
```

    ## 'data.frame':    1030 obs. of  9 variables:
    ##  $ cement      : num  540 540 332 332 199 ...
    ##  $ slag        : num  0 0 142 142 132 ...
    ##  $ ash         : num  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ water       : num  162 162 228 228 192 228 228 228 228 228 ...
    ##  $ superplastic: num  2.5 2.5 0 0 0 0 0 0 0 0 ...
    ##  $ coarseagg   : num  1040 1055 932 932 978 ...
    ##  $ fineagg     : num  676 676 594 594 826 ...
    ##  $ age         : int  28 28 270 365 360 90 365 28 28 28 ...
    ##  $ strength    : num  80 61.9 40.3 41 44.3 ...

## Step 2 - exploring and preparing the data

Generally, we want our data to be scaled down to a narrow range. If the
data look normal, then we standardize to a N(0,1). If they look uniform
or severely non-normal then min-max normalization is preferred.

``` r
normalize <- function(x){
  return((x - min(x)) / (max(x) - min(x)))
}
```

``` r
concrete_norm <- as.data.frame(lapply(concrete, normalize))
summary(concrete_norm$strength)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##  0.0000  0.2664  0.4001  0.4172  0.5457  1.0000

``` r
summary(concrete$strength)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##    2.33   23.71   34.45   35.82   46.13   82.60

``` r
samp <- sample(1:nrow(concrete_norm), nrow(concrete_norm)*0.75)
concrete_train <- concrete_norm[ samp, ]
concrete_test  <- concrete_norm[-samp, ]
```

## Step 3 - training a model on the data

``` r
library(neuralnet)
```

``` r
concrete_model <- neuralnet(strength ~ cement + slag + ash + water + superplastic +
                              coarseagg + fineagg + age, data = concrete_train)
plot(concrete_model)
```

The weights are indicated by the numbers of the explanatory variables
and the **bias terms** are included by in blue, labeled 1. Bias terms
allows the value at the nodes to be shifted up or down, similar to an
intercept in linear regression. The “Error” at the bottom is the Sum of
Squared Error, or the difference between the actual and predicted
values. Smaller SSEs are better

## Step 4 - evalutating model performance

``` r
model_results <- compute(concrete_model, concrete_test[1:8])
predicted_strength <- model_results$net.result
cor(predicted_strength, concrete_test$strength)
```

    ##          [,1]
    ## [1,] 0.808607

Results might differ every time the neural net is run because it starts
with random weights. For reproducable examples, we can use `set.seed()`.

This is a fairly strong correlation, which implies the model is doing a
good job. Let’s see if we can improve the performance.

## Step 5 - improving model performance

We can add `hidden` to the `neuralnet` function to include more than one
hidden node.

``` r
concrete_model2 <- neuralnet(strength ~ cement + slag + ash + water + superplastic +
                              coarseagg + fineagg + age, 
                             hidden = 5, data = concrete_train)
plot(concrete_model2)
```

The error has decreased and the steps needed to find optimal weights
have increased.

``` r
model_results2 <- compute(concrete_model2, concrete_test[1:8])
predicted_strength2 <- model_results2$net.result
cor(predicted_strength2, concrete_test$strength)
```

    ##           [,1]
    ## [1,] 0.9334814

This is a big improvement, but there is still more we can do. Our choice
of activation function will have a big impact on the results. Lately, an
activation function called **rectifier** has become popular because it
has been successful with image recognition. Rather than use the
**rectified linear unit (ReLU)** directly, we use a **softplus**
approximation since the derivative of the ReLU is not defined at x = 0.

``` r
softplus <- function(x){log(1 + exp(x))}
```

So, we’ll tell `neuralnet()` to use softplus, as well as add an
aditional 5 layer hidden layer.

``` r
set.seed(12345)
concrete_model3 <- neuralnet(strength ~ cement + slag + ash + water + superplastic +
                              coarseagg + fineagg + age, 
                             hidden = c(5,5), data = concrete_train,
                             act.fct = softplus)
# plot(concrete_model3)
```

``` r
# model_results3 <- compute(concrete_model3, concrete_test[1:8])
```

I’m not quite sure why the above code isn’t working, it worked for the
author of the book.

Our predicted values are on the normalized scale:

``` r
strengths <- data.frame(
  actual = concrete$strength[-samp],
  pred = predicted_strength2
)
head(strengths, n = 3)
```

    ##   actual      pred
    ## 1  79.99 0.7898202
    ## 6  47.03 0.4714906
    ## 9  45.85 0.3637970

It would be useful to get our predicted values back on the original
scale.

``` r
unnormalize <- function(x){
  return( (x*(max(concrete$strength)) - 
             min(concrete$strength)) + min(concrete$strength))
}
```

``` r
strengths$pred_new <- unnormalize(strengths$pred)
strengths$error <- strengths$pred_new - strengths$actual
head(strengths, n = 3)
```

    ##   actual      pred pred_new      error
    ## 1  79.99 0.7898202 65.23915 -14.750849
    ## 6  47.03 0.4714906 38.94512  -8.084877
    ## 9  45.85 0.3637970 30.04963 -15.800366

# Understanding support vector machines

A **support vector machine (SVM)** is used for classification and
numeric prediction; it creates a flat boundary called a **hyperplane**
of homogeneous groups. It is a powerful methods that can model complex
relationships.

The math is complex but SVM is most commonly used for binary
classification, and is what we’ll go over in this chapter.

## Classification with hyperplanes

If two groups can be separated perfectly by a straight line or flat
surface, they are said to be **linearly separable**.

If there are multiple lines that can be used to separate, then the
algorithm searches for the **maximum margin hyperplane (MMH)**, which
creates the greatest separation between the two groups.

**Support vectors** are the points from each class that are the closest
to MMH; each class must have at least one and they alone define the MMH.

## The case of nonlinearly separable data

If the data is nonlinearly separable, then a **slack variable** is used.
It creates a soft margin that allows some points to fall on the wrong
side of the line. A cost value is applied to all points that violate the
constraints. The greater the cost parameter, the harder the optimization
will try to achieve 100% separation.

## Using kernels for nonlinear spaces

We can also use a *kernel trick* to map the problem to a higher
dimension space and by doing this, nonlinear relationships may appear
linear.

Strengths:

  - used for classifiation and for numeric prediction
  - not overly influenced by noisy data
  - may be easier to use than neural nets
  - high accuracy in data mining competitions

Weaknesses:

  - best model requires testing of various combinations of kernels and
    model parameters
  - slow to train when there are many features
  - complex black box model that can be nearly impossible to interpret

# Example - performing OCR with SVMs

We’ll be developing a model similar to those used at **optical character
recognition (OCR)**, which process paper-based documents by converting
printed or handwritten text into electronic form.

## Step 2 - exploring and preparing the data

``` r
letters <- read.csv("https://raw.githubusercontent.com/stedy/Machine-Learning-with-R-datasets/master/letterdata.csv", header = TRUE)
str(letters)
```

    ## 'data.frame':    20000 obs. of  17 variables:
    ##  $ letter: Factor w/ 26 levels "A","B","C","D",..: 20 9 4 14 7 19 2 1 10 13 ...
    ##  $ xbox  : int  2 5 4 7 2 4 4 1 2 11 ...
    ##  $ ybox  : int  8 12 11 11 1 11 2 1 2 15 ...
    ##  $ width : int  3 3 6 6 3 5 5 3 4 13 ...
    ##  $ height: int  5 7 8 6 1 8 4 2 4 9 ...
    ##  $ onpix : int  1 2 6 3 1 3 4 1 2 7 ...
    ##  $ xbar  : int  8 10 10 5 8 8 8 8 10 13 ...
    ##  $ ybar  : int  13 5 6 9 6 8 7 2 6 2 ...
    ##  $ x2bar : int  0 5 2 4 6 6 6 2 2 6 ...
    ##  $ y2bar : int  6 4 6 6 6 9 6 2 6 2 ...
    ##  $ xybar : int  6 13 10 4 6 5 7 8 12 12 ...
    ##  $ x2ybar: int  10 3 3 4 5 6 6 2 4 1 ...
    ##  $ xy2bar: int  8 9 7 10 9 6 6 8 8 9 ...
    ##  $ xedge : int  0 2 3 6 1 0 2 1 1 8 ...
    ##  $ xedgey: int  8 8 7 10 7 8 8 6 6 1 ...
    ##  $ yedge : int  0 4 3 2 5 9 7 2 1 1 ...
    ##  $ yedgex: int  8 10 9 8 10 7 10 7 7 8 ...

All features must be numeric and scaled to a small interval. The R
package we’ll use will rescale automatically.

``` r
samp_letters <- sample(1:nrow(letters), nrow(letters)*0.75)
letters_train <- letters[ samp_letters,]
letters_test  <- letters[-samp_letters,]
```

## Step 3 - training a model on the data

``` r
# install.packages("kernlab")
library(kernlab)
letter_classifier <- ksvm(letter~., data = letters_train,
                          kernel = "vanilladot")
```

    ##  Setting default kernel parameters

``` r
letter_classifier
```

    ## Support Vector Machine object of class "ksvm" 
    ## 
    ## SV type: C-svc  (classification) 
    ##  parameter : cost C = 1 
    ## 
    ## Linear (vanilla) kernel function. 
    ## 
    ## Number of Support Vectors : 6771 
    ## 
    ## Objective Function Value : -19.7147 -17.8549 -26.6578 -4.9389 -11.1466 -30.7328 -57.8525 -18.8965 -58.0771 -33.2622 -14.8572 -31.2298 -35.512 -50.4726 -14.5396 -35.9357 -38.6089 -16.2895 -16.1912 -34.0254 -28.4355 -6.9676 -12.3003 -40.4154 -15.3827 -8.9098 -139.9354 -43.4937 -71.2279 -104.3276 -137.0796 -60.2593 -39.9192 -71.5958 -25.224 -24.1025 -18.6 -34.6751 -41.6531 -109.823 -181.5176 -186.18 -19.7318 -9.7753 -56.9182 -9.6734 -51.818 -10.5655 -20.1358 -11.4561 -105.5655 -28.2288 -218.5159 -67.8727 -7.7335 -4.3139 -139.2622 -84.5463 -17.4519 -14.3834 -69.0255 -13.279 -30.122 -16.8053 -21.8533 -26.7036 -55.6439 -11.3831 -4.96 -12.45 -4.3843 -4.193 -6.6305 -38.2959 -50.9348 -170.52 -41.7339 -40.4299 -46.2478 -15.9395 -17.9692 -78.4438 -103.3405 -46.0625 -32.3674 -115.7778 -31.0959 -27.1075 -34.548 -14.7427 -6.0174 -42.2998 -11.0719 -21.52 -53.9614 -145.3518 -47.5945 -36.7793 -28.169 -70.2598 -118.2551 -9.4976 -3.9524 -12.1871 -27.4448 -135.8507 -51.4898 -165.2142 -84.7056 -9.7645 -13.131 -2.7274 -60.3382 -6.8214 -97.9207 -47.6258 -91.1183 -64.6846 -59.4267 -22.9532 -13.7574 -8.5212 -25.4445 -14.5379 -239.6226 -29.9558 -26.5075 -121.1038 -126.8132 -10.9269 -37.2703 -7.9342 -57.7757 -71.3046 -30.7398 -206.6335 -33.1877 -15.6966 -126.1558 -168.6448 -42.3303 -24.9408 -150.2925 -72.8281 -346.2599 -126.2765 -149.2857 -32.0823 -34.736 -55.7956 -26.9221 -46.4889 -6.5325 -10.6319 -30.0641 -52.5475 -178.8881 -55.1996 -91.8099 -142.6639 -566.6631 -110.727 -136.7774 -307.8475 -29.3971 -62.0288 -147.1009 -109.7812 -37.2969 -59.9533 -47.4376 -7.4169 -184.3942 -13.9761 -36.9743 -2.1121 -5.738 -15.9022 -23.357 -60.0984 -22.0378 -177.6996 -19.3186 -5.0403 -4.5896 -0.8385 -124.7779 -8.8682 -73.3411 -18.2666 -11.5488 -4.1861 -12.1863 -24.7817 -21.4893 -67.6353 -21.08 -89.756 -13.3795 -9.3321 -6.8085 -1.3536 -72.4284 -7.1823 -99.8503 -101.8891 -42.7503 -22.0463 -64.631 -24.5312 -53.6473 -243.1707 -42.0882 -40.2738 -33.392 -18.9536 -11.8624 -117.5536 -6.1799 -5.7228 -9.4031 -11.9066 -23.9591 -22.7472 -140.6874 -35.4551 -92.3969 -31.2733 -16.3672 -10.4669 -3.2841 -92.6963 -7.3749 -11.5799 -64.343 -102.435 -14.8882 -13.8142 -56.5339 -2.9343 -7.657 -79.1204 -30.7978 -106.4893 -3.3328 -7.7394 -1.2924 -90.5278 -23.0303 -9.5453 -49.5675 -3.0465 -16.3573 -65.5023 -40.4922 -47.0257 -4.8166 -17.6041 -2.3637 -81.6728 -117.0124 -112.8069 -24.4901 -19.2281 -53.6331 -35.0535 -62.186 -22.2243 -5.956 -3.8471 -49.1512 -34.1871 -53.0653 -27.1892 -10.1247 -55.2483 -14.6599 -18.3099 -57.6715 -4.8063 -55.888 -234.1243 -14.0396 -12.3638 -15.2858 -8.1784 -65.4401 -11.4482 -36.1189 -45.5627 -23.9232 -15.0263 -46.5342 -16.4925 -63.2258 -5.5973 -5.7795 -78.1211 -3.8412 -6.5827 -1.0956 -128.0207 -23.0681 -344.8817 -31.7718 -23.5921 -3.5916 -70.4296 -140.057 -74.6951 -23.5596 -35.5442 -11.1374 -23.1367 -1.9965 -57.9478 -8.0486 -147.3549 -1.9244 -1.8521 -9.4002 -0.4593 -22.8801 -32.7344 -6.621 
    ## Training error : 0.131867

## Step 4 - evaluating model performance

``` r
letter_predictions <- predict(letter_classifier, letters_test)
head(letter_predictions)
```

    ## [1] B X O C J H
    ## Levels: A B C D E F G H I J K L M N O P Q R S T U V W X Y Z

``` r
table(letter_predictions, letters_test$letter)
```

    ##                   
    ## letter_predictions   A   B   C   D   E   F   G   H   I   J   K   L   M   N   O
    ##                  A 181   0   0   0   0   1   0   0   1   5   0   0   1   1   3
    ##                  B   0 159   0   6   2   4   0   6   0   0   3   0   1   2   0
    ##                  C   1   0 166   0   4   0   8   3   1   0   1   2   1   0   4
    ##                  D   0   4   0 175   0   0   4  14   4   2   0   1   0   3   6
    ##                  E   0   1   4   0 147   4   1   0   0   0   0   9   0   0   0
    ##                  F   0   0   1   0   2 189   0   3   4   1   0   0   0   0   0
    ##                  G   1   5   5   1   7   1 144   2   1   0   4   2   0   0   3
    ##                  H   0   2   0   2   0   0   2 114   0   0   4   1   3   3  13
    ##                  I   0   1   0   0   0   0   0   0 173  11   0   0   0   0   0
    ##                  J   2   0   0   0   0   0   0   2   5 161   0   0   0   0   0
    ##                  K   1   1   7   1   1   0   3   7   0   0 141   0   0   2   0
    ##                  L   1   0   0   0   1   0   3   1   1   0   3 190   0   0   0
    ##                  M   2   0   0   1   0   0   1   0   0   0   0   0 189   2   1
    ##                  N   0   2   0   4   0   1   0   1   0   1   0   0   0 195   1
    ##                  O   0   0   4   0   0   0   2   7   0   0   0   0   0   1 143
    ##                  P   0   1   0   0   0   2   0   0   0   0   0   0   0   0   1
    ##                  Q   0   0   0   0   3   0   5   1   1   0   0   4   0   0   1
    ##                  R   0   8   0   2   2   0   6  12   0   1  12   0   2   1   1
    ##                  S   2   1   1   0   4   6   7   0   5   6   0   1   0   0   0
    ##                  T   0   1   0   0   3   2   0   1   0   0   0   0   0   0   0
    ##                  U   0   0   3   0   0   0   1   3   0   0   1   0   4   1   2
    ##                  V   0   0   0   0   0   0   7   0   0   0   0   0   0   0   0
    ##                  W   0   0   0   0   0   0   0   0   0   0   0   0   4   1   6
    ##                  X   0   0   0   0   1   0   0   2   0   2   4   1   0   0   0
    ##                  Y   1   0   0   0   0   1   0   1   0   0   0   0   0   0   0
    ##                  Z   0   0   0   0   1   1   0   0   1   2   0   0   0   0   0
    ##                   
    ## letter_predictions   P   Q   R   S   T   U   V   W   X   Y   Z
    ##                  A   0   3   0   0   1   2   0   1   0   0   0
    ##                  B   1   0   5  16   0   0   2   0   0   0   0
    ##                  C   0   0   0   0   0   0   0   0   0   0   0
    ##                  D   1   2   3   0   1   0   0   0   0   0   0
    ##                  E   0   3   0   7   2   0   0   0   4   0   1
    ##                  F   8   0   0   4   4   0   1   0   0   2   1
    ##                  G   9   9   5   7   2   0   1   0   1   0   0
    ##                  H   0   0   3   0   1   4   1   2   0   1   0
    ##                  I   0   0   0   7   0   0   0   0   2   0   1
    ##                  J   0   1   0   0   0   0   0   0   0   0  12
    ##                  K   1   0   9   0   1   0   0   0   5   0   0
    ##                  L   0   1   0   1   0   0   0   0   1   0   0
    ##                  M   0   0   0   0   0   3   0   4   0   0   0
    ##                  N   0   0   2   0   0   0   0   3   0   0   0
    ##                  O   0   5   1   0   0   0   0   0   0   0   0
    ##                  P 177   1   1   1   0   0   0   0   0   0   0
    ##                  Q   1 160   0   7   1   0   0   0   1   3   3
    ##                  R   0   0 147   2   1   0   0   0   1   0   0
    ##                  S   0   8   0 120   1   0   0   0   2   0  17
    ##                  T   0   0   0   2 188   1   1   0   0   5   0
    ##                  U   0   0   0   0   0 188   1   3   1   1   0
    ##                  V   1   2   0   0   0   0 148   0   0   6   0
    ##                  W   0   0   0   0   0   0   2 184   0   0   0
    ##                  X   0   0   0   4   1   0   0   0 176   1   1
    ##                  Y   5   1   0   1   1   0   2   0   1 164   0
    ##                  Z   0   2   0   9   2   0   0   0   1   0 158

``` r
agreement <- letter_predictions == letters_test$letter
table(agreement)
```

    ## agreement
    ## FALSE  TRUE 
    ##   723  4277

``` r
prop.table(table(agreement))
```

    ## agreement
    ##  FALSE   TRUE 
    ## 0.1446 0.8554

## Step 5 - improving model performance

Our model is over 20 times better than random chance.

``` r
letter_classifier_rbf <- ksvm(letter~., data = letters_train,
                              kernel = "rbfdot")
```

``` r
letter_predictions_rbf <- predict(letter_classifier_rbf, letters_test)
agreement_rbf <- letter_predictions_rbf == letters_test$letter
table(agreement_rbf)
```

    ## agreement_rbf
    ## FALSE  TRUE 
    ##   311  4689

``` r
prop.table(table(agreement_rbf))
```

    ## agreement_rbf
    ##  FALSE   TRUE 
    ## 0.0622 0.9378

We can also vary the cost parameter.

``` r
# cost_values <- c(1, seq(5,40, by = 5))
# 
# accuracy_values <- sapply(cost_values, function(x) {
#   set.seed(12345)
#   m <- ksvm(letter~., data = letters_train,
#             kernel = "rbfdot", C = x)
#   pred = predict(m, letters_test)
#   agree <- ifelse(pred == letters_test$letter, 1, 0)
#   accuracy <- sum(agree) / nrow(letters_test)
#   return(accuracy)
# })
# plot(cost_values, accuracy_values, type = "b")
```
