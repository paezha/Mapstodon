
<!-- README.md is generated from README.Rmd. Please edit that file -->

# Mauna Loa

In this notebook I create a contour map of shield volcano Mauna Loa in
Hawaii. The map uses open data and [Tanaka
contours](http://wiki.gis.com/wiki/index.php/Tanaka_contours).

I will use the following packages:

``` r
library(elevatr) # To get elevation data
#> Warning: package 'elevatr' was built under R version 4.2.2
library(MexBrewer) # Color palettes
library(metR) # For contouring 
#> Warning: package 'metR' was built under R version 4.2.2
library(showtext) # For importing fonts
library(terra) # To work with rasters
library(tidyverse) # Data carpentry and ggplot2
```

## Load fonts

Import Google Fonts:

``` r
font_add_google("Amatic SC", "pressstart")

showtext_auto()
```

## Retrieve open elevation data

Use {elevatr} to get elevation data:

``` r

maunaloa <- get_elev_raster(locations = data.frame(x = c(-155.522829, -155.682829), 
                                               y = c(19.329488, 19.649488)),
                        z = 10, 
                        prj = "EPSG:4326",
                        clip = "locations")
#> Mosaicing & Projecting
#> Clipping DEM to locations
#> Note: Elevation units are in meters.
```

Convert to {terra} `SpatRaster` object and thereof to data frame:

``` r
maunaloa <- rast(maunaloa)

maunaloa <- as.data.frame(maunaloa, 
                      xy = TRUE) %>%
  rename(elev = 3)
```

Plot contours; use a palette from
[{MexBrewer}](https://github.com/paezha/MexBrewer):

``` r
cols <- mex.brewer("Alacena", n = 11)

ggplot(data = maunaloa,
       aes(x, y, z = elev)) +
  geom_contour_filled(breaks = seq(min(maunaloa$elev), 
                                   max(maunaloa$elev), 
                                   length.out = 11)) +
  #geom_contour_tanaka() + 
  scale_fill_manual(values = cols) +
  coord_equal()
```

![](README_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

Repeat plot but now with Tanaka countours:

``` r
ggplot(data = maunaloa,
       aes(x, y, z = elev)) +
  geom_contour_filled(breaks = seq(min(maunaloa$elev), 
                                   max(maunaloa$elev), 
                                   length.out = 11)) +
  geom_contour_tanaka(sun.angle = 60, 
                      smooth = 10) + 
  scale_fill_manual(values = cols) +
  ggtitle("MAUNA LOA") +
  labs(subtitle = "Hawaii", caption = "@paezha@mastodon.online") +
  coord_equal() +
  theme_void() + 
  theme(legend.position = "none",
        plot.background = element_rect(color = NA,
                                       fill = cols[11]),
        plot.title = element_text(color = "white",
                                  face = "bold",
                                  size = 100, 
                                  hjust = 0.5,
                                  vjust = -4),
        plot.subtitle = element_text(color = "white",
                                     size = 80, 
                                     hjust = 0.5,
                                     vjust = -149),
        plot.caption = element_text(color = "white",
                                    size = 40,
                                    hjust = 0.9,
                                    vjust = 14))

ggsave("maunaloa-tanaka-contours.png",
       #width = 8,
       height = 12,
       units = "in")
#> Saving 7 x 12 in image
```

![](maunaloa-tanaka-contours.png)
