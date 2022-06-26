Project 1 - API Vignette
================
Tommy King
2022-06-26

-   [Introduction](#introduction)
-   [Requirements](#requirements)

# Introduction

In this vignette, we’re going to be taking a look at the PokeAPI,
writing some functions to interact with it.

# Requirements

For the work done here, we will be using the `tidyverse` package for all
of our data manipulation, the `httr` package for making https requests
to the API and `jsonlite` for interacting with json files. The code
chunk for loading in these packages is included below

``` r
library(tidyverse)
library(httr)
library(jsonlite)
```
