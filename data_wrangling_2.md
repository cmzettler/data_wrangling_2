data wrangling 2
================

``` r
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.1 ──

    ## ✓ ggplot2 3.3.5     ✓ purrr   0.3.4
    ## ✓ tibble  3.1.4     ✓ dplyr   1.0.7
    ## ✓ tidyr   1.1.3     ✓ stringr 1.4.0
    ## ✓ readr   2.0.1     ✓ forcats 0.5.1

    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library(rvest)
```

    ## 
    ## Attaching package: 'rvest'

    ## The following object is masked from 'package:readr':
    ## 
    ##     guess_encoding

``` r
library(httr)


knitr::opts_chunk$set(
  fig.width = 6, 
  fig.asp = .6, 
  out.width = "90%"
)
```

## Getting data from the web

National Survey on Drug Use and Health (2013-2014 & 2014-2015)

``` r
url = "http://samhda.s3-us-gov-west-1.amazonaws.com/s3fs-public/field-uploads/2k15StateFiles/NSDUHsaeShortTermCHG2015.htm"

drug_use_html = 
  read_html(url)

drug_use_df = 
  drug_use_html %>% 
  html_table() %>% 
  first() %>% 
  slice(-1)
```

The read\_html function goes fully onto the internet every time.

First row is the footnote (first row everywhere)

## Star Wars

Get some star wars data…

``` r
sw_url = "https://www.imdb.com/list/ls070150896/"

sw_html = 
  read_html(sw_url)
```

Not a well formatted thing, so we nee to use selector gadget

``` r
sw_titles = sw_html %>% 
  html_elements(".lister-item-header a") %>% 
  html_text()

sw_revenue = sw_html %>% 
  html_elements(".text-small:nth-child(7) span:nth-child(5)") %>% 
  html_text()

sw_df = 
  tibble(
    title = sw_titles, 
    revenue = sw_revenue
  )
```

## Napoleon Dynamite

Dynamite reviews

``` r
dynamite_url = "https://www.amazon.com/product-reviews/B00005JNBQ/ref=cm_cr_arp_d_viewopt_rvwer?ie=UTF8&reviewerType=avp_only_reviews&sortBy=recent&pageNumber=1"

dynamite_html = 
  read_html(dynamite_url)

dynamite_review_titles = 
  dynamite_html %>% 
  html_elements(".a-text-bold span") %>% 
  html_text

dynamite_stars = 
  dynamite_html %>% 
  html_elements("#cm_cr-review_list .review-rating") %>% 
  html_text

dynamite_df = 
  tibble(titles = dynamite_review_titles, 
         stars = dynamite_stars)
```

## APIs

Get some data from an API about water.

``` r
water_df = 
  GET("https://data.cityofnewyork.us/resource/ia2d-e54m.csv") %>% 
  content()
```

    ## Rows: 42 Columns: 4

    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## dbl (4): year, new_york_city_population, nyc_consumption_million_gallons_per...

    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

Let’s see what JSON looks like…

``` r
water_df = 
  GET("https://data.cityofnewyork.us/resource/ia2d-e54m.json") %>% 
  content("text") %>% 
  jsonlite::fromJSON() %>% 
  as_tibble()
```

## BRFSS data via API

``` r
brfss_df = 
  GET("https://chronicdata.cdc.gov/resource/acme-vg9e.csv",
      query = list("$limit" = 5000)) %>% 
  content() 
```

    ## Rows: 5000 Columns: 23

    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (16): locationabbr, locationdesc, class, topic, question, response, data...
    ## dbl  (6): year, sample_size, data_value, confidence_limit_low, confidence_li...
    ## lgl  (1): locationid

    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

There are actuallly 134,000 rows, but it only gives the first 1,000.
(Bandwidth protection)

## Pokemon API

``` r
pokemon_data = 
  GET("https://pokeapi.co/api/v2/pokemon/1") %>% 
  content()
  
pokemon_data[["name"]]
```

    ## [1] "bulbasaur"

``` r
pokemon_data[["height"]]
```

    ## [1] 7

``` r
pokemon_data[["abilities"]]
```

    ## [[1]]
    ## [[1]]$ability
    ## [[1]]$ability$name
    ## [1] "overgrow"
    ## 
    ## [[1]]$ability$url
    ## [1] "https://pokeapi.co/api/v2/ability/65/"
    ## 
    ## 
    ## [[1]]$is_hidden
    ## [1] FALSE
    ## 
    ## [[1]]$slot
    ## [1] 1
    ## 
    ## 
    ## [[2]]
    ## [[2]]$ability
    ## [[2]]$ability$name
    ## [1] "chlorophyll"
    ## 
    ## [[2]]$ability$url
    ## [1] "https://pokeapi.co/api/v2/ability/34/"
    ## 
    ## 
    ## [[2]]$is_hidden
    ## [1] TRUE
    ## 
    ## [[2]]$slot
    ## [1] 3

not nicely formatted as a csv

R package - wrappers for APIs

rnoaa package rtweet (twitter data)
