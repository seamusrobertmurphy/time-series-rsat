Downloading & Processing Data Cubes: `rsat.pkg`
================

### Data Sources

So far, the `rsat` packaged has proved most reliable in successfully
accessing, identifying, processing and downloading time-series data
cubes. In addition, as the package was written by remote sensing experts
with remote sensing experts in mind, the package provides a far richer
and more sophisticated set of functions and customizable tools. As a
result however, it involves longer processing and coding time.

``` r
library(getSpatialData) 
login_USGS(username = "seamusrobertmurphy@gmail.com")
login_CopHub(username = "seamusrobertmurphy@gmail.com")
login_earthdata(username = "seamusrobertmurphy@gmail.com")
services()
```

``` r
library(rsat)
set_credentials("username","password", "earthdata")
```

### Loading Data Libraries

``` r
library(sf)
library(tmap)
roi = sf::read_sf("~/git_repos/remote-sensing-lake-chilwa/roi/chilwa_watershed_4326.shp")
toi <- as.Date("2021-01-01") + 0:15
# Visualize aoi
tmap_mode("view")
tm_shape(st_geometry(roi)) +  
  tm_polygons()

rcd <- rsat_search(
  region = roi, dates = toi,
  product = c("LANDSAT_ETM_C1", "LANDSAT_ETM_C1", "LANDSAT_TM_C1"))

class(rcd)
rsat::rsat_search()
```

You can also embed plots, for example:

Note that the `echo = FALSE` parameter was added to the code chunk to
prevent printing of the R code that generated the plot.
