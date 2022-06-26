Project 1 - API Vignette
================
Tommy King
2022-06-26

-   [Introduction](#introduction)
-   [Requirements](#requirements)
-   [Functions Written for (Gotta) Catching All of the
    Data](#functions-written-for-gotta-catching-all-of-the-data)
    -   [Damage Type Functions](#damage-type-functions)

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

## Damage Type Functions

In this section of functions, we will make some tools for querying the
Pokemon type data so we can do some analysis on the balance between
various types and their damage effectiveness vs. other types.

### It’s super effective!

This first function we’re going to write is going to take a Pokemon type
and utilize the `types` endpoint in the PokeAPI to return a list of
types that the given type is super effective against. This will help us
out later on in our analysis when we want to look at how many types each
type is super effective against. We take the desired type as an argument
here, grab the JSON from the API and drill down into the attribute
`double_damage_to` which will give us a vector of the super effective
types.

``` r
superE <- function(type){
  type_info <- fromJSON(paste0("https://pokeapi.co/api/v2/type/",type,"/"))
  
  return(type_info$damage_relations$double_damage_to$name)
}
```

### It’s not very effective…

The next function we’ll add is very similar to the previous one, but
instead of getting the types our given type is super effective against,
we’ll grab the types that our given one only does half damage against.

``` r
notVery <- function(type){
  type_info <- fromJSON(paste0("https://pokeapi.co/api/v2/type/",type,"/"))
  
  return(type_info$damage_relations$half_damage_to$name)
}
```

### It doesn’t affect…

In the same vein here, this function will grab any types that the given
type don’t do any damage aginst. We’ll eventually combine this with the
previous function to compile any “bad” outcomes, ones where the given
type isn’t doing as much damage as it should.

``` r
noEffect <- function(type){
  type_info <- fromJSON(paste0("https://pokeapi.co/api/v2/type/",type,"/"))
  
  return(type_info$damage_relations$no_damage_to$name)
}
```
