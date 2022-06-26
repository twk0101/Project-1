Project 1 - API Vignette
================
Tommy King
2022-06-26

-   [Introduction](#introduction)
-   [Requirements](#requirements)
-   [Functions Written for (Gotta) Catching All of the
    Data](#functions-written-for-gotta-catching-all-of-the-data)
    -   [Damage Type Analysis Function](#damage-type-analysis-function)

# Introduction

In this vignette, we’re going to be taking a look at the PokeAPI,
writing some functions to interact with it, and doing some exploratory
data analysis on the information we’re grabbing.

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

# Functions Written for (Gotta) Catching All of the Data

## Damage Type Analysis Function

In this section of functions, we will make some tools for querying the
Pokemon type data so we can do some analysis on the balance between
various types and their damage effectiveness vs. other types. We’ll
write one master function here that takes the user argument of a type
and a specification of what kind of damage class we’re interested in:  
\* `super_effective` (double damage **from** the given type),  
\* `not_very` (half damage **from** the given type)  
\* `no_effect` (no damage **from** the given type)  
\* `super_against` (double damage **against** the given type)  
\* `not_very_against` (half damage **against** the given type)  
\* `no_effect_against` (no damage **against** the given type)

Our function below will query the `type` endpoint on the PokeAPI. It’s
going to grab the JSON at the url we build using the type we take in as
an argument, and then drills down onto the desired damage class based on
the other argument we’re taking in. The return value here is a vector of
the names of the other type(s) that fit the specifications.

``` r
damage <- function(type, damage_class){
  type_info <- fromJSON(paste0("https://pokeapi.co/api/v2/type/",type,"/"))
 
  attrib <- switch(damage_class, "super_effective" = "double_damage_to",
                                 "not_very" = "half_damage_to",
                                 "no_effect" = "no_damage_to",
                                 "super_against" = "double_damage_against",
                                 "not_very_against" = "half_damage_against",
                                 "no_effect_against" = "no_damage_against")
  
  damage_list <- type_info$damage_relations[[attrib]]
  return(damage_list$name)
}
```
