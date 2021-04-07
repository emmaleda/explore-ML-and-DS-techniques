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

## Understanding the nearest neighbor classification

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

## The k-NN algorithm

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

## Measuring similarity with distance

The algorithm treats the features as coordinates in a multidimensional
feature space. It uses the Euclidean distance to measure space.

## Choosing an appropriate k

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

## Preparting data for use with k-NN

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
