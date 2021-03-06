---
title: "Functions and Control Flow"
author: "José Bayoán Santiago Calderón"
date: "2019-06-07"
output: 
  html_document: 
    keep_md: yes
---

## Housekeeping


```r
invisible(x = suppressPackageStartupMessages(expr = library(package = tidyverse)))
invisible(x = suppressPackageStartupMessages(expr = library(package = maditr)))
invisible(x = suppressPackageStartupMessages(expr = library(package = testthat)))
```


## Functions and Control Flow

Functions are re-usable code.

Usually, functions have three components:

- Arguments
- Logic
- Output

### Example: Factorial

#### Recursive


```r
my_factorial_recursive <- function(n) {
  if ((n %% 1L > 0L) || (n < 0L)) {
    stop("n must be a natural number or zero")
  } else if (n %in% 0L:1L) {
    1L
  } else {
    n * my_factorial_recursive(n - 1L)
    }
  }
for (i in 0L:10L) {
  test_that(desc = "my_factorial_recursive",
            code = expect_equal(object = my_factorial_recursive(n = i),
                                expected = factorial(x = i)))
  }
```

#### Break


```r
my_factorial_break <- function(n) {
  if ((n %% 1L > 0L) || (n < 0L)) {
    stop("n must be a natural number or zero")
  } else if (n %in% 0L:1L) {
    return(value = 1L)
  } else {
    tot <- 1L
    while (TRUE) {
      tot <- tot * n
      n <- n - 1L
      if (n <= 1L) {
        break
        }
      }
    tot
    }
  }
for (i in 0L:10L) {
  test_that(desc = "my_factorial_break",
            code = expect_equal(object = my_factorial_break(n = i),
                                expected = factorial(x = i)))
  }
```

#### While


```r
my_factorial_while <- function(n) {
  if ((n %% 1L > 0L) || (n < 0L)) {
    stop("n must be a natural number or zero")
  }
  tot <- 1L
  while (n > 1L) {
    tot <- tot * n
    n <- n - 1L
    }
  tot
  }
for (i in 0L:10L) {
  test_that(desc = "my_factorial_while",
            code = expect_equal(object = my_factorial_while(n = i),
                                expected = factorial(x = i)))
  }
```

#### For


```r
my_factorial_for <- function(n) {
  if ((n %% 1L > 0L) || (n < 0L)) {
    stop("n must be a natural number or zero")
    }
  if (n %in% 0L:1L) {
    return(value = 1L)
    }
  tot <- 1L
  for (i in 2L:n) {
    tot <- tot * i
  }
  return(value = tot)
  # return(value = prod(1L:n))
  }
for (i in 0L:10L) {
  test_that(desc = "my_factorial_for",
            code = expect_equal(object = my_factorial_for(n = i),
                                expected = factorial(x = i)))
  }
```


### Example Luhn Algorithm (test suite from Exercism.io R track)


```r
luhn <- function(input) {
  if (str_detect(string = input, pattern = "[^\\d\\s]")) {
  # if (str_detect(string = input, pattern = "[^\\d\\s\\p{L}]")) {
    return(value = FALSE)
  }
  input <- str_remove_all(string = input, pattern = "\\s")
  if (nchar(x = input) < 2L) {
    return(value = FALSE)
    }
  total <- 0L
  iseven <- FALSE
  for (idx in nchar(x = input):1L) {
    x <- as.integer(str_sub(string = input, start = idx, end = idx))
    if (iseven) {
      total <- total + ifelse(test = x <= 4, yes = 2 * x, no = 2 * x - 9)
      } else {
      total <- total + x
      }
    iseven <- !iseven
    }
  (total %% 10) == 0L
  }
```


```r
test_that("single digit strings can not be valid", {
  input <- "1"
  expect_equal(luhn(input), FALSE)
  })

test_that("A single zero is invalid", {
  input <- "0"
  expect_equal(luhn(input), FALSE)
  })

test_that("a simple valid SIN that remains valid if reversed", {
  input <- "059"
  expect_equal(luhn(input), TRUE)
  })

test_that("a simple valid SIN that becomes invalid if reversed", {
  input <- "59"
  expect_equal(luhn(input), TRUE)
  })

test_that("valid Canadian SIN", {
  input <- "046 454 286"
  expect_equal(luhn(input), TRUE)
  })

test_that("invalid Canadian SIN", {
  input <- "046 454 287"
  expect_equal(luhn(input), FALSE)
  })

test_that("invalid credit card", {
  input <- "8273 1232 7352 0569"
  expect_equal(luhn(input), FALSE)
  })

test_that("valid strings with a non-digit added become invalid", {
  input <- "055a 444 285"
  expect_equal(luhn(input), FALSE)
  })

test_that("punctuation is not allowed", {
  input <- "055-444-285"
  expect_equal(luhn(input), FALSE)
  })

test_that("symbols are not allowed", {
  input <- "055£ 444$ 285"
  expect_equal(luhn(input), FALSE)
  })

test_that("single zero with space is invalid", {
  input <- " 0"
  expect_equal(luhn(input), FALSE)
  })

test_that("more than a single zero is valid", {
  input <- "0000 0"
  expect_equal(luhn(input), TRUE)
})

test_that("another valid sin", {
  input <- "055 444 285"
  expect_equal(luhn(input), TRUE)
})

test_that("nine doubled is nine", {
  input <- "091"
  expect_equal(luhn(input), TRUE)
})
```

## Vectorized Code

### Basic examples


```r
x <- 1L:1e5L
system.time(expr = x^2)
```

```
##    user  system elapsed 
##       0       0       0
```

```r
system.time(expr = {
  for (idx in seq_along(x)) {
    x[idx] <- idx^2
    }})
```

```
##    user  system elapsed 
##   0.007   0.000   0.007
```


```r
data("iris")
```


```r
x <- list()
for (i in 1L:10L) {
  x[[i]] <- iris
}
```


```r
magic <- function(data) {
  take_if(data = data, Species %in% "setosa")
}
```


```r
y <- copy(x = x)
for (idx in seq_along(along.with = y)) {
  y[[idx]] <- magic(data = y[[idx]])
}
```


```r
y <- do.call(what = rbind, args = y)
```


```r
z <- map_df(.x = x, .f = magic)
```


```r
expect_equal(object = y,
             expected = z)
```


```r
system.time(expr = {
  y <- copy(x = x)
  for (idx in seq_along(along.with = y)) {
    y[[idx]] <- magic(data = y[[idx]])
    }
  y <- do.call(what = rbind, args = y)
  })
```

```
##    user  system elapsed 
##   0.005   0.000   0.005
```

```r
system.time(expr = {
  map_df(.x = x, .f = magic)
  })
```

```
##    user  system elapsed 
##   0.005   0.000   0.005
```
