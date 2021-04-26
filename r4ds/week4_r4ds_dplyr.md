Relational Data with dplyr
================
Emma Grossman
4/26/2021

# Introduction

In the real world, it is rare that data analysis only involves a single
table of data. Multiple tables are called relational data, because the
relationships between tables are as important as the individual
datasets.

There are several ways to join data from multiple tables:

  - *mutating joins* - add new variables to a df from matching
    observations in another
  - *filtering joins* - filter observations from one df based on whether
    they match an observation in the other table
  - *set operations* - treat observations as if they were set elements

<!-- end list -->

``` r
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.0 ──

    ## ✓ ggplot2 3.3.3     ✓ purrr   0.3.4
    ## ✓ tibble  3.1.0     ✓ dplyr   1.0.5
    ## ✓ tidyr   1.1.3     ✓ stringr 1.4.0
    ## ✓ readr   1.4.0     ✓ forcats 0.5.1

    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library(nycflights13)
```

## nycflights13

There are four tibbles related to flights.

``` r
head(airlines)
```

    ## # A tibble: 6 x 2
    ##   carrier name                    
    ##   <chr>   <chr>                   
    ## 1 9E      Endeavor Air Inc.       
    ## 2 AA      American Airlines Inc.  
    ## 3 AS      Alaska Airlines Inc.    
    ## 4 B6      JetBlue Airways         
    ## 5 DL      Delta Air Lines Inc.    
    ## 6 EV      ExpressJet Airlines Inc.

``` r
head(airports)
```

    ## # A tibble: 6 x 8
    ##   faa   name                          lat   lon   alt    tz dst   tzone         
    ##   <chr> <chr>                       <dbl> <dbl> <dbl> <dbl> <chr> <chr>         
    ## 1 04G   Lansdowne Airport            41.1 -80.6  1044    -5 A     America/New_Y…
    ## 2 06A   Moton Field Municipal Airp…  32.5 -85.7   264    -6 A     America/Chica…
    ## 3 06C   Schaumburg Regional          42.0 -88.1   801    -6 A     America/Chica…
    ## 4 06N   Randall Airport              41.4 -74.4   523    -5 A     America/New_Y…
    ## 5 09J   Jekyll Island Airport        31.1 -81.4    11    -5 A     America/New_Y…
    ## 6 0A9   Elizabethton Municipal Air…  36.4 -82.2  1593    -5 A     America/New_Y…

``` r
head(planes)
```

    ## # A tibble: 6 x 9
    ##   tailnum  year type           manufacturer   model  engines seats speed engine 
    ##   <chr>   <int> <chr>          <chr>          <chr>    <int> <int> <int> <chr>  
    ## 1 N10156   2004 Fixed wing mu… EMBRAER        EMB-1…       2    55    NA Turbo-…
    ## 2 N102UW   1998 Fixed wing mu… AIRBUS INDUST… A320-…       2   182    NA Turbo-…
    ## 3 N103US   1999 Fixed wing mu… AIRBUS INDUST… A320-…       2   182    NA Turbo-…
    ## 4 N104UW   1999 Fixed wing mu… AIRBUS INDUST… A320-…       2   182    NA Turbo-…
    ## 5 N10575   2002 Fixed wing mu… EMBRAER        EMB-1…       2    55    NA Turbo-…
    ## 6 N105UW   1999 Fixed wing mu… AIRBUS INDUST… A320-…       2   182    NA Turbo-…

``` r
head(weather)
```

    ## # A tibble: 6 x 15
    ##   origin  year month   day  hour  temp  dewp humid wind_dir wind_speed wind_gust
    ##   <chr>  <int> <int> <int> <int> <dbl> <dbl> <dbl>    <dbl>      <dbl>     <dbl>
    ## 1 EWR     2013     1     1     1  39.0  26.1  59.4      270      10.4         NA
    ## 2 EWR     2013     1     1     2  39.0  27.0  61.6      250       8.06        NA
    ## 3 EWR     2013     1     1     3  39.0  28.0  64.4      240      11.5         NA
    ## 4 EWR     2013     1     1     4  39.9  28.0  62.2      250      12.7         NA
    ## 5 EWR     2013     1     1     5  39.0  28.0  64.4      260      12.7         NA
    ## 6 EWR     2013     1     1     6  37.9  28.0  67.2      240      11.5         NA
    ## # … with 4 more variables: precip <dbl>, pressure <dbl>, visib <dbl>,
    ## #   time_hour <dttm>

`flights` and `planes` have the variables `tailnum` in common; `flights`
and `airlines` have `carrier` in common; `flights` and `airports` have
`origin` and `dest` in common; and `flights` and `weather` have `origin`
in common.

### Keys

*Keys* are variables used to connect pairs of tables. A *primary key*
uniquely identifies an observation in a table, like an ID column. A
*foreign key* uniquely identifies an observation in another table.

One way to ensure that a primary key indeed uniquely identifies each
observation is by using `count()`.

``` r
planes %>%
  count(tailnum) %>%
  filter(n > 1)
```

    ## # A tibble: 0 x 2
    ## # … with 2 variables: tailnum <chr>, n <int>

Since there are no `tailnum` assigned to multiple planes, it is indeed
unique.

``` r
weather %>%
  count(year, month, day, hour, origin) %>%
  filter(n > 1)
```

    ## # A tibble: 3 x 6
    ##    year month   day  hour origin     n
    ##   <int> <int> <int> <int> <chr>  <int>
    ## 1  2013    11     3     1 EWR        2
    ## 2  2013    11     3     1 JFK        2
    ## 3  2013    11     3     1 LGA        2

Sometimes there is no primary key. No combination of the variables
uniquely identifies each observation.

``` r
flights %>%
  count(year, month, day, flight) %>%
  filter(n > 1)
```

What if we include `tailnum`?

``` r
flights %>%
  count(year, month, day, tailnum) %>%
  filter(n > 1)
```

    ## # A tibble: 64,928 x 5
    ##     year month   day tailnum     n
    ##    <int> <int> <int> <chr>   <int>
    ##  1  2013     1     1 N0EGMQ      2
    ##  2  2013     1     1 N11189      2
    ##  3  2013     1     1 N11536      2
    ##  4  2013     1     1 N11544      3
    ##  5  2013     1     1 N11551      2
    ##  6  2013     1     1 N12540      2
    ##  7  2013     1     1 N12567      2
    ##  8  2013     1     1 N13123      2
    ##  9  2013     1     1 N13538      3
    ## 10  2013     1     1 N13566      3
    ## # … with 64,918 more rows

If a dataset lacks primary keys, we can create a *surrogate key* by
adding an ID column. A primary key and a foreign key form a *relation*.
Relations are typically one-to-many.

### Mutating Joins

Let’s create a smaller dataset.

``` r
flights2 <- flights %>%
  select(year:day, hour, origin, dest, tailnum, carrier)
head(flights2)
```

    ## # A tibble: 6 x 8
    ##    year month   day  hour origin dest  tailnum carrier
    ##   <int> <int> <int> <dbl> <chr>  <chr> <chr>   <chr>  
    ## 1  2013     1     1     5 EWR    IAH   N14228  UA     
    ## 2  2013     1     1     5 LGA    IAH   N24211  UA     
    ## 3  2013     1     1     5 JFK    MIA   N619AA  AA     
    ## 4  2013     1     1     5 JFK    BQN   N804JB  B6     
    ## 5  2013     1     1     6 LGA    ATL   N668DN  DL     
    ## 6  2013     1     1     5 EWR    ORD   N39463  UA

If we want to add the airline names to `flights2`. We can do this with
`left_join()`.

``` r
flights2 %>%
  select(-origin, -dest)%>%
  left_join(airlines, by = "carrier")
```

    ## # A tibble: 336,776 x 7
    ##     year month   day  hour tailnum carrier name                    
    ##    <int> <int> <int> <dbl> <chr>   <chr>   <chr>                   
    ##  1  2013     1     1     5 N14228  UA      United Air Lines Inc.   
    ##  2  2013     1     1     5 N24211  UA      United Air Lines Inc.   
    ##  3  2013     1     1     5 N619AA  AA      American Airlines Inc.  
    ##  4  2013     1     1     5 N804JB  B6      JetBlue Airways         
    ##  5  2013     1     1     6 N668DN  DL      Delta Air Lines Inc.    
    ##  6  2013     1     1     5 N39463  UA      United Air Lines Inc.   
    ##  7  2013     1     1     6 N516JB  B6      JetBlue Airways         
    ##  8  2013     1     1     6 N829AS  EV      ExpressJet Airlines Inc.
    ##  9  2013     1     1     6 N593JB  B6      JetBlue Airways         
    ## 10  2013     1     1     6 N3ALAA  AA      American Airlines Inc.  
    ## # … with 336,766 more rows

We could accomplish the same thing using `mutate`.

``` r
flights2 %>%
  select(-origin, -dest) %>%
  mutate(name = airlines$name[match(carrier, airlines$carrier)])
```

    ## # A tibble: 336,776 x 7
    ##     year month   day  hour tailnum carrier name                    
    ##    <int> <int> <int> <dbl> <chr>   <chr>   <chr>                   
    ##  1  2013     1     1     5 N14228  UA      United Air Lines Inc.   
    ##  2  2013     1     1     5 N24211  UA      United Air Lines Inc.   
    ##  3  2013     1     1     5 N619AA  AA      American Airlines Inc.  
    ##  4  2013     1     1     5 N804JB  B6      JetBlue Airways         
    ##  5  2013     1     1     6 N668DN  DL      Delta Air Lines Inc.    
    ##  6  2013     1     1     5 N39463  UA      United Air Lines Inc.   
    ##  7  2013     1     1     6 N516JB  B6      JetBlue Airways         
    ##  8  2013     1     1     6 N829AS  EV      ExpressJet Airlines Inc.
    ##  9  2013     1     1     6 N593JB  B6      JetBlue Airways         
    ## 10  2013     1     1     6 N3ALAA  AA      American Airlines Inc.  
    ## # … with 336,766 more rows

#### Inner Join

For an inner join, unmatched rows are not included in the result.

#### Outer Join

Keeps observations that appear in at least one of the tables. **Left
join** keeps all observations in x, **right join** keeps all
observations in y, and a **full join** keeps all observations in x and
y.

Left join is the most common and should be the default.

### Duplicate Keys

What happens if we our key isn’t unique?

``` r
x <- tribble(
  ~key, ~val_x,
     1,   "x1",
     2,   "x2",
     2,   "x3",
     1,   "x4"
)

y <- tribble(
  ~key, ~val_x,
     1,   "y1",
     2,   "y2",
)

left_join(x, y, by = "key")
```

    ## # A tibble: 4 x 3
    ##     key val_x.x val_x.y
    ##   <dbl> <chr>   <chr>  
    ## 1     1 x1      y1     
    ## 2     2 x2      y2     
    ## 3     2 x3      y2     
    ## 4     1 x4      y1

With a duplicate key, all possible combinations are returned.

### Defining the Key Columns

  - if we use `by = NULL`, all variables that appear in both tables are
    used to join the tables, also called a *natural* join

<!-- end list -->

``` r
flights2 %>%
  left_join(weather)
```

    ## Joining, by = c("year", "month", "day", "hour", "origin")

    ## # A tibble: 336,776 x 18
    ##     year month   day  hour origin dest  tailnum carrier  temp  dewp humid
    ##    <int> <int> <int> <dbl> <chr>  <chr> <chr>   <chr>   <dbl> <dbl> <dbl>
    ##  1  2013     1     1     5 EWR    IAH   N14228  UA       39.0  28.0  64.4
    ##  2  2013     1     1     5 LGA    IAH   N24211  UA       39.9  25.0  54.8
    ##  3  2013     1     1     5 JFK    MIA   N619AA  AA       39.0  27.0  61.6
    ##  4  2013     1     1     5 JFK    BQN   N804JB  B6       39.0  27.0  61.6
    ##  5  2013     1     1     6 LGA    ATL   N668DN  DL       39.9  25.0  54.8
    ##  6  2013     1     1     5 EWR    ORD   N39463  UA       39.0  28.0  64.4
    ##  7  2013     1     1     6 EWR    FLL   N516JB  B6       37.9  28.0  67.2
    ##  8  2013     1     1     6 LGA    IAD   N829AS  EV       39.9  25.0  54.8
    ##  9  2013     1     1     6 JFK    MCO   N593JB  B6       37.9  27.0  64.3
    ## 10  2013     1     1     6 LGA    ORD   N3ALAA  AA       39.9  25.0  54.8
    ## # … with 336,766 more rows, and 7 more variables: wind_dir <dbl>,
    ## #   wind_speed <dbl>, wind_gust <dbl>, precip <dbl>, pressure <dbl>,
    ## #   visib <dbl>, time_hour <dttm>

  - join by a named character vector: `by = c("a" = "b")`: variable `a`
    in table x matches to variable `b` in table y. The variables from x
    will be used in the output.

<!-- end list -->

``` r
flights2 %>%
  left_join(airports, c("dest" = "faa"))
```

    ## # A tibble: 336,776 x 15
    ##     year month   day  hour origin dest  tailnum carrier name     lat   lon   alt
    ##    <int> <int> <int> <dbl> <chr>  <chr> <chr>   <chr>   <chr>  <dbl> <dbl> <dbl>
    ##  1  2013     1     1     5 EWR    IAH   N14228  UA      Georg…  30.0 -95.3    97
    ##  2  2013     1     1     5 LGA    IAH   N24211  UA      Georg…  30.0 -95.3    97
    ##  3  2013     1     1     5 JFK    MIA   N619AA  AA      Miami…  25.8 -80.3     8
    ##  4  2013     1     1     5 JFK    BQN   N804JB  B6      <NA>    NA    NA      NA
    ##  5  2013     1     1     6 LGA    ATL   N668DN  DL      Harts…  33.6 -84.4  1026
    ##  6  2013     1     1     5 EWR    ORD   N39463  UA      Chica…  42.0 -87.9   668
    ##  7  2013     1     1     6 EWR    FLL   N516JB  B6      Fort …  26.1 -80.2     9
    ##  8  2013     1     1     6 LGA    IAD   N829AS  EV      Washi…  38.9 -77.5   313
    ##  9  2013     1     1     6 JFK    MCO   N593JB  B6      Orlan…  28.4 -81.3    96
    ## 10  2013     1     1     6 LGA    ORD   N3ALAA  AA      Chica…  42.0 -87.9   668
    ## # … with 336,766 more rows, and 3 more variables: tz <dbl>, dst <chr>,
    ## #   tzone <chr>

``` r
flights2 %>%
  left_join(airports, c("origin" = "faa"))
```

    ## # A tibble: 336,776 x 15
    ##     year month   day  hour origin dest  tailnum carrier name     lat   lon   alt
    ##    <int> <int> <int> <dbl> <chr>  <chr> <chr>   <chr>   <chr>  <dbl> <dbl> <dbl>
    ##  1  2013     1     1     5 EWR    IAH   N14228  UA      Newar…  40.7 -74.2    18
    ##  2  2013     1     1     5 LGA    IAH   N24211  UA      La Gu…  40.8 -73.9    22
    ##  3  2013     1     1     5 JFK    MIA   N619AA  AA      John …  40.6 -73.8    13
    ##  4  2013     1     1     5 JFK    BQN   N804JB  B6      John …  40.6 -73.8    13
    ##  5  2013     1     1     6 LGA    ATL   N668DN  DL      La Gu…  40.8 -73.9    22
    ##  6  2013     1     1     5 EWR    ORD   N39463  UA      Newar…  40.7 -74.2    18
    ##  7  2013     1     1     6 EWR    FLL   N516JB  B6      Newar…  40.7 -74.2    18
    ##  8  2013     1     1     6 LGA    IAD   N829AS  EV      La Gu…  40.8 -73.9    22
    ##  9  2013     1     1     6 JFK    MCO   N593JB  B6      John …  40.6 -73.8    13
    ## 10  2013     1     1     6 LGA    ORD   N3ALAA  AA      La Gu…  40.8 -73.9    22
    ## # … with 336,766 more rows, and 3 more variables: tz <dbl>, dst <chr>,
    ## #   tzone <chr>

### Filtering Joins

Rather than affect variables, filtering joins affect the observations
themselves.

  - **semi\_join(x, y)** *keeps* all observations in x that have a match
    in y
  - **anti\_join(x, y)** *drops* all observations in x that have a match
    in y

<!-- end list -->

``` r
top_dest <- flights %>%
  count(dest, sort = TRUE) %>%
  head(10)
top_dest
```

    ## # A tibble: 10 x 2
    ##    dest      n
    ##    <chr> <int>
    ##  1 ORD   17283
    ##  2 ATL   17215
    ##  3 LAX   16174
    ##  4 BOS   15508
    ##  5 MCO   14082
    ##  6 CLT   14064
    ##  7 SFO   13331
    ##  8 FLL   12055
    ##  9 MIA   11728
    ## 10 DCA    9705

We could filter this by hand:

``` r
flights %>%
  filter(dest %in% top_dest$dest)
```

    ## # A tibble: 141,145 x 19
    ##     year month   day dep_time sched_dep_time dep_delay arr_time sched_arr_time
    ##    <int> <int> <int>    <int>          <int>     <dbl>    <int>          <int>
    ##  1  2013     1     1      542            540         2      923            850
    ##  2  2013     1     1      554            600        -6      812            837
    ##  3  2013     1     1      554            558        -4      740            728
    ##  4  2013     1     1      555            600        -5      913            854
    ##  5  2013     1     1      557            600        -3      838            846
    ##  6  2013     1     1      558            600        -2      753            745
    ##  7  2013     1     1      558            600        -2      924            917
    ##  8  2013     1     1      558            600        -2      923            937
    ##  9  2013     1     1      559            559         0      702            706
    ## 10  2013     1     1      600            600         0      851            858
    ## # … with 141,135 more rows, and 11 more variables: arr_delay <dbl>,
    ## #   carrier <chr>, flight <int>, tailnum <chr>, origin <chr>, dest <chr>,
    ## #   air_time <dbl>, distance <dbl>, hour <dbl>, minute <dbl>, time_hour <dttm>

but once we add more variables it is much more difficult to filter by
hand. We can use semi\_join instead.

``` r
flights %>%
  semi_join(top_dest)
```

    ## Joining, by = "dest"

    ## # A tibble: 141,145 x 19
    ##     year month   day dep_time sched_dep_time dep_delay arr_time sched_arr_time
    ##    <int> <int> <int>    <int>          <int>     <dbl>    <int>          <int>
    ##  1  2013     1     1      542            540         2      923            850
    ##  2  2013     1     1      554            600        -6      812            837
    ##  3  2013     1     1      554            558        -4      740            728
    ##  4  2013     1     1      555            600        -5      913            854
    ##  5  2013     1     1      557            600        -3      838            846
    ##  6  2013     1     1      558            600        -2      753            745
    ##  7  2013     1     1      558            600        -2      924            917
    ##  8  2013     1     1      558            600        -2      923            937
    ##  9  2013     1     1      559            559         0      702            706
    ## 10  2013     1     1      600            600         0      851            858
    ## # … with 141,135 more rows, and 11 more variables: arr_delay <dbl>,
    ## #   carrier <chr>, flight <int>, tailnum <chr>, origin <chr>, dest <chr>,
    ## #   air_time <dbl>, distance <dbl>, hour <dbl>, minute <dbl>, time_hour <dttm>

Using anti-joins, we can easily find join mismatches.

``` r
flights %>%
  anti_join(planes, by = "tailnum") %>%
  count(tailnum, sort = TRUE)
```

    ## # A tibble: 722 x 2
    ##    tailnum     n
    ##    <chr>   <int>
    ##  1 <NA>     2512
    ##  2 N725MQ    575
    ##  3 N722MQ    513
    ##  4 N723MQ    507
    ##  5 N713MQ    483
    ##  6 N735MQ    396
    ##  7 N0EGMQ    371
    ##  8 N534MQ    364
    ##  9 N542MQ    363
    ## 10 N531MQ    349
    ## # … with 712 more rows
