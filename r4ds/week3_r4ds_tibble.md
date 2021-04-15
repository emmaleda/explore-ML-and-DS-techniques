Week 3: Tibbles with tibble
================
Emma Grossman
4/10/2021

tibbles are data frames that tweak outdated behaviors of data frames.

## Creating Tibbles

Coerce a data frame into a tibble:

``` r
as_tibble(iris)
```

    ## # A tibble: 150 x 5
    ##    Sepal.Length Sepal.Width Petal.Length Petal.Width Species
    ##           <dbl>       <dbl>        <dbl>       <dbl> <fct>  
    ##  1          5.1         3.5          1.4         0.2 setosa 
    ##  2          4.9         3            1.4         0.2 setosa 
    ##  3          4.7         3.2          1.3         0.2 setosa 
    ##  4          4.6         3.1          1.5         0.2 setosa 
    ##  5          5           3.6          1.4         0.2 setosa 
    ##  6          5.4         3.9          1.7         0.4 setosa 
    ##  7          4.6         3.4          1.4         0.3 setosa 
    ##  8          5           3.4          1.5         0.2 setosa 
    ##  9          4.4         2.9          1.4         0.2 setosa 
    ## 10          4.9         3.1          1.5         0.1 setosa 
    ## # … with 140 more rows

Create a new tibble from individual vectors:

``` r
tibble(
  x = 1:5,
  y = 1,
  z = x^2 + y
)
```

    ## # A tibble: 5 x 3
    ##       x     y     z
    ##   <int> <dbl> <dbl>
    ## 1     1     1     2
    ## 2     2     1     5
    ## 3     3     1    10
    ## 4     4     1    17
    ## 5     5     1    26

We can see that tibble recycles inputs of length one, like y in the
example above.

tibble never changes the type of input (converts strings to factors),
changes the names of variables, or creates row names. tibble can have
unconventional or *nonsyntactic* variable names, however:

``` r
tibble(
  `:)` = "smile",
  ` ` = "space",
  `2000` = "number"
)
```

    ## # A tibble: 1 x 3
    ##   `:)`  ` `   `2000`
    ##   <chr> <chr> <chr> 
    ## 1 smile space number

We can also create a tibble with `tribble()`, which stands for
transposed tibble; it is customized for data entry in code.

``` r
tribble(
  ~x, ~y, ~z, # column names have a tilde in front
  #--/--/----
  "a", 2, 3.6,
  "b", 1, 8.5
)
```

    ## # A tibble: 2 x 3
    ##   x         y     z
    ##   <chr> <dbl> <dbl>
    ## 1 a         2   3.6
    ## 2 b         1   8.5

## Tibbles Versus data.frame

The main differences are printing and subsetting.

### Printing

When printing a tibble we only see the first 10 rows and all columns
that fit on the screen. It also reports column type.

``` r
tibble(
  a = lubridate::now()   + runif(1e3) * 86400,
  b = lubridate::today() + runif(1e3) * 30,
  c = 1:1e3,
  d = runif(1e3),
  e = sample(letters, 1e3, replace = TRUE)
)
```

    ## # A tibble: 1,000 x 5
    ##    a                   b              c       d e    
    ##    <dttm>              <date>     <int>   <dbl> <chr>
    ##  1 2021-04-14 18:13:16 2021-04-20     1 0.877   c    
    ##  2 2021-04-15 00:02:10 2021-05-13     2 0.257   q    
    ##  3 2021-04-15 11:36:28 2021-04-27     3 0.548   s    
    ##  4 2021-04-14 17:21:22 2021-05-08     4 0.346   t    
    ##  5 2021-04-14 16:26:44 2021-05-05     5 0.946   g    
    ##  6 2021-04-15 07:13:27 2021-05-03     6 0.604   c    
    ##  7 2021-04-14 15:35:27 2021-04-30     7 0.678   g    
    ##  8 2021-04-14 19:54:59 2021-04-18     8 0.773   s    
    ##  9 2021-04-14 20:07:40 2021-05-11     9 0.00859 l    
    ## 10 2021-04-15 10:23:17 2021-05-08    10 0.825   v    
    ## # … with 990 more rows

tibbles are designed so we don’t overwhelm our console.

We can change the number of rows and `width` of the display:

``` r
nycflights13::flights %>%
  print(n = 11, width = Inf)
```

    ## # A tibble: 336,776 x 19
    ##     year month   day dep_time sched_dep_time dep_delay arr_time sched_arr_time
    ##    <int> <int> <int>    <int>          <int>     <dbl>    <int>          <int>
    ##  1  2013     1     1      517            515         2      830            819
    ##  2  2013     1     1      533            529         4      850            830
    ##  3  2013     1     1      542            540         2      923            850
    ##  4  2013     1     1      544            545        -1     1004           1022
    ##  5  2013     1     1      554            600        -6      812            837
    ##  6  2013     1     1      554            558        -4      740            728
    ##  7  2013     1     1      555            600        -5      913            854
    ##  8  2013     1     1      557            600        -3      709            723
    ##  9  2013     1     1      557            600        -3      838            846
    ## 10  2013     1     1      558            600        -2      753            745
    ## 11  2013     1     1      558            600        -2      849            851
    ##    arr_delay carrier flight tailnum origin dest  air_time distance  hour minute
    ##        <dbl> <chr>    <int> <chr>   <chr>  <chr>    <dbl>    <dbl> <dbl>  <dbl>
    ##  1        11 UA        1545 N14228  EWR    IAH        227     1400     5     15
    ##  2        20 UA        1714 N24211  LGA    IAH        227     1416     5     29
    ##  3        33 AA        1141 N619AA  JFK    MIA        160     1089     5     40
    ##  4       -18 B6         725 N804JB  JFK    BQN        183     1576     5     45
    ##  5       -25 DL         461 N668DN  LGA    ATL        116      762     6      0
    ##  6        12 UA        1696 N39463  EWR    ORD        150      719     5     58
    ##  7        19 B6         507 N516JB  EWR    FLL        158     1065     6      0
    ##  8       -14 EV        5708 N829AS  LGA    IAD         53      229     6      0
    ##  9        -8 B6          79 N593JB  JFK    MCO        140      944     6      0
    ## 10         8 AA         301 N3ALAA  LGA    ORD        138      733     6      0
    ## 11        -2 B6          49 N793JB  JFK    PBI        149     1028     6      0
    ##    time_hour          
    ##    <dttm>             
    ##  1 2013-01-01 05:00:00
    ##  2 2013-01-01 05:00:00
    ##  3 2013-01-01 05:00:00
    ##  4 2013-01-01 05:00:00
    ##  5 2013-01-01 06:00:00
    ##  6 2013-01-01 05:00:00
    ##  7 2013-01-01 06:00:00
    ##  8 2013-01-01 06:00:00
    ##  9 2013-01-01 06:00:00
    ## 10 2013-01-01 06:00:00
    ## 11 2013-01-01 06:00:00
    ## # … with 336,765 more rows

We can use base R and the pipe to view a dataset in a new tab. This is
helpful after a long string of manipulations.

``` r
# nycflights13::flights %>%
#   View()
```

### Subsetting

``` r
df <- tibble(
  x = runif(5),
  y = rnorm(5)
)

# Extract by name
df$x
```

    ## [1] 0.3147065 0.2110258 0.7360280 0.4978754 0.1955717

``` r
df[["x"]]
```

    ## [1] 0.3147065 0.2110258 0.7360280 0.4978754 0.1955717

``` r
# Extract by position
df[[1]]
```

    ## [1] 0.3147065 0.2110258 0.7360280 0.4978754 0.1955717

If we want to use the pipe, there is a special placeholder.

``` r
df %>% .$x
```

    ## [1] 0.3147065 0.2110258 0.7360280 0.4978754 0.1955717

``` r
df %>% .[["x"]]
```

    ## [1] 0.3147065 0.2110258 0.7360280 0.4978754 0.1955717

We can also convert back to a data.frame with `as.data.frame()`.

``` r
class(as.data.frame(df))
```

    ## [1] "data.frame"

``` r
# ?enframe
enframe(1:3)
```

    ## # A tibble: 3 x 2
    ##    name value
    ##   <int> <int>
    ## 1     1     1
    ## 2     2     2
    ## 3     3     3

``` r
enframe(list(one = 1, two = 2:3, three = 4:6))
```

    ## # A tibble: 3 x 2
    ##   name  value    
    ##   <chr> <list>   
    ## 1 one   <dbl [1]>
    ## 2 two   <int [2]>
    ## 3 three <int [3]>
