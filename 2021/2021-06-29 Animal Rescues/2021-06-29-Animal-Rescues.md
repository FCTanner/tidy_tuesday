2021-06-29 Animal Rescues
================

``` r
library(tidyverse)
```

    ## Warning: package 'tidyverse' was built under R version 4.0.5

    ## -- Attaching packages --------------------------------------- tidyverse 1.3.1 --

    ## v ggplot2 3.3.3     v purrr   0.3.4
    ## v tibble  3.1.2     v dplyr   1.0.6
    ## v tidyr   1.1.3     v stringr 1.4.0
    ## v readr   1.4.0     v forcats 0.5.1

    ## Warning: package 'tibble' was built under R version 4.0.5

    ## Warning: package 'tidyr' was built under R version 4.0.5

    ## Warning: package 'dplyr' was built under R version 4.0.5

    ## Warning: package 'forcats' was built under R version 4.0.5

    ## -- Conflicts ------------------------------------------ tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library(ggmap)
```

    ## Warning: package 'ggmap' was built under R version 4.0.5

    ## Google's Terms of Service: https://cloud.google.com/maps-platform/terms/.

    ## Please cite ggmap if you use it! See citation("ggmap") for details.

``` r
theme_set(theme_bw())
```

## Read data

``` r
animal_rescues <- readr::read_csv('https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2021/2021-06-29/animal_rescues.csv')
```

    ## 
    ## -- Column specification --------------------------------------------------------
    ## cols(
    ##   .default = col_character(),
    ##   incident_number = col_double(),
    ##   cal_year = col_double(),
    ##   hourly_notional_cost = col_double(),
    ##   easting_rounded = col_double(),
    ##   northing_rounded = col_double()
    ## )
    ## i Use `spec()` for the full column specifications.

## EDA

What animals are involved?

  - Cats are most common
  - 12 incidents with snakes - what happened?
  - 2 incidents with fish - what happened?

<!-- end list -->

``` r
animal_rescues %>% 
  group_by(animal_group_parent) %>% 
  count
```

    ## # A tibble: 28 x 2
    ## # Groups:   animal_group_parent [28]
    ##    animal_group_parent     n
    ##    <chr>               <int>
    ##  1 Bird                 1530
    ##  2 Budgie                  2
    ##  3 Bull                    1
    ##  4 cat                    17
    ##  5 Cat                  3649
    ##  6 Cow                     8
    ##  7 Deer                  130
    ##  8 Dog                  1194
    ##  9 Ferret                  8
    ## 10 Fish                    2
    ## # ... with 18 more rows

### Looking at a few incidents

FISH IN DANGER OF DYING IN POND - more details please :)

``` r
animal_rescues %>% 
 filter(animal_group_parent %in% c("Horse"))
```

    ## # A tibble: 193 x 31
    ##    incident_number date_time_of_call cal_year fin_year type_of_incident
    ##              <dbl> <chr>                <dbl> <chr>    <chr>           
    ##  1         2872091 05/01/2009 12:27      2009 2008/09  Special Service 
    ##  2        40866091 12/03/2009 11:51      2009 2008/09  Special Service 
    ##  3        76285091 07/05/2009 10:46      2009 2009/10  Special Service 
    ##  4        87402091 24/05/2009 14:53      2009 2009/10  Special Service 
    ##  5        89957091 28/05/2009 11:11      2009 2009/10  Special Service 
    ##  6       102278091 15/06/2009 15:13      2009 2009/10  Special Service 
    ##  7       123495091 11/07/2009 11:26      2009 2009/10  Special Service 
    ##  8       137525091 02/08/2009 11:38      2009 2009/10  Special Service 
    ##  9       144791091 13/08/2009 12:05      2009 2009/10  Special Service 
    ## 10       151984091 23/08/2009 20:55      2009 2009/10  Special Service 
    ## # ... with 183 more rows, and 26 more variables: pump_count <chr>,
    ## #   pump_hours_total <chr>, hourly_notional_cost <dbl>,
    ## #   incident_notional_cost <chr>, final_description <chr>,
    ## #   animal_group_parent <chr>, originof_call <chr>, property_type <chr>,
    ## #   property_category <chr>, special_service_type_category <chr>,
    ## #   special_service_type <chr>, ward_code <chr>, ward <chr>,
    ## #   borough_code <chr>, borough <chr>, stn_ground_name <chr>, uprn <chr>,
    ## #   street <chr>, usrn <chr>, postcode_district <chr>, easting_m <chr>,
    ## #   northing_m <chr>, easting_rounded <dbl>, northing_rounded <dbl>,
    ## #   latitude <chr>, longitude <chr>

### Animals rescued from height

  - Apparently a snake was rescued from height, what happened there?
  - Will focus on cats on trees, where does that happen?

<!-- end list -->

``` r
animal_rescues %>% 
  filter(special_service_type_category == "Animal rescue from height") %>% 
  group_by(animal_group_parent) %>% 
  count
```

    ## # A tibble: 14 x 2
    ## # Groups:   animal_group_parent [14]
    ##    animal_group_parent                  n
    ##    <chr>                            <int>
    ##  1 Bird                               965
    ##  2 Budgie                               1
    ##  3 cat                                  3
    ##  4 Cat                               1494
    ##  5 Dog                                122
    ##  6 Ferret                               2
    ##  7 Fox                                 24
    ##  8 Horse                                2
    ##  9 Lizard                               1
    ## 10 Pigeon                               1
    ## 11 Snake                                1
    ## 12 Squirrel                            25
    ## 13 Unknown - Domestic Animal Or Pet    74
    ## 14 Unknown - Wild Animal                6

``` r
cats <- animal_rescues %>% 
  filter(special_service_type_category == "Animal rescue from height",
         animal_group_parent %in% c("cat", "Cat")) %>% 
  mutate(latitude = as.numeric(latitude), longitude= as.numeric(longitude)) %>% 
  filter(!is.na(latitude))
```

    ## Warning in mask$eval_all_mutate(quo): NAs introduced by coercion
    
    ## Warning in mask$eval_all_mutate(quo): NAs introduced by coercion

``` r
horses <- animal_rescues %>% 
  filter(special_service_type_category == "Animal rescue from water",
         animal_group_parent %in% c("Horse")) %>% 
  mutate(latitude = as.numeric(latitude), longitude= as.numeric(longitude)) %>% 
  filter(!is.na(latitude))
```

## ggmap approach

``` r
summary(as.numeric(animal_rescues$longitude))
```

    ## Warning in summary(as.numeric(animal_rescues$longitude)): NAs introduced by
    ## coercion

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
    ##  -0.560  -0.216  -0.102  -0.111  -0.011   0.466    3843

``` r
summary(as.numeric(animal_rescues$latitude))
```

    ## Warning in summary(as.numeric(animal_rescues$latitude)): NAs introduced by
    ## coercion

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
    ##    0.00   51.46   51.52   51.50   51.57   51.69    3843

``` r
london_loc <- c(bottom = 51.20000, top = 51.62000, left = -0.58000, right = 0.37000)
london <- get_stamenmap(bbox= london_loc, source="stamen", maptype="terrain", crop=FALSE)
```

    ## Source : http://tile.stamen.com/terrain/10/510/339.png

    ## Source : http://tile.stamen.com/terrain/10/511/339.png

    ## Source : http://tile.stamen.com/terrain/10/512/339.png

    ## Source : http://tile.stamen.com/terrain/10/513/339.png

    ## Source : http://tile.stamen.com/terrain/10/510/340.png

    ## Source : http://tile.stamen.com/terrain/10/511/340.png

    ## Source : http://tile.stamen.com/terrain/10/512/340.png

    ## Source : http://tile.stamen.com/terrain/10/513/340.png

    ## Source : http://tile.stamen.com/terrain/10/510/341.png

    ## Source : http://tile.stamen.com/terrain/10/511/341.png

    ## Source : http://tile.stamen.com/terrain/10/512/341.png

    ## Source : http://tile.stamen.com/terrain/10/513/341.png

``` r
p_lond <- ggmap(london) +
  geom_point(data = horses, 
             aes(x = longitude , y = latitude),
             alpha = 0.5,
             color = "blue",
             size = 1) +
  theme_void() +
  labs(title = "52 horses were rescued from water by the\nLondon fire brigade between 2009 and 2021",
       caption = "Data: London.gov | Map: Stamen | Graphic: @TannerFlorian") +
  theme(legend.position = "NULL",
        plot.title = element_text(size = 6, hjust = 0.5, vjust = 2),
        plot.caption = element_text(size = 4.5, hjust = 1))
```

``` r
ggsave(plot = p_lond, filename = "horses.png", device = "png",  units = "in", width = 4, height = 2.25, scale = 1, type = "cairo", dpi = 600)
```

``` r
sessionInfo()
```

    ## R version 4.0.3 (2020-10-10)
    ## Platform: x86_64-w64-mingw32/x64 (64-bit)
    ## Running under: Windows 10 x64 (build 18363)
    ## 
    ## Matrix products: default
    ## 
    ## locale:
    ## [1] LC_COLLATE=English_United States.1252 
    ## [2] LC_CTYPE=English_United States.1252   
    ## [3] LC_MONETARY=English_United States.1252
    ## [4] LC_NUMERIC=C                          
    ## [5] LC_TIME=English_United States.1252    
    ## 
    ## attached base packages:
    ## [1] stats     graphics  grDevices utils     datasets  methods   base     
    ## 
    ## other attached packages:
    ##  [1] ggmap_3.0.0     forcats_0.5.1   stringr_1.4.0   dplyr_1.0.6    
    ##  [5] purrr_0.3.4     readr_1.4.0     tidyr_1.1.3     tibble_3.1.2   
    ##  [9] ggplot2_3.3.3   tidyverse_1.3.1
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] Rcpp_1.0.5          lubridate_1.7.10    lattice_0.20-41    
    ##  [4] png_0.1-7           assertthat_0.2.1    digest_0.6.27      
    ##  [7] utf8_1.1.4          R6_2.5.0            cellranger_1.1.0   
    ## [10] plyr_1.8.6          backports_1.2.0     reprex_2.0.0       
    ## [13] evaluate_0.14       httr_1.4.2          pillar_1.6.1       
    ## [16] RgoogleMaps_1.4.5.3 rlang_0.4.10        curl_4.3           
    ## [19] readxl_1.3.1        rstudioapi_0.13     rmarkdown_2.6      
    ## [22] labeling_0.4.2      munsell_0.5.0       broom_0.7.6        
    ## [25] compiler_4.0.3      modelr_0.1.8        xfun_0.22          
    ## [28] pkgconfig_2.0.3     htmltools_0.5.1.1   tidyselect_1.1.1   
    ## [31] fansi_0.4.2         crayon_1.4.1        dbplyr_2.1.1       
    ## [34] withr_2.3.0         bitops_1.0-7        grid_4.0.3         
    ## [37] jsonlite_1.7.2      gtable_0.3.0        lifecycle_1.0.0    
    ## [40] DBI_1.1.0           magrittr_2.0.1      scales_1.1.1       
    ## [43] cli_2.5.0           stringi_1.5.3       farver_2.0.3       
    ## [46] fs_1.5.0            sp_1.4-5            xml2_1.3.2         
    ## [49] ellipsis_0.3.2      generics_0.1.0      vctrs_0.3.8        
    ## [52] rjson_0.2.20        tools_4.0.3         glue_1.4.2         
    ## [55] hms_1.0.0           jpeg_0.1-8.1        yaml_2.2.1         
    ## [58] colorspace_2.0-0    rvest_1.0.0         knitr_1.30         
    ## [61] haven_2.3.1
