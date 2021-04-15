Data Import with readr
================
Emma Grossman
4/11/2021

`readr` turns files into data frames.

  - `read_cvs()`: comma-delimited files
  - `read_csv2()`: semicolon-separated files (common in countries where
    the comma is used as the decimal place)
  - `read_tsv()`:tab-delimited files
  - `read_delim()`: allows the user to specify any delimiter
  - `read_fwf()`: fixed-width files
  - `read_log()`: Apache style log files

We can supply an inline CSV file

``` r
read_csv("a,b,c
         1,2,3
         4,5,6")
```

    ## # A tibble: 2 x 3
    ##       a     b     c
    ##   <dbl> <dbl> <dbl>
    ## 1     1     2     3
    ## 2     4     5     6

`read_csv()` is using the first line of data as column names. We can
change this a couple ways:

``` r
read_csv("The first line of metadata
         The second line of metadata
         x,y,z
         1,2,3", skip = 2)
```

    ## # A tibble: 1 x 3
    ##       x     y     z
    ##   <dbl> <dbl> <dbl>
    ## 1     1     2     3

Or by using comments:

``` r
read_csv("# A comment I want to skip
         x,y,z
         1,2,3", comment = "#")
```

    ## # A tibble: 1 x 3
    ##       x     y     z
    ##   <dbl> <dbl> <dbl>
    ## 1     1     2     3

If the data do not have column names,

``` r
read_csv("1,2,3\n4,5,6", col_names = FALSE)
```

    ## # A tibble: 2 x 3
    ##      X1    X2    X3
    ##   <dbl> <dbl> <dbl>
    ## 1     1     2     3
    ## 2     4     5     6

We can also pass column names in as a vector:

``` r
read_csv("1,2,3\n4,5,6", col_names =c("x","y","z"))
```

    ## # A tibble: 2 x 3
    ##       x     y     z
    ##   <dbl> <dbl> <dbl>
    ## 1     1     2     3
    ## 2     4     5     6

``` r
read_csv("a,b\n1,2,3\n4,5,6")
```

    ## Warning: 2 parsing failures.
    ## row col  expected    actual         file
    ##   1  -- 2 columns 3 columns literal data
    ##   2  -- 2 columns 3 columns literal data

    ## # A tibble: 2 x 2
    ##       a     b
    ##   <dbl> <dbl>
    ## 1     1     2
    ## 2     4     5

``` r
read_csv("a,b\n\"1")
```

    ## Warning: 2 parsing failures.
    ## row col                     expected    actual         file
    ##   1  a  closing quote at end of file           literal data
    ##   1  -- 2 columns                    1 columns literal data

    ## # A tibble: 1 x 2
    ##       a b    
    ##   <dbl> <chr>
    ## 1     1 <NA>

``` r
read_csv("a,b\n1,2\na,b")
```

    ## # A tibble: 2 x 2
    ##   a     b    
    ##   <chr> <chr>
    ## 1 1     2    
    ## 2 a     b

``` r
read_csv("a;b\n1;3")
```

    ## # A tibble: 1 x 1
    ##   `a;b`
    ##   <chr>
    ## 1 1;3

Parsing a Vector

``` r
str(parse_logical(c("TRUE","FALSE","NA")))
```

    ##  logi [1:3] TRUE FALSE NA

``` r
str(parse_integer(c("1","2","3")))
```

    ##  int [1:3] 1 2 3

``` r
str(parse_date(c("2010-01-01","1979-10-14")))
```

    ##  Date[1:2], format: "2010-01-01" "1979-10-14"

These parsers are the building blocks for `readr`

``` r
parse_integer(c("1","241",".","456"), na = ".")
```

    ## [1]   1 241  NA 456

``` r
x <- parse_integer(c("123","345","abc","123.45"))
```

    ## Warning: 2 parsing failures.
    ## row col               expected actual
    ##   3  -- an integer             abc   
    ##   4  -- no trailing characters 123.45

``` r
problems(x)
```

    ## # A tibble: 2 x 4
    ##     row   col expected               actual
    ##   <int> <int> <chr>                  <chr> 
    ## 1     3    NA an integer             abc   
    ## 2     4    NA no trailing characters 123.45

Parsing numbers is tricky because there is different notation in
different countries.

``` r
parse_double("1.23")
```

    ## [1] 1.23

``` r
parse_double("1,23", locale = locale(decimal_mark = ","))
```

    ## [1] 1.23

`parse_number` ignores non-numeric characters:

``` r
parse_number("$100")
```

    ## [1] 100

``` r
parse_number("20%")
```

    ## [1] 20

``` r
parse_number("It cost $123.45")
```

    ## [1] 123.45

``` r
# Used in America
parse_number("$123,456,789")
```

    ## [1] 123456789

``` r
# Used in many parts of Europe
parse_number(
  "123.456.789",
  locale = locale(grouping_mark = ".")
)
```

    ## [1] 123456789

``` r
# Used in Switzerland
parse_number(
  "123'456'789",
  locale = locale(grouping_mark = "'")
)
```

    ## [1] 123456789

Strings can also be difficult because there are different encodings. The
most common now is UTF-8.

``` r
charToRaw("Hadley")
```

    ## [1] 48 61 64 6c 65 79

``` r
charToRaw("Emma")
```

    ## [1] 45 6d 6d 61

The remainder of subjects in this chapter are things that I know
already, so I’m just going to copy the code without much explanation.

### Factors

``` r
fruit <- c("apple","banana")
parse_factor(c("apple","banana", "bananana"), levels = fruit)
```

    ## Warning: 1 parsing failure.
    ## row col           expected   actual
    ##   3  -- value in level set bananana

    ## [1] apple  banana <NA>  
    ## attr(,"problems")
    ## # A tibble: 1 x 4
    ##     row   col expected           actual  
    ##   <int> <int> <chr>              <chr>   
    ## 1     3    NA value in level set bananana
    ## Levels: apple banana

### Dates, Date-Times and Times

``` r
parse_datetime("2010-10-01T2010")
```

    ## [1] "2010-10-01 20:10:00 UTC"

``` r
# If time is omitted, it will be set to midnight
parse_datetime("20101010")
```

    ## [1] "2010-10-10 UTC"

``` r
parse_date("2010-10-01")
```

    ## [1] "2010-10-01"

``` r
library(hms)
parse_time("01:10 am")
```

    ## 01:10:00

``` r
parse_time("20:10:01")
```

    ## 20:10:01

``` r
parse_date("01/02/15", "%m/%d/%y")
```

    ## [1] "2015-01-02"

``` r
parse_date("01/02/15", "%d/%m/%y")
```

    ## [1] "2015-02-01"

``` r
parse_date("01/02/15", "%y/%m/%d")
```

    ## [1] "2001-02-15"

``` r
parse_date("1 janvier 2015", "%d %B %Y", locale = locale("fr"))
```

    ## [1] "2015-01-01"

### Parsing a file

``` r
guess_parser("2010-10-01")
```

    ## [1] "date"

``` r
guess_parser("15:01")
```

    ## [1] "time"

``` r
guess_parser(c("TRUE","FALSE"))
```

    ## [1] "logical"

``` r
guess_parser(c("1","5","9"))
```

    ## [1] "double"

``` r
guess_parser(c("12,345,678"))
```

    ## [1] "number"

``` r
str(parse_guess("2010-10-10"))
```

    ##  Date[1:1], format: "2010-10-10"

### Problems

``` r
challenge <- read_csv(readr_example("challenge.csv"))
```

    ## 
    ## ── Column specification ────────────────────────────────────────────────────────
    ## cols(
    ##   x = col_double(),
    ##   y = col_logical()
    ## )

    ## Warning: 1000 parsing failures.
    ##  row col           expected     actual                                                                                         file
    ## 1001   y 1/0/T/F/TRUE/FALSE 2015-01-16 '/Library/Frameworks/R.framework/Versions/3.6/Resources/library/readr/extdata/challenge.csv'
    ## 1002   y 1/0/T/F/TRUE/FALSE 2018-05-18 '/Library/Frameworks/R.framework/Versions/3.6/Resources/library/readr/extdata/challenge.csv'
    ## 1003   y 1/0/T/F/TRUE/FALSE 2015-09-05 '/Library/Frameworks/R.framework/Versions/3.6/Resources/library/readr/extdata/challenge.csv'
    ## 1004   y 1/0/T/F/TRUE/FALSE 2012-11-28 '/Library/Frameworks/R.framework/Versions/3.6/Resources/library/readr/extdata/challenge.csv'
    ## 1005   y 1/0/T/F/TRUE/FALSE 2020-01-13 '/Library/Frameworks/R.framework/Versions/3.6/Resources/library/readr/extdata/challenge.csv'
    ## .... ... .................. .......... ............................................................................................
    ## See problems(...) for more details.

``` r
problems(challenge)
```

    ## # A tibble: 1,000 x 5
    ##      row col   expected       actual   file                                     
    ##    <int> <chr> <chr>          <chr>    <chr>                                    
    ##  1  1001 y     1/0/T/F/TRUE/… 2015-01… '/Library/Frameworks/R.framework/Version…
    ##  2  1002 y     1/0/T/F/TRUE/… 2018-05… '/Library/Frameworks/R.framework/Version…
    ##  3  1003 y     1/0/T/F/TRUE/… 2015-09… '/Library/Frameworks/R.framework/Version…
    ##  4  1004 y     1/0/T/F/TRUE/… 2012-11… '/Library/Frameworks/R.framework/Version…
    ##  5  1005 y     1/0/T/F/TRUE/… 2020-01… '/Library/Frameworks/R.framework/Version…
    ##  6  1006 y     1/0/T/F/TRUE/… 2016-04… '/Library/Frameworks/R.framework/Version…
    ##  7  1007 y     1/0/T/F/TRUE/… 2011-05… '/Library/Frameworks/R.framework/Version…
    ##  8  1008 y     1/0/T/F/TRUE/… 2020-07… '/Library/Frameworks/R.framework/Version…
    ##  9  1009 y     1/0/T/F/TRUE/… 2011-04… '/Library/Frameworks/R.framework/Version…
    ## 10  1010 y     1/0/T/F/TRUE/… 2010-05… '/Library/Frameworks/R.framework/Version…
    ## # … with 990 more rows

``` r
challenge <- read_csv(
  readr_example("challenge.csv"),
  col_types = cols(
    x = col_double(),
    y = col_character()
  )
)

tail(challenge)
```

    ## # A tibble: 6 x 2
    ##       x y         
    ##   <dbl> <chr>     
    ## 1 0.805 2019-11-21
    ## 2 0.164 2018-03-29
    ## 3 0.472 2014-08-04
    ## 4 0.718 2015-08-16
    ## 5 0.270 2020-02-04
    ## 6 0.608 2019-01-06

``` r
challenge2 <- read_csv(
  readr_example("challenge.csv"),
  guess_max = 1001
)
```

    ## 
    ## ── Column specification ────────────────────────────────────────────────────────
    ## cols(
    ##   x = col_double(),
    ##   y = col_date(format = "")
    ## )

``` r
# read in data as character vectors

challenge2 <- read_csv(
  readr_example("challenge.csv"),
  col_types = cols(.default = col_character())
)
```

``` r
df <- tribble(
  ~x, ~y,
  "1","1.21",
  "2","2.32",
  "3","4.56"
)
df
```

    ## # A tibble: 3 x 2
    ##   x     y    
    ##   <chr> <chr>
    ## 1 1     1.21 
    ## 2 2     2.32 
    ## 3 3     4.56

``` r
type_convert(df)
```

    ## 
    ## ── Column specification ────────────────────────────────────────────────────────
    ## cols(
    ##   x = col_double(),
    ##   y = col_double()
    ## )

    ## # A tibble: 3 x 2
    ##       x     y
    ##   <dbl> <dbl>
    ## 1     1  1.21
    ## 2     2  2.32
    ## 3     3  4.56
