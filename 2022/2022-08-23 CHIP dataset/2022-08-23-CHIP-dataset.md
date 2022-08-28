2022-08-23 CHIP dataset
================
Florian Tanner
2022-08-26 15:48:07

``` r
rm(list = ls())

library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.2 ──
    ## ✔ ggplot2 3.3.6     ✔ purrr   0.3.4
    ## ✔ tibble  3.1.8     ✔ dplyr   1.0.9
    ## ✔ tidyr   1.2.0     ✔ stringr 1.4.0
    ## ✔ readr   2.1.2     ✔ forcats 0.5.1
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

``` r
library(ggtext)
library(ggrepel)
library(png)
library(patchwork)
library(showtext)
```

    ## Loading required package: sysfonts
    ## Loading required package: showtextdb

``` r
sysfonts::font_add_google("Roboto Condensed")
showtext::showtext_auto()
showtext::showtext_opts(dpi = 300)
```

``` r
chips_full <- read_csv("chip_dataset.csv") |> 
  janitor::clean_names()
```

    ## New names:
    ## Rows: 4854 Columns: 14
    ## ── Column specification
    ## ──────────────────────────────────────────────────────── Delimiter: "," chr
    ## (5): Product, Type, Release Date, Foundry, Vendor dbl (9): ...1, Process Size
    ## (nm), TDP (W), Die Size (mm^2), Transistors (mil...
    ## ℹ Use `spec()` to retrieve the full column specification for this data. ℹ
    ## Specify the column types or set `show_col_types = FALSE` to quiet this message.
    ## • `` -> `...1`

``` r
nvidia_logo <- grid::rasterGrob(readPNG("nvidia_logo_crop.png"))
```

``` r
data <- chips_full |> 
  filter(type == "GPU",
         vendor == "NVIDIA",
         str_detect(product, pattern = c("RTX | GTX|GT")),
         release_date != "NaT") |> 
  mutate(year = substr(release_date, 1, 4),
         product_line = case_when(str_detect(product, "GT") & 
                                    str_detect(product, "GTX", negate = T) &
                                    str_detect(product, "GTS", negate = T)
                                  ~ "GT",
                                  str_detect(product, "GTX") ~ "GTX",
                                  str_detect(product, "RTX ") ~ "RTX",
                                  TRUE ~ "other"),
         release_date = as.Date(release_date),
         product_label = str_remove(product, "NVIDIA "),
         product_label = str_remove(product_label, "GeForce "))
```

``` r
theme_nvidia <- 
  theme_bw(base_size = 40, base_family = "Roboto Condensed") +
  theme(panel.background = element_rect(fill = "#76b900", color = "#76b900"),
        panel.grid = element_blank(),
        plot.background = element_rect(fill = "#76b900", color = "#76b900"),
        axis.title.x = element_text(family = "Roboto Condensed"),
        axis.title.y = element_text(family = "Roboto Condensed"),
        plot.title = element_blank(),
        plot.caption =element_textbox_simple(size = 22 ,lineheight = 1.2,
                                             padding = margin(5.5, 5.5, 5.5, 5.5),
                                             margin = margin(25, 0, 0, 0),
                                             maxheight = NULL),
        legend.position = "none") 

nvidia_scheme <- c("#F5F5F5", "#B3CAE7", "#7598C1")
```

``` r
nvidia_title <- "FP32 GPU performance of <span style = 'color: #F5F5F5;font-weight:bold;'>GT</span>,<br><span style = 'color: #B3CAE7;font-weight:bold;'>GTX</span> and <span style = 'color: #7598C1;font-weight:bold;'>RTX</span> series has doubled<br>approximately every two years"

nvidia_caption <-  "Data: Sun, Yifan, et al. Summarizing CPU and GPU design trends with product data. arXiv preprint arXiv:1911.11313 (2019). | Graphic: @TannerFlorian"
```

``` r
p <- data |> 
  filter(product_line != "other") |> 
  ggplot(aes(x = release_date, y = fp32_gflops, color = product_line )) +
  geom_point(color = "black", size = 3) +
  geom_point(size = 2.5) +
  ggrepel::geom_label_repel(aes(label=product_label, fill = product_line), color= "black", 
                            family = "Roboto Condensed") +
  scale_color_manual(values = nvidia_scheme) +
  scale_fill_manual(values = nvidia_scheme) +
  scale_x_date(limits = as.Date(c("2006-01-01", "2022-01-01"))) +
  scale_y_log10() +
  geom_richtext(aes(x = as.Date("2006-01-01"), y = 20000, label = nvidia_title),
                fill = NA, label.color = NA, size = 23, hjust = 0, color = "black",
                family = "Roboto Condensed", fontface= "bold", lineheight =1.3) +
  labs(y = "FP32 GFLOPS, log scale", x = "Release date", caption= nvidia_caption)



p <- p +
  inset_element(nvidia_logo, left = 0.7,
                bottom = 0.001,
                right = 0.99,
                top = 0.3) & theme_nvidia
```

``` r
gt_gtx_rtx_gr <-data |> 
  filter(product_line != "other") |> 
  group_by(year) |> 
  summarise(mean_yearly_fp32_gflops = mean(fp32_gflops, na.rm = T)) |> 
  filter(is.numeric(mean_yearly_fp32_gflops)) |> 
  mutate(ann_growth = (lead(mean_yearly_fp32_gflops) - mean_yearly_fp32_gflops)/mean_yearly_fp32_gflops) |> 
  summarise(mean_gr = mean(ann_growth, na.rm = T)) |> 
  ungroup() |> 
  mutate(mean_doubling_time = log(2)/log(1+mean_gr))
```

``` r
ggsave(plot = p, filename = "large.png", units = "cm", width = 60, height = 60, limitsize = F, device = "png")
```

    ## Warning: Removed 28 rows containing missing values (geom_point).
    ## Removed 28 rows containing missing values (geom_point).

    ## Warning: Removed 28 rows containing missing values (geom_label_repel).

    ## Warning: ggrepel: 82 unlabeled data points (too many overlaps). Consider
    ## increasing max.overlaps

``` r
sessionInfo()
```

    ## R version 4.2.1 (2022-06-23 ucrt)
    ## Platform: x86_64-w64-mingw32/x64 (64-bit)
    ## Running under: Windows 10 x64 (build 19044)
    ## 
    ## Matrix products: default
    ## 
    ## locale:
    ## [1] LC_COLLATE=English_Australia.utf8  LC_CTYPE=English_Australia.utf8   
    ## [3] LC_MONETARY=English_Australia.utf8 LC_NUMERIC=C                      
    ## [5] LC_TIME=English_Australia.utf8    
    ## 
    ## attached base packages:
    ## [1] stats     graphics  grDevices utils     datasets  methods   base     
    ## 
    ## other attached packages:
    ##  [1] showtext_0.9-5  showtextdb_3.0  sysfonts_0.8.8  patchwork_1.1.1
    ##  [5] png_0.1-7       ggrepel_0.9.1   ggtext_0.1.1    forcats_0.5.1  
    ##  [9] stringr_1.4.0   dplyr_1.0.9     purrr_0.3.4     readr_2.1.2    
    ## [13] tidyr_1.2.0     tibble_3.1.8    ggplot2_3.3.6   tidyverse_1.3.2
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] Rcpp_1.0.9          lubridate_1.8.0     assertthat_0.2.1   
    ##  [4] digest_0.6.29       utf8_1.2.2          R6_2.5.1           
    ##  [7] cellranger_1.1.0    backports_1.4.1     reprex_2.0.1       
    ## [10] evaluate_0.16       httr_1.4.3          pillar_1.8.0       
    ## [13] rlang_1.0.4         curl_4.3.2          googlesheets4_1.0.0
    ## [16] readxl_1.4.0        rstudioapi_0.13     rmarkdown_2.14     
    ## [19] googledrive_2.0.0   bit_4.0.4           munsell_0.5.0      
    ## [22] gridtext_0.1.4      broom_1.0.0         janitor_2.1.0      
    ## [25] compiler_4.2.1      modelr_0.1.8        xfun_0.32          
    ## [28] pkgconfig_2.0.3     htmltools_0.5.3     tidyselect_1.1.2   
    ## [31] fansi_1.0.3         crayon_1.5.1        tzdb_0.3.0         
    ## [34] dbplyr_2.2.1        withr_2.5.0         grid_4.2.1         
    ## [37] jsonlite_1.8.0      gtable_0.3.0        lifecycle_1.0.1    
    ## [40] DBI_1.1.3           magrittr_2.0.3      scales_1.2.0       
    ## [43] vroom_1.5.7         cli_3.3.0           stringi_1.7.8      
    ## [46] farver_2.1.1        fs_1.5.2            snakecase_0.11.0   
    ## [49] xml2_1.3.3          ellipsis_0.3.2      generics_0.1.3     
    ## [52] vctrs_0.4.1         tools_4.2.1         bit64_4.0.5        
    ## [55] glue_1.6.2          markdown_1.1        hms_1.1.1          
    ## [58] parallel_4.2.1      fastmap_1.1.0       yaml_2.3.5         
    ## [61] colorspace_2.0-3    gargle_1.2.0        rvest_1.0.2        
    ## [64] knitr_1.39          haven_2.5.0
