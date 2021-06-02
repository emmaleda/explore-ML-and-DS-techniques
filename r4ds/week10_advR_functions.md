(Advanced R) Functions
================
Emma Grossman
5/31/2021

## 6.2 Function fundamentals

Functions are comprised of 3 things: (1) arguments, (2) body and (3)
environment.

Functions are objects, like vectors are objects.

The `formals()` are a list of function arguments, the `body()` is code
inside a function, and the `environment()` determines how the function
finds values associated with the names.

``` r
f02 <- function(x, y) {
  # A comment
  x + y
}

formals(f02)
```

    ## $x
    ## 
    ## 
    ## $y

``` r
body(f02)
```

    ## {
    ##     x + y
    ## }

``` r
environment(f02)
```

    ## <environment: R_GlobalEnv>

Functions can also possess additional `attributes()` (like comments)
which can be accessed with `srcref` (short for source reference).

``` r
attr(f02, "srcref")
```

    ## function(x, y) {
    ##   # A comment
    ##   x + y
    ## }

Certain functions in R are known as primitive functions because they
call directly on C code rather than R code.

``` r
sum
```

    ## function (..., na.rm = FALSE)  .Primitive("sum")

``` r
typeof(sum)
```

    ## [1] "builtin"

``` r
typeof(`[`)
```

    ## [1] "special"

``` r
formals(sum)
```

    ## NULL

``` r
body(sum)
```

    ## NULL

``` r
environment(sum)
```

    ## NULL

> It’s very important to understand that R functions are objects in
> their own right, a language property often called “first-class
> functions”.

Sometimes, we chose not to give functions names. These functions are
known as **anonymous functions**.

``` r
lapply(mtcars, function(x) length(unique(x)))
```

    ## $mpg
    ## [1] 25
    ## 
    ## $cyl
    ## [1] 3
    ## 
    ## $disp
    ## [1] 27
    ## 
    ## $hp
    ## [1] 22
    ## 
    ## $drat
    ## [1] 22
    ## 
    ## $wt
    ## [1] 29
    ## 
    ## $qsec
    ## [1] 30
    ## 
    ## $vs
    ## [1] 2
    ## 
    ## $am
    ## [1] 2
    ## 
    ## $gear
    ## [1] 3
    ## 
    ## $carb
    ## [1] 6

``` r
Filter(function(x) !is.numeric(x), mtcars)
```

    ## data frame with 0 columns and 32 rows

``` r
integrate(function(x) sin(x) ^ 2, 0, pi)
```

    ## 1.570796 with absolute error < 1.7e-14

Functions can also be created in lists.

``` r
funs <- list(
  half = function(x) x / 2,
  double = function(x) x * 2
)

funs$double(10)
```

    ## [1] 20

> In R, you’ll often see functions called closures. This name reflects
> the fact that R functions capture, or enclose, their environments

If the arguments of a function are in a data structure, we can use the
function `do.call()`.

``` r
args <- list(1:10, na.rm = TRUE)
mean(args)
```

    ## Warning in mean.default(args): argument is not numeric or logical: returning NA

    ## [1] NA

``` r
do.call(mean, args)
```

    ## [1] 5.5

## 6.3 Function composition

Piping works well for creating functions.

``` r
square <- function(x) x^2
deviation <- function(x) x - mean(x)

x <- runif(100)

sqrt(mean(square(deviation(x))))
```

    ## [1] 0.2761665

``` r
out <- deviation(x)
out <- square(out)
out <- mean(out)
out <- sqrt(out)
out
```

    ## [1] 0.2761665

``` r
library(magrittr)

x %>%
  deviation() %>%
  square() %>%
  mean() %>%
  sqrt()
```

    ## [1] 0.2761665

## 6.4 Lexical scoping

**Scoping** is the act of finding a value associated with a name.

> R uses lexical scoping: it looks up the values of names based on how a
> function is defined, not how it is called. “Lexical” here is not the
> English adjective that means relating to words or a vocabulary. It’s a
> technical CS term that tells us that the scoping rules use a
> parse-time, rather than a run-time structure.

The four rules that R uses for lexical scoping are (1) name masking, (2)
functions vs. variables, (3) a fresh start and (4) dynamic lookup.

**Name Masking:**

Names defined inside of a function mask names outside.

``` r
x <- 10
y <- 20
g02 <- function() {
  x <- 1
  y <- 2
  c(x, y)
}
g02()
```

    ## [1] 1 2

If R cannot find a name in a function, it looks one level up.

``` r
x <- 2
g03 <- function() {
  y <- 1
  c(x, y)
}
g03()
```

    ## [1] 2 1

``` r
y
```

    ## [1] 20

``` r
x <- 1
g04 <- function() {
  y <- 2
  i <- function() {
    z <- 3
    c(x, y, z)
  }
  i()
}
g04()
```

    ## [1] 1 2 3

``` r
g07 <- function(x) x + 1
g08 <- function() {
  g07 <- function(x) x + 100
  g07(10)
}
g08()
```

    ## [1] 110

> However, when a function and a non-function share the same name (they
> must, of course, reside in different environments), applying these
> rules gets a little more complicated. When you use a name in a
> function call, R ignores non-function objects when looking for that
> value.

``` r
g09 <- function(x) x + 100
g10 <- function() {
  g09 <- 10
  g09(g09)
}
g10()
```

    ## [1] 110

``` r
g11 <- function() {
  if (!exists("a")) {
    a <- 1
  } else {
    a <- a + 1
  }
  a
}

g11()
```

    ## [1] 1

``` r
g11()
```

    ## [1] 1

> This means that a function has no way to tell what happened the last
> time it was run; each invocation is completely independent.

``` r
g12 <- function() x + 1
x <- 15
g12()
```

    ## [1] 16

``` r
x <- 20
g12()
```

    ## [1] 21

``` r
codetools::findGlobals(g12)
```

    ## [1] "+" "x"

``` r
environment(g12) <- emptyenv()
g12()
```

    ## Error in x + 1: could not find function "+"

## 6.5 Lazy evaluation

Function arguments are only evaluated if they are accessed, a system
called **lazy evaluation**. This is useful because we can include
computationally expensive components in a function which will only be
evaluated if needed.

A **promise** is the data structure that allows for lazy evaluation. It
is made up of an expression, an environment, and a value.

``` r
y <- 10
h02 <- function(x) {
  y <- 100
  x + 1
}

h02(y)
```

    ## [1] 11

``` r
h02(y <- 1000)
```

    ## [1] 1001

``` r
y
```

    ## [1] 1000

``` r
double <- function(x) { 
  message("Calculating...")
  x * 2
}

h03 <- function(x) {
  c(x, x)
}

h03(double(20))
```

    ## Calculating...

    ## [1] 40 40

``` r
h04 <- function(x = 1, y = x * 2, z = a + b) {
  a <- 10
  b <- 100
  
  c(x, y, z)
}

h04()
```

    ## [1]   1   2 110

``` r
h05 <- function(x = ls()) {
  a <- 1
  x
}

# ls() evaluated inside h05:
h05()
```

    ## [1] "a" "x"

``` r
# ls() evaluated in global environment:
h05(ls())
```

    ##  [1] "args"      "deviation" "double"    "f02"       "funs"      "g02"      
    ##  [7] "g03"       "g04"       "g07"       "g08"       "g09"       "g10"      
    ## [13] "g11"       "g12"       "h02"       "h03"       "h04"       "h05"      
    ## [19] "out"       "square"    "x"         "y"

``` r
args(sample)
```

    ## function (x, size, replace = FALSE, prob = NULL) 
    ## NULL

``` r
sample <- function(x, size = NULL, replace = FALSE, prob = NULL) {
  if (is.null(size)) {
    size <- length(x)
  }
  
  x[sample.int(length(x), size, replace = replace, prob = prob)]
}
```

``` r
`%||%` <- function(lhs, rhs) {
  if (!is.null(lhs)) {
    lhs
  } else {
    rhs
  }
}

sample <- function(x, size = NULL, replace = FALSE, prob = NULL) {
  size <- size %||% length(x)
  x[sample.int(length(x), size, replace = replace, prob = prob)]
}
```

> Because of lazy evaluation, you don’t need to worry about unnecessary
> computation: the right side of %||% will only be evaluated if the left
> side is NULL.

## 5.6 `...` dot-dot-dot

`...` can be used to pass on arguments to another function.

``` r
i01 <- function(y, z) {
  list(y = y, z = z)
}

i02 <- function(x, ...) {
  i01(...)
}

str(i02(x = 1, y = 2, z = 3))
```

    ## List of 2
    ##  $ y: num 2
    ##  $ z: num 3

Very rarely used:

``` r
i03 <- function(...) {
  list(first = ..1, third = ..3)
}
str(i03(1, 2, 3))
```

    ## List of 2
    ##  $ first: num 1
    ##  $ third: num 3

`list(...)` evalutates and stores arguments.

``` r
i04 <- function(...) {
  list(...)
}
str(i04(a = 1, b = 2))
```

    ## List of 2
    ##  $ a: num 1
    ##  $ b: num 2

If a function takes another function as an argument, aditional arguments
are passed on with `...`.

``` r
x <- list(c(1, 3, NA), c(4, NA, 6))
str(lapply(x, mean, na.rm = TRUE))
```

    ## List of 2
    ##  $ : num 2
    ##  $ : num 5

``` r
print(factor(letters), max.levels = 4)
```

    ##  [1] a b c d e f g h i j k l m n o p q r s t u v w x y z
    ## 26 Levels: a b c ... z

``` r
print(y ~ x, showEnv = TRUE)
```

    ## y ~ x
    ## <environment: R_GlobalEnv>

An error will not be generated from a misspelled argument, so typos are
common.

## 6.7 Exiting a function

There are implicit and explicit ways to return.

``` r
j01 <- function(x) {
  if (x < 10) {
    0
  } else {
    10
  }
}
j01(5)
```

    ## [1] 0

``` r
j01(15)
```

    ## [1] 10

``` r
j02 <- function(x) {
  if (x < 10) {
    return(0)
  } else {
    return(10)
  }
}
j02(5)
```

    ## [1] 0

We can prevent automatic printing by including `invisible()`:

``` r
j03 <- function() 1
j03()
```

    ## [1] 1

``` r
j04 <- function() invisible(1)
j04()
```

To ensure that `j04` is performing as expected, we can use `print()` or
wrap it in parenthesis.

``` r
print(j04())
```

    ## [1] 1

``` r
(j04())
```

    ## [1] 1

``` r
str(withVisible(j04())) # visibility flag
```

    ## List of 2
    ##  $ value  : num 1
    ##  $ visible: logi FALSE

``` r
#> List of 2
#>  $ value  : num 1
#>  $ visible: logi FALSE
```

> The most common function that returns invisibly is \<-:

``` r
a <- 2
(a <- 2)
```

    ## [1] 2

``` r
j05 <- function() {
  stop("I'm an error") # an error which terminates the function
  return(10)
}
j05()
```

    ## Error in j05(): I'm an error

`on.exit()` is a function that sets up an **exit handler**.

``` r
j06 <- function(x) {
  cat("Hello\n")
  on.exit(cat("Goodbye!\n"), add = TRUE)
  
  if (x) {
    return(10)
  } else {
    stop("Error")
  }
}

j06(TRUE)
```

    ## Hello
    ## Goodbye!

    ## [1] 10

``` r
j06(FALSE)
```

    ## Hello

    ## Error in j06(FALSE): Error

    ## Goodbye!

Important\! \> Always set add = TRUE when using on.exit(). If you don’t,
each call to on.exit() will overwrite the previous exit handler. Even
when only registering a single handler, it’s good practice to set add =
TRUE so that you won’t get any unpleasant surprises if you later add
more exit handlers.

It is easy to handle code that requires clean up:

``` r
cleanup <- function(dir, code) {
  old_dir <- setwd(dir)
  on.exit(setwd(old_dir), add = TRUE)
  
  old_opt <- options(stringsAsFactors = FALSE)
  on.exit(options(old_opt), add = TRUE)
}
```

``` r
with_dir <- function(dir, code) {
  old <- setwd(dir)
  on.exit(setwd(old), add = TRUE)

  force(code)
}

getwd()
```

    ## [1] "/Users/emmaleda/Documents/School/OSU/6. Spring 2021/R&C/explore-ML-and-DS-techniques/r4ds"

``` r
with_dir("~", getwd())
```

    ## [1] "/Users/emmaleda"

## 6.8 Function forms

There are four variates of function calls:

1.  **prefix**, in which the function name comes before the arguments
2.  **infix**, in which the function rests in between the arguments like
    the “+” in “x + y”
3.  **replacement**, in which the function assigns values by replacing
    others
4.  **special** functions like `if` and `for`

Types 2, 3, and 4 can all be written in prefix forms in R.

``` r
x + y
`+`(x, y)

names(df) <- c("x", "y", "z")
`names<-`(df, c("x", "y", "z"))

for(i in 1:10) print(i)
`for`(i, 1:10, print(i))
```

Infix functions can be created in R by starting and ending with `%`.

``` r
`%+%` <- function(a, b) paste0(a, b)
"new " %+% "string"
```

    ## [1] "new string"

``` r
`% %` <- function(a, b) paste(a, b)
`%/\\%` <- function(a, b) paste(a, b)

"a" % % "b"
```

    ## [1] "a b"

``` r
"a" %/\% "b"
```

    ## [1] "a b"

``` r
`%-%` <- function(a, b) paste0("(", a, " %-% ", b, ")")
"a" %-% "b" %-% "c"
```

    ## [1] "((a %-% b) %-% c)"
