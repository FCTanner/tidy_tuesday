2022-03-01 Alternative Fuel Stations
================
Florian Tanner
2022-03-02 09:21:06

``` r
rm(list = ls())

library(tidyverse)
```

    ## -- Attaching packages --------------------------------------- tidyverse 1.3.1 --

    ## v ggplot2 3.3.5     v purrr   0.3.4
    ## v tibble  3.1.4     v dplyr   1.0.7
    ## v tidyr   1.1.3     v stringr 1.4.0
    ## v readr   2.0.1     v forcats 0.5.1

    ## -- Conflicts ------------------------------------------ tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library(ggmap)
```

    ## Google's Terms of Service: https://cloud.google.com/maps-platform/terms/.

    ## Please cite ggmap if you use it! See citation("ggmap") for details.

``` r
library(gganimate)

library(showtext)
```

    ## Loading required package: sysfonts

    ## Loading required package: showtextdb

``` r
sysfonts::font_add_google("Roboto")
showtext_opts(dpi = 300)
showtext_auto(enable = TRUE)

font <- "Roboto"
```

# Read data

``` r
stations <- readr::read_csv('https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2022/2022-03-01/stations.csv') 
```

    ## Warning: One or more parsing issues, see `problems()` for details

    ## Rows: 59927 Columns: 70

    ## -- Column specification --------------------------------------------------------
    ## Delimiter: ","
    ## chr (43): FUEL_TYPE_CODE, STATION_NAME, STREET_ADDRESS, INTERSECTION_DIRECTI...
    ## dbl (15): X, Y, OBJECTID, EV_LEVEL1_EVSE_NUM, EV_LEVEL2_EVSE_NUM, EV_DC_FAST...
    ## lgl (12): PLUS4, EV_OTHER_INFO, HYDROGEN_STATUS_LINK, LPG_PRIMARY, E85_BLEND...

    ## 
    ## i Use `spec()` to retrieve the full column specification for this data.
    ## i Specify the column types or set `show_col_types = FALSE` to quiet this message.

# Ideas

-   For each state, what is the availability of alternative fuel
    stations
-   Density
-   Average distance to stations
-   Adjust for population? Adjust for size of the state?

# Notes

-   Inspiration <https://www.littlemissdata.com/blog/maps>

-   Need to filter for

    -   access\_code == “public”

-   cng\_renewable\_source + lng\_renewable\_source +
    ev\_renewable\_source

    -   This is interesting: How many stations can do it?

-   Vast majority of stations is for electric

-   group by city or state?

# EDA

``` r
public_access <- stations |>  
  janitor::clean_names() |> 
  filter(access_code == "public") |> 
  mutate(fuel_type= case_when(fuel_type_code == "ELEC" ~ "Electric",
                              fuel_type_code == "BD" ~ "Biodiesel (B20 and above)",
                              fuel_type_code == "CNG" ~ "Compressed Natural Gas (CNG)",
                              fuel_type_code == "E85" ~ "Ethanol (E85)",
                              fuel_type_code == "HY" ~ "Hydrogen",
                              fuel_type_code == "LNG" ~ "Liquefied Natural Gas (LNG)",
                              fuel_type_code == "LPG" ~ "Propane (LPG)"))
```

### How many are planned / expected?

``` r
table(public_access$status_code)
```

    ## 
    ##     E     P     T 
    ## 54298   193   139

``` r
table(public_access$fuel_type_code)
```

    ## 
    ##    BD   CNG   E85  ELEC    HY   LNG   LPG 
    ##   327   866  4142 46544   119   102  2530

# Maps of individual stations

``` r
us_loc <- c(left = -125, bottom = 25, right = -65, top = 49)
us <- get_stamenmap(bbox= us_loc, source="stamen", maptype="toner-lite", crop=FALSE, zoom = 6)
```

    ## 84 tiles needed, this may take a while (try a smaller zoom).

    ## Source : http://tile.stamen.com/toner-lite/6/9/21.png

    ## Source : http://tile.stamen.com/toner-lite/6/10/21.png

    ## Source : http://tile.stamen.com/toner-lite/6/11/21.png

    ## Source : http://tile.stamen.com/toner-lite/6/12/21.png

    ## Source : http://tile.stamen.com/toner-lite/6/13/21.png

    ## Source : http://tile.stamen.com/toner-lite/6/14/21.png

    ## Source : http://tile.stamen.com/toner-lite/6/15/21.png

    ## Source : http://tile.stamen.com/toner-lite/6/16/21.png

    ## Source : http://tile.stamen.com/toner-lite/6/17/21.png

    ## Source : http://tile.stamen.com/toner-lite/6/18/21.png

    ## Source : http://tile.stamen.com/toner-lite/6/19/21.png

    ## Source : http://tile.stamen.com/toner-lite/6/20/21.png

    ## Source : http://tile.stamen.com/toner-lite/6/9/22.png

    ## Source : http://tile.stamen.com/toner-lite/6/10/22.png

    ## Source : http://tile.stamen.com/toner-lite/6/11/22.png

    ## Source : http://tile.stamen.com/toner-lite/6/12/22.png

    ## Source : http://tile.stamen.com/toner-lite/6/13/22.png

    ## Source : http://tile.stamen.com/toner-lite/6/14/22.png

    ## Source : http://tile.stamen.com/toner-lite/6/15/22.png

    ## Source : http://tile.stamen.com/toner-lite/6/16/22.png

    ## Source : http://tile.stamen.com/toner-lite/6/17/22.png

    ## Source : http://tile.stamen.com/toner-lite/6/18/22.png

    ## Source : http://tile.stamen.com/toner-lite/6/19/22.png

    ## Source : http://tile.stamen.com/toner-lite/6/20/22.png

    ## Source : http://tile.stamen.com/toner-lite/6/9/23.png

    ## Source : http://tile.stamen.com/toner-lite/6/10/23.png

    ## Source : http://tile.stamen.com/toner-lite/6/11/23.png

    ## Source : http://tile.stamen.com/toner-lite/6/12/23.png

    ## Source : http://tile.stamen.com/toner-lite/6/13/23.png

    ## Source : http://tile.stamen.com/toner-lite/6/14/23.png

    ## Source : http://tile.stamen.com/toner-lite/6/15/23.png

    ## Source : http://tile.stamen.com/toner-lite/6/16/23.png

    ## Source : http://tile.stamen.com/toner-lite/6/17/23.png

    ## Source : http://tile.stamen.com/toner-lite/6/18/23.png

    ## Source : http://tile.stamen.com/toner-lite/6/19/23.png

    ## Source : http://tile.stamen.com/toner-lite/6/20/23.png

    ## Source : http://tile.stamen.com/toner-lite/6/9/24.png

    ## Source : http://tile.stamen.com/toner-lite/6/10/24.png

    ## Source : http://tile.stamen.com/toner-lite/6/11/24.png

    ## Source : http://tile.stamen.com/toner-lite/6/12/24.png

    ## Source : http://tile.stamen.com/toner-lite/6/13/24.png

    ## Source : http://tile.stamen.com/toner-lite/6/14/24.png

    ## Source : http://tile.stamen.com/toner-lite/6/15/24.png

    ## Source : http://tile.stamen.com/toner-lite/6/16/24.png

    ## Source : http://tile.stamen.com/toner-lite/6/17/24.png

    ## Source : http://tile.stamen.com/toner-lite/6/18/24.png

    ## Source : http://tile.stamen.com/toner-lite/6/19/24.png

    ## Source : http://tile.stamen.com/toner-lite/6/20/24.png

    ## Source : http://tile.stamen.com/toner-lite/6/9/25.png

    ## Source : http://tile.stamen.com/toner-lite/6/10/25.png

    ## Source : http://tile.stamen.com/toner-lite/6/11/25.png

    ## Source : http://tile.stamen.com/toner-lite/6/12/25.png

    ## Source : http://tile.stamen.com/toner-lite/6/13/25.png

    ## Source : http://tile.stamen.com/toner-lite/6/14/25.png

    ## Source : http://tile.stamen.com/toner-lite/6/15/25.png

    ## Source : http://tile.stamen.com/toner-lite/6/16/25.png

    ## Source : http://tile.stamen.com/toner-lite/6/17/25.png

    ## Source : http://tile.stamen.com/toner-lite/6/18/25.png

    ## Source : http://tile.stamen.com/toner-lite/6/19/25.png

    ## Source : http://tile.stamen.com/toner-lite/6/20/25.png

    ## Source : http://tile.stamen.com/toner-lite/6/9/26.png

    ## Source : http://tile.stamen.com/toner-lite/6/10/26.png

    ## Source : http://tile.stamen.com/toner-lite/6/11/26.png

    ## Source : http://tile.stamen.com/toner-lite/6/12/26.png

    ## Source : http://tile.stamen.com/toner-lite/6/13/26.png

    ## Source : http://tile.stamen.com/toner-lite/6/14/26.png

    ## Source : http://tile.stamen.com/toner-lite/6/15/26.png

    ## Source : http://tile.stamen.com/toner-lite/6/16/26.png

    ## Source : http://tile.stamen.com/toner-lite/6/17/26.png

    ## Source : http://tile.stamen.com/toner-lite/6/18/26.png

    ## Source : http://tile.stamen.com/toner-lite/6/19/26.png

    ## Source : http://tile.stamen.com/toner-lite/6/20/26.png

    ## Source : http://tile.stamen.com/toner-lite/6/9/27.png

    ## Source : http://tile.stamen.com/toner-lite/6/10/27.png

    ## Source : http://tile.stamen.com/toner-lite/6/11/27.png

    ## Source : http://tile.stamen.com/toner-lite/6/12/27.png

    ## Source : http://tile.stamen.com/toner-lite/6/13/27.png

    ## Source : http://tile.stamen.com/toner-lite/6/14/27.png

    ## Source : http://tile.stamen.com/toner-lite/6/15/27.png

    ## Source : http://tile.stamen.com/toner-lite/6/16/27.png

    ## Source : http://tile.stamen.com/toner-lite/6/17/27.png

    ## Source : http://tile.stamen.com/toner-lite/6/18/27.png

    ## Source : http://tile.stamen.com/toner-lite/6/19/27.png

    ## Source : http://tile.stamen.com/toner-lite/6/20/27.png

``` r
p_static <- ggmap(us) + 
  geom_point(data = public_access, aes(x = x, y =y, color = fuel_type_code), size = 0.2)+ 
  ggsci::scale_color_aaas() + 
  facet_wrap(~fuel_type, ncol = 2) +
  theme_void() +
  theme(legend.position = "none",
        text = element_text(family = font),
        strip.text.x = element_text(size = rel(1.7), family = font))
```

``` r
ggsave(plot = p_static, filename = "fuel_static.png", units = "cm", height = 20, width= 20, limitsize = F, dpi = 600,scale= 0.7 )
```

    ## Warning: Removed 434 rows containing missing values (geom_point).

``` r
sessionInfo()
```

    ## R version 4.1.2 (2021-11-01)
    ## Platform: x86_64-w64-mingw32/x64 (64-bit)
    ## Running under: Windows 10 x64 (build 19044)
    ## 
    ## Matrix products: default
    ## 
    ## locale:
    ## [1] LC_COLLATE=English_Australia.1252  LC_CTYPE=English_Australia.1252   
    ## [3] LC_MONETARY=English_Australia.1252 LC_NUMERIC=C                      
    ## [5] LC_TIME=English_Australia.1252    
    ## 
    ## attached base packages:
    ## [1] stats     graphics  grDevices utils     datasets  methods   base     
    ## 
    ## other attached packages:
    ##  [1] showtext_0.9-4  showtextdb_3.0  sysfonts_0.8.5  gganimate_1.0.7
    ##  [5] ggmap_3.0.0     forcats_0.5.1   stringr_1.4.0   dplyr_1.0.7    
    ##  [9] purrr_0.3.4     readr_2.0.1     tidyr_1.1.3     tibble_3.1.4   
    ## [13] ggplot2_3.3.5   tidyverse_1.3.1
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] bitops_1.0-7        fs_1.5.0            lubridate_1.7.10   
    ##  [4] bit64_4.0.5         progress_1.2.2      httr_1.4.2         
    ##  [7] ggsci_2.9           tools_4.1.2         backports_1.2.1    
    ## [10] utf8_1.2.2          R6_2.5.1            DBI_1.1.1          
    ## [13] colorspace_2.0-2    withr_2.4.3         sp_1.4-5           
    ## [16] tidyselect_1.1.1    prettyunits_1.1.1   bit_4.0.4          
    ## [19] curl_4.3.2          compiler_4.1.2      cli_3.0.1          
    ## [22] rvest_1.0.1         xml2_1.3.2          labeling_0.4.2     
    ## [25] scales_1.1.1        digest_0.6.27       rmarkdown_2.10     
    ## [28] jpeg_0.1-9          pkgconfig_2.0.3     htmltools_0.5.2    
    ## [31] dbplyr_2.1.1        fastmap_1.1.0       rlang_0.4.11       
    ## [34] readxl_1.3.1        rstudioapi_0.13     farver_2.1.0       
    ## [37] generics_0.1.0      jsonlite_1.7.2      vroom_1.5.4        
    ## [40] magrittr_2.0.1      Rcpp_1.0.7          munsell_0.5.0      
    ## [43] fansi_0.5.0         lifecycle_1.0.0     stringi_1.7.4      
    ## [46] yaml_2.2.1          snakecase_0.11.0    plyr_1.8.6         
    ## [49] grid_4.1.2          parallel_4.1.2      crayon_1.4.1       
    ## [52] lattice_0.20-45     haven_2.4.3         hms_1.1.0          
    ## [55] knitr_1.34          pillar_1.6.2        rjson_0.2.21       
    ## [58] reprex_2.0.1        glue_1.4.2          evaluate_0.14      
    ## [61] gifski_1.4.3-1      modelr_0.1.8        png_0.1-7          
    ## [64] vctrs_0.3.8         tzdb_0.1.2          tweenr_1.0.2       
    ## [67] RgoogleMaps_1.4.5.3 cellranger_1.1.0    gtable_0.3.0       
    ## [70] assertthat_0.2.1    xfun_0.25           janitor_2.1.0      
    ## [73] broom_0.7.9         ellipsis_0.3.2
