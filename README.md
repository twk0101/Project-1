Project 1 - API Vignette
================
Tommy King
2022-06-26

-   [Introduction](#introduction)
-   [Requirements](#requirements)
-   [Functions Written for (Gotta) Catching All of the
    Data](#functions-written-for-gotta-catching-all-of-the-data)
    -   [Damage Type Analysis Function](#damage-type-analysis-function)
    -   [Matchup Matrix Generation.](#matchup-matrix-generation)
    -   [Pokemon Type Distribution
        Function](#pokemon-type-distribution-function)

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

# What Pokemon types are ground types super effective against?
damage("ground", "super_effective")
```

    ## [1] "poison"   "rock"     "steel"    "fire"     "electric"

## Matchup Matrix Generation.

Next, we’re going to write another function that will help us aggregate
some of this data based on a “good” or “bad” damage trait for a given
type. “Good” outcomes are when the given type is super effective against
another type, or when another type is not very effective, or does no
damage to it. “Bad” outcomes are the inverse, when another type is super
effective against the given type, or when the given type is not very
effective or does no damage to another type. To give this function a
little more utility, we’re going to give the user the opportunity to
weight these outcomes (`extreme_weight` corresponds to the good outcome
of being super effective and multiplied by -1 corresponds to the bad
outcomes of doing no damage and having an opposing type be super
effective. `modest_weight` corresponds to the good outcomes of having an
opposing type do half or no damage and multiplied by -1 corresponds to
the bad outcome of doing half damage against another type.) We then
create a matrix of all the possible type matchups and fill in a value
based on the weights and the data we pull from the previous function.

We’ll make a vector of all the possible types, use this vector for both
the rows and columns of our matrix and then fill in the individual cells
by iterating through all the types and getting the corresponding damage
lists from our previous function. Note that we’re adding the weight
values here to the matchup matrix for each new damage class we find in
the hopes of estimating the total advantage or disadvantage that the
type in the row has versus the type in the column.

``` r
generate_matchups <- function(extreme_weight = 2, modest_weight = 1){  
  types <- c("normal", "fire", "water", "electric", "grass", "ice", "fighting", "poison", "ground",          "flying", "psychic", "bug", "rock", "ghost", "dragon", "dark", "steel", "fairy")
  
  matchups <- matrix(0, 18, 18)
  rownames(matchups) <- as.list(types)
  colnames(matchups) <- as.list(types)
  
  for (t in types)
  {
    super <- damage(t, "super_effective")
    half <- damage(t, "not_very")
    no <- damage(t, "no_effect")
    
    for (s in super)
    {
      matchups[t,s] = matchups[t,s] + extreme_weight
      matchups[s,t] = matchups[s,t] + (-1 * extreme_weight)
    }
    for (h in half)
    {
      matchups[t,h] = matchups[t,h] + (-1 * modest_weight)
      matchups[h,t] = matchups[h,t] + modest_weight
    }
    for (n in no)
    {
      matchups[t,n] = matchups[t,n] + (-1 * extreme_weight)
      matchups[n,t] =matchups[n,t] + extreme_weight
    }
  }
  
  return(matchups)
}

# Taking a look at our generated matrix with the default weights
generate_matchups()
```

    ##          normal fire water electric grass ice fighting poison ground
    ## normal        0    0     0        0     0   0       -2      0      0
    ## fire          0    0    -3        0     3   3        0      0     -2
    ## water         0    3     0       -2    -3   1        0      0      2
    ## electric      0    0     2        0    -1   0        0      0     -4
    ## grass         0   -3     3        1     0  -2        0     -3      3
    ## ice           0   -3    -1        0     2   0       -2      0      2
    ## fighting      2    0     0        0     0   2        0     -1      0
    ## poison        0    0     0        0     3   0        1      0     -3
    ## ground        0    2    -2        4    -3  -2        0      3      0
    ## flying        0    0     0       -3     3  -2        3      0      2
    ## psychic       0    0     0        0     0   0        3      2      0
    ## bug           0   -3     0        0     3   0        0     -1      1
    ## rock          1    3    -2        0    -2   2       -3      1     -3
    ## ghost         0    0     0        0     0   0        2      1      0
    ## dragon        0    1     1        1     1  -2        0      0      0
    ## dark          0    0     0        0     0   0       -3      0      0
    ## steel         1   -3    -1       -1     1   3       -2      2     -2
    ## fairy         0   -1     0        0     0   0        3     -3      0
    ##          flying psychic bug rock ghost dragon dark steel fairy
    ## normal        0       0   0   -1     0      0    0    -1     0
    ## fire          0       0   3   -3     0     -1    0     3     1
    ## water         0       0   0    2     0     -1    0     1     0
    ## electric      3       0   0    0     0     -1    0     1     0
    ## grass        -3       0  -3    2     0     -1    0    -1     0
    ## ice           2       0   0   -2     0      2    0    -3     0
    ## fighting     -3      -3   0    3    -2      0    3     2    -3
    ## poison        0      -2   1   -1    -1      0    0    -2     3
    ## ground       -2       0  -1    3     0      0    0     2     0
    ## flying        0       0   3   -3     0      0    0    -1     0
    ## psychic       0       0  -2    0    -2      0   -4    -1     0
    ## bug          -3       2   0   -2    -1      0    2    -1    -1
    ## rock          3       0   2    0     0      0    0    -3     0
    ## ghost         0       2   1    0     0      0   -3     0     0
    ## dragon        0       0   0    0     0      0    0    -1    -4
    ## dark          0       4  -2    0     3      0    0     0    -3
    ## steel         1       1   1    3     0      1    0     0     3
    ## fairy         0       0   1    0     0      4    3    -3     0

## Pokemon Type Distribution Function

Finally, we’re going to make a function that will generate the
distribution of the number of Pokemon belonging to each type so that we
can factor in the number of pokemon belonging to an opposing type and
factor this into our overall interest in deciding which type has the
highest overall advantage. We’re going to query a different endpoint in
the PokeAPI for this, `Pokemon` instead of `types`. We’re going to
maintain a dataframe that’s incrementing the counts for each type. Note
that we’re going to count a Pokemon that has more than one type as one
in each category for this purpose.

``` r
generate_distribution <- function(){
  counts <- rep(0, 18)
  types <- c("normal", "fire", "water", "electric", "grass", "ice", "fighting", "poison", "ground",          "flying", "psychic", "bug", "rock", "ghost", "dragon", "dark", "steel", "fairy")
  
  for (i in  1:905)
  {
    poke <- fromJSON(paste0("https://pokeapi.co/api/v2/pokemon/",i,"/"))
    for (t in poke$types$type$name)
    {
      counts[grep(t, types)] = counts[grep(t, types)] + 1
    }
  }
  
  return(counts)
}

# Taking a look at our raw counts
poke_counts <- generate_distribution()
poke_counts
```

    ##  [1] 117  71 142  57 107  41  62  71  68 103  91  85  66  52  56  56  54
    ## [18]  55
