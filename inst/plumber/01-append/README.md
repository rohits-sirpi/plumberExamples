
<!-- README.md is generated from README.Rmd. Please edit that file -->

# Appender

This API highlights how to use the global environment to keep track of
API state. The [managing
state](https://www.rplumber.io/articles/execution-model.html#managing-state-1)
section of the Plumber documentation contains further details.

## Endpoints

  - `/append`: Append a value to a global set of values
  - `/tail`: Return the last n values of the global value store
  - `/graph`: Return a plot of the values contained in the global value
    store

## Definition

### [plumber.R](plumber.R)

``` r
values <- 15

MAX_VALS <- 50

#* Append to our values
#* @param val:int value to append
#* @post /append
function(val, res){
  v <- as.numeric(val)
  if (is.na(v)){
    res$status <- 400
    res$body <- "val parameter must be a number"
  }
  values <<- c(values, val)

  if (length(values) > MAX_VALS){
    values <<- tail(values, n=MAX_VALS)
  }

  list(result="success")
}

#* Get the last few values
#* @get /tail
function(n="10", res){
  n <- as.numeric(n)
  if (is.na(n) || n < 1 || n > MAX_VALS){
    res$status <- 400
    res$body <- "parameter 'n' must be a number between 1 and 100"
  }

  list(val=tail(values, n=n))
}

#* Get a graph of the values
#* @png
#* @get /graph
function(){
  plot(values, type="b", ylim=c(1,100), main="Recent Values")
}
```

### [Tidy Plumber](tidy-plumber.R)

``` r
library(plumber)

values <- 15

MAX_VALS <- 50

pr() %>% 
  pr_post(path = "/append",
          handler = function(val, res){
            v <- as.numeric(val)
            if (is.na(v)){
              res$status <- 400
              res$body <- "val parameter must be a number"
            }
            values <<- c(values, val)
            
            if (length(values) > MAX_VALS){
              values <<- tail(values, n=MAX_VALS)
            }
            
            list(result="success")
          }) %>% 
  pr_get(path = "/tail",
         handler = function(n="10", res){
           n <- as.numeric(n)
           if (is.na(n) || n < 1 || n > MAX_VALS){
             res$status <- 400
             res$body <- "parameter 'n' must be a number between 1 and 100"
           }
           
           list(val=tail(values, n=n))
         }) %>% 
  pr_get(path = "/graph",
         handler = function(){
           plot(values, type="b", ylim=c(1,100), main="Recent Values")
         },
         serializer = serializer_png()) %>% 
  pr_run()
```