Advanced R: Vectors
================
Emma Grossman
6/1/2021

## 3.2 Atomic vectors

There are four types of atomic vectors: logical, integer, double, and
character. Two other very rare types exist: complex and raw.

`c()` is short for combine.

We can use `typeof()` to determine the type of a vector.

``` r
lgl_var <- c(TRUE, FALSE)
int_var <- c(1L, 6L, 10L)
dbl_var <- c(1, 2.5, 4.5)
chr_var <- c("these are", "some strings")

typeof(lgl_var)
```

    ## [1] "logical"

`NA` is used for missing data. It is infections: most operations
including a `NA` value returns another `NA` value, with a few
exceptions.

``` r
NA > 5
```

    ## [1] NA

``` r
10 * NA
```

    ## [1] NA

``` r
!NA
```

    ## [1] NA

The exceptions:

``` r
NA ^ 0
```

    ## [1] 1

``` r
NA | TRUE
```

    ## [1] TRUE

``` r
NA & FALSE
```

    ## [1] FALSE

We use `is.na()` to test for missingness rather than `something == NA`.

If we attempt to combine vectors of different types, coercion occurs.

``` r
str(c("a", 1))
```

    ##  chr [1:2] "a" "1"

``` r
x <- c(FALSE, FALSE, TRUE)
as.numeric(x)
```

    ## [1] 0 0 1

``` r
# Total number of TRUEs
sum(x)
```

    ## [1] 1

``` r
# Proportion that are TRUE
mean(x)
```

    ## [1] 0.3333333

``` r
as.integer(c("1", "1.5", "a"))
```

    ## Warning: NAs introduced by coercion

    ## [1]  1  1 NA

## 3.3 Attributes

Matrices, arrays, factors, and date-times are data structures built on
vectors using attributes.

Attributes attach metadata to data.

``` r
a <- 1:3
attr(a, "x") <- "abcdef"
attr(a, "x")
```

    ## [1] "abcdef"

``` r
attr(a, "y") <- 4:6
str(attributes(a))
```

    ## List of 2
    ##  $ x: chr "abcdef"
    ##  $ y: int [1:3] 4 5 6

``` r
# Or equivalently
a <- structure(
  1:3, 
  x = "abcdef",
  y = 4:6
)
str(attributes(a))
```

    ## List of 2
    ##  $ x: chr "abcdef"
    ##  $ y: int [1:3] 4 5 6

Only two attributes are maintained: **names** and **dim**

Three ways to create names are:

``` r
# When creating it: 
x <- c(a = 1, b = 2, c = 3)

# By assigning a character vector to names()
x <- 1:3
names(x) <- c("a", "b", "c")

# Inline, with setNames():
x <- setNames(1:3, c("a", "b", "c"))
```

We can remove names with `x <- unname(x)` or with `names(x) <- NULL`.

``` r
str(1:3)                   # 1d vector
```

    ##  int [1:3] 1 2 3

``` r
str(matrix(1:3, ncol = 1)) # column vector
```

    ##  int [1:3, 1] 1 2 3

``` r
str(matrix(1:3, nrow = 1)) # row vector
```

    ##  int [1, 1:3] 1 2 3

``` r
str(array(1:3, 3))         # "array" vector
```

    ##  int [1:3(1d)] 1 2 3

## 3.4 S3 atomic vectors

> One of the most important vector attributes is class, which underlies
> the S3 object system. Having a class attribute turns an object into an
> S3 object, which means it will behave differently from a regular
> vector when passed to a generic function. Every S3 object is built on
> top of a base type, and often stores additional information in other
> attributes.

There are four important S3 atomic vectors in R: factor vectors, date
vectors, date-time vectors, and duration (difftime) vectors.

Factor vectors must only contain a set of predefined values and they are
built from integer vectors (with class and levels).

``` r
x <- factor(c("a", "b", "b", "a"))
x
```

    ## [1] a b b a
    ## Levels: a b

``` r
typeof(x)
```

    ## [1] "integer"

``` r
attributes(x)
```

    ## $levels
    ## [1] "a" "b"
    ## 
    ## $class
    ## [1] "factor"

``` r
sex_char <- c("m", "m", "m")
sex_factor <- factor(sex_char, levels = c("m", "f"))

table(sex_char)
```

    ## sex_char
    ## m 
    ## 3

``` r
table(sex_factor)
```

    ## sex_factor
    ## m f 
    ## 3 0

There is a subset of factors called ordered factors, in which the levels
of the factor have an inherent ordering. Low, medium, and high, for
example.

``` r
grade <- ordered(c("b", "b", "a", "c"), levels = c("c", "b", "a"))
grade
```

    ## [1] b b a c
    ## Levels: c < b < a

Date vectors derive from double vectors and they have an attribute class
of date.

``` r
today <- Sys.Date()

typeof(today)
```

    ## [1] "double"

``` r
attributes(today)
```

    ## $class
    ## [1] "Date"

``` r
date <- as.Date("1980-02-01")
unclass(date)
```

    ## [1] 3683

When we unclass a date, it provides the number of days since 1970-01-01.

A duration vector is built on a double, with a `unit` attribute.

``` r
one_week_1 <- as.difftime(1, units = "weeks")
one_week_1
```

    ## Time difference of 1 weeks

``` r
typeof(one_week_1)
```

    ## [1] "double"

``` r
attributes(one_week_1)
```

    ## $class
    ## [1] "difftime"
    ## 
    ## $units
    ## [1] "weeks"

``` r
one_week_2 <- as.difftime(7, units = "days")
one_week_2
```

    ## Time difference of 7 days

``` r
typeof(one_week_2)
```

    ## [1] "double"

``` r
attributes(one_week_2)
```

    ## $class
    ## [1] "difftime"
    ## 
    ## $units
    ## [1] "days"

## 3.5 Lists

Lists are complex because elements can be of any type (*technically*
elements of a list are the same type because they are references).

``` r
l1 <- list(
  1:3, 
  "a", 
  c(TRUE, FALSE, TRUE), 
  c(2.3, 5.9)
)

typeof(l1)
```

    ## [1] "list"

``` r
str(l1)
```

    ## List of 4
    ##  $ : int [1:3] 1 2 3
    ##  $ : chr "a"
    ##  $ : logi [1:3] TRUE FALSE TRUE
    ##  $ : num [1:2] 2.3 5.9

Lists are often smaller than we think they are.

``` r
lobstr::obj_size(mtcars)
```

    ## 7,208 B

``` r
l2 <- list(mtcars, mtcars, mtcars, mtcars)
lobstr::obj_size(l2)
```

    ## 7,288 B

Lists can contain other lists:

``` r
l3 <- list(list(list(1)))
str(l3)
```

    ## List of 1
    ##  $ :List of 1
    ##   ..$ :List of 1
    ##   .. ..$ : num 1

We can use dimension to create list-matrices and list-arrays

``` r
l <- list(1:3, "a", TRUE, 1.0)
dim(l) <- c(2, 2)
l
```

    ##      [,1]      [,2]
    ## [1,] Integer,3 TRUE
    ## [2,] "a"       1

``` r
l[[1, 1]]
```

    ## [1] 1 2 3

## 3.6 Data frames and tibbles

Data frames are a named list vector with attributes `names`, `row.names`
and a class of `data.frame`.

``` r
df1 <- data.frame(x = 1:3, y = letters[1:3])
typeof(df1)
```

    ## [1] "list"

``` r
attributes(df1)
```

    ## $names
    ## [1] "x" "y"
    ## 
    ## $class
    ## [1] "data.frame"
    ## 
    ## $row.names
    ## [1] 1 2 3

Each column of a data frame is restricted to be the same length.

A Tibble is a modern reimagination of the data frame.

> A concise, and fun, way to summarise the main differences is that
> tibbles are lazy and surly: they do less and complain more.

``` r
library(tibble)

df2 <- tibble(x = 1:3, y = letters[1:3])
typeof(df2)
```

    ## [1] "list"

``` r
attributes(df2)
```

    ## $names
    ## [1] "x" "y"
    ## 
    ## $row.names
    ## [1] 1 2 3
    ## 
    ## $class
    ## [1] "tbl_df"     "tbl"        "data.frame"

``` r
df <- data.frame(
  x = 1:3, 
  y = c("a", "b", "c")
)
str(df)
```

    ## 'data.frame':    3 obs. of  2 variables:
    ##  $ x: int  1 2 3
    ##  $ y: Factor w/ 3 levels "a","b","c": 1 2 3

``` r
df1 <- data.frame(
  x = 1:3,
  y = c("a", "b", "c"),
  stringsAsFactors = FALSE
)
str(df1)
```

    ## 'data.frame':    3 obs. of  2 variables:
    ##  $ x: int  1 2 3
    ##  $ y: chr  "a" "b" "c"

Tibbles will never coerce their input.

``` r
df2 <- tibble(
  x = 1:3, 
  y = c("a", "b", "c")
)
str(df2)
```

    ## tibble [3 × 2] (S3: tbl_df/tbl/data.frame)
    ##  $ x: int [1:3] 1 2 3
    ##  $ y: chr [1:3] "a" "b" "c"

Data frames will change the name of columns that are non-syntactic,
tibbles do not.

``` r
names(data.frame(`1` = 1))
```

    ## [1] "X1"

``` r
names(tibble(`1` = 1))
```

    ## [1] "1"

If an element of a data frame or tibble has a shorter length, it will be
recycled. Data frames recycle for any shorter length variable, but
tibbles will only recyle if a the shorter variable is of length 1.

``` r
data.frame(x = 1:4, y = 1:2)
```

    ##   x y
    ## 1 1 1
    ## 2 2 2
    ## 3 3 1
    ## 4 4 2

``` r
data.frame(x = 1:4, y = 1:3)
```

    ## Error in data.frame(x = 1:4, y = 1:3): arguments imply differing number of rows: 4, 3

``` r
tibble(x = 1:4, y = 1)
```

    ## # A tibble: 4 x 2
    ##       x     y
    ##   <int> <dbl>
    ## 1     1     1
    ## 2     2     1
    ## 3     3     1
    ## 4     4     1

``` r
tibble(x = 1:4, y = 1:2)
```

    ## Error: Tibble columns must have compatible sizes.
    ## * Size 4: Existing data.
    ## * Size 2: Column `y`.
    ## ℹ Only values of size one are recycled.

One final difference is that tibbles allow the user to create new
variables using newly created variables, for example:

``` r
tibble(
  x = 1:3,
  y = x * 2 # x was just created
)
```

    ## # A tibble: 3 x 2
    ##       x     y
    ##   <int> <dbl>
    ## 1     1     2
    ## 2     2     4
    ## 3     3     6

We can have row names.

``` r
df3 <- data.frame(
  age = c(35, 27, 18),
  hair = c("blond", "brown", "black"),
  row.names = c("Bob", "Susan", "Sam")
)
df3
```

    ##       age  hair
    ## Bob    35 blond
    ## Susan  27 brown
    ## Sam    18 black

``` r
rownames(df3)
```

    ## [1] "Bob"   "Susan" "Sam"

``` r
df3["Bob", ]
```

    ##     age  hair
    ## Bob  35 blond

Matrices are transposeable, data frames are not. It is best to avoid row
names, they are an added complication.

``` r
df3[c(1, 1, 1), ]
```

    ##       age  hair
    ## Bob    35 blond
    ## Bob.1  35 blond
    ## Bob.2  35 blond

So, tibbles do not support row names.

``` r
as_tibble(df3, rownames = "name")
```

    ## # A tibble: 3 x 3
    ##   name    age hair 
    ##   <chr> <dbl> <fct>
    ## 1 Bob      35 blond
    ## 2 Susan    27 brown
    ## 3 Sam      18 black

``` r
dplyr::starwars
```

    ## # A tibble: 87 x 14
    ##    name    height  mass hair_color  skin_color eye_color birth_year sex   gender
    ##    <chr>    <int> <dbl> <chr>       <chr>      <chr>          <dbl> <chr> <chr> 
    ##  1 Luke S…    172    77 blond       fair       blue            19   male  mascu…
    ##  2 C-3PO      167    75 <NA>        gold       yellow         112   none  mascu…
    ##  3 R2-D2       96    32 <NA>        white, bl… red             33   none  mascu…
    ##  4 Darth …    202   136 none        white      yellow          41.9 male  mascu…
    ##  5 Leia O…    150    49 brown       light      brown           19   fema… femin…
    ##  6 Owen L…    178   120 brown, grey light      blue            52   male  mascu…
    ##  7 Beru W…    165    75 brown       light      blue            47   fema… femin…
    ##  8 R5-D4       97    32 <NA>        white, red red             NA   none  mascu…
    ##  9 Biggs …    183    84 black       light      brown           24   male  mascu…
    ## 10 Obi-Wa…    182    77 auburn, wh… fair       blue-gray       57   male  mascu…
    ## # … with 77 more rows, and 5 more variables: homeworld <chr>, species <chr>,
    ## #   films <list>, vehicles <list>, starships <list>

I’ve definitely run into this issue before: \> When you subset columns
with df\[, vars\], you will get a vector if vars selects one variable,
otherwise you’ll get a data frame. This is a frequent source of bugs
when using \[ in a function, unless you always remember to use df\[,
vars, drop = FALSE\].

Another inconvenient subsetting behavior: \> When you attempt to extract
a single column with
df\(x and there is no column x, a data frame will instead select any variable that starts with x. If no variable starts with x, df\)x
will return NULL. This makes it easy to select the wrong variable or to
select a variable that doesn’t exist.

Tibbles modify this by only returning tibbles and they don’t do partial
matching.

``` r
df1 <- data.frame(xyz = "a")
df2 <- tibble(xyz = "a")

str(df1$x)
```

    ##  Factor w/ 1 level "a": 1

``` r
str(df2$x)
```

    ## Warning: Unknown or uninitialised column: `x`.

    ##  NULL

Data frames can contain lists.

``` r
df <- data.frame(x = 1:3)
df$y <- list(1:2, 1:3, 1:4)

data.frame(
  x = 1:3, 
  y = I(list(1:2, 1:3, 1:4))
)
```

    ##   x          y
    ## 1 1       1, 2
    ## 2 2    1, 2, 3
    ## 3 3 1, 2, 3, 4

Tibbles make this a little easier:

``` r
tibble(
  x = 1:3, 
  y = list(1:2, 1:3, 1:4)
)
```

    ## # A tibble: 3 x 2
    ##       x y        
    ##   <int> <list>   
    ## 1     1 <int [2]>
    ## 2     2 <int [3]>
    ## 3     3 <int [4]>

It is possible to have a matrix or array as column of data.

``` r
dfm <- data.frame(
  x = 1:3 * 10
)
dfm$y <- matrix(1:9, nrow = 3)
dfm$z <- data.frame(a = 3:1, b = letters[1:3], stringsAsFactors = FALSE)

str(dfm)
```

    ## 'data.frame':    3 obs. of  3 variables:
    ##  $ x: num  10 20 30
    ##  $ y: int [1:3, 1:3] 1 2 3 4 5 6 7 8 9
    ##  $ z:'data.frame':   3 obs. of  2 variables:
    ##   ..$ a: int  3 2 1
    ##   ..$ b: chr  "a" "b" "c"

``` r
dfm[1, ]
```

    ##    x y.1 y.2 y.3 z.a z.b
    ## 1 10   1   4   7   3   a

## 3.7 `NULL`

> NULL is special because it has a unique type, is always length zero,
> and can’t have any attributes:

``` r
typeof(NULL)
```

    ## [1] "NULL"

``` r
length(NULL)
```

    ## [1] 0

``` r
x <- NULL
attr(x, "y") <- 1
```

    ## Error in attr(x, "y") <- 1: attempt to set an attribute on NULL

``` r
is.null(NULL)
```

    ## [1] TRUE
