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