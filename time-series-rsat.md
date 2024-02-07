Downloading & Processing Data Cubes: `rsat.pkg`
================

### Data Sources

So far, the `rsat` packaged has proved most reliable. Better at
accessing, scanning and downloading, though it doesnt include a quick
download option and tends to involve longer processing and coding time.
In-built vignettes seem to be best learning resources
(i.e. `rsat::rsat1_search`).

In the following chunk, we attempt a sort of ‘sign-in’ blitz. This is a
crude attempt to solve ongoing connection issues between `rsat`
functions and their registered *APIs*. Two NASA APIs (USGS-eros &
Earthdata) reported in package documentation. However, source code shows
edits in package updates and github commits. Though, the main issue
seems to be the constant maintenance and reformatting of API end-points
by NASA & USGS. See below some details on connection tests, make sure to
change your username and passwords where suggested. More on GDAL
configuration for passive computing in
[gdal_cubes.md](https://github.com/seamusrobertmurphy/time-series-gdalcubes/blob/main/time-series-gdalcubes.md)

``` r
library(rsat)
# rsat::set_credentials("seamusrobertmurphy@gmail.com", "password") working username!

library(espa.tools)
espa.tools::espa_inventory_login(
  usgs_eros_username, 
  usgs_eros_password, 
  verbose = F)

library(earthdatalogin) 
edl_netrc(
  username = "username_here",
  password = "password_here",
  cloud_config = T # see gdal_cubes.md for GDAL config of cloud computing/links
)
```

Most reliable function from `getSpatialData` that promtps a PAP sign-in
method:

``` r
library(getSpatialData) ## is working ???
getSpatialData::login_earthdata(verbose=T)
getSpatialData::login_USGS(verbose=T)
services()
```

### Loading Data Libraries

``` r
library(sf)
library(tmap)
# search paramters
ip <- st_sf(st_as_sfc(st_bbox(c(
  xmin = 35.09583,
  xmax = 36.29583,
  ymin = -15.90890,
  ymax = -14.77500 
), crs = 4326)))
toi_2018 <- seq(as.Date("2018-10-01"),as.Date("2021-12-30"),1)

# scan all collections 
rsat_products()

# confirm available data of study site/window
browse = rsat_search(
  region = ip,
  dates = toi_2018,
  product = c(
    "landsat_ot_c2_l1", 
    "landsat_etm_c2_l1", 
    "landsat_tm_c2_l1"),
  verbose=F) # NOTE: tm unavailable for toi_2018  

# assign location for downloads 
db.path <- "/media/seamus/Ubuntu 22_04 LTS amd64/rsat/edits/database"
ds.path <- "/media/seamus/Ubuntu 22_04 LTS amd64/rsat/edits/datasets"
dir.create(db.path)
dir.create(ds.path)
```

Derive `rtoi` filter object for bugfree functions. This spatial fileter
object assigns download path for files and metadata, and allows to
restore and rapid plotting even after connection breaks. NOTE: better to
recall previous `rtoi` or can following chunk if previous `rtoi` was
purged.

``` r
# derive 'rtoi' object for spatial filter and handling results
chilwa_2018 <- new_rtoi(
  name = "chilwa_2018",
  region = ip,
  db_path = db.path,
  rtoi_path = ds.path)
```

``` r
# apply temporal filter to rtoi collection
rsat_search(
  region = chilwa_2018, 
  product = c("landsat_ot_c2_l1"), 
  dates = toi_2018)
```

### Quickfire download (w/o processing)

``` r
# download to rtoi-assigned folder
rsat_download(chilwa_2018)
# test_function(rsat_download)

# apply mosaic-clip-compression function
rsat_mosaic(chilwa_2018,
  out_path = "/media/seamus/Ubuntu 22_04 LTS amd64/rsat/edits/mosaics/", # default to  
  warp = "extent",
  region = ip,
  overwrite = T)

# check results
rsat::plot(chilwa_2018, 
  as.Date("2018-01-11"), 
  band_name = c("nir", "red"))
```

``` r
# convert to spatRaster
chilwa_2018_ndvi_rast = rsat_get_SpatRaster(
  chilwa_2018, "landsat_ot_c2_l1", "NDVI")

# convert to RasterStack
chilwa_2018_ndvi_raster = rsat_get_raster(
  chilwa_2018, "landsat_ot_c2_l1", "NDVI")

# convert to stars
chilwa_2018_ndvi_stars = rsat_get_stars(
  chilwa_2018, "landsat_ot_c2_l1", "NDVI")

# extract single band in above formats
rsat_list_data(chilwa_2018)
# chilwa_2018_nir = rsat_get_SpatRaster(chilwa_2018,
#  "landsat_ot_c2_l1",
#  "LC81980182015183LGN00_B08")
```

### Custom band math

``` r
# derive custom function 
NDVI = function(nir08, red){
  ndvi <- (nir08 - red)/(nir08 + red)
  return(ndvi)
}

# apply function to bands in rtoi collection
rsat_derive(chilwa_2018, 
  product = c("landsat_ot_c2_l1"),
  variable = "ndvi", fun = NDVI)

# check results 
plot(chilwa_2018,
     as.Date("2018-11-11"),
     variable = "ndvi",
     xsize = 500,
     ysize = 500,
     zlim = c(-1,1))
```

``` r
# alternatively, use pre-loaded spectral index
show_variables() 

# replace with with variable of choice
rsat_derive(chilwa_2018, "NDWI", product = "landsat_ot_c2_l1")
rsat_list_data(chilwa_2018)

plot(chilwa_2018, as.Date("2018-11-11"),
     variable = "ndvi",
     xsize = 500,
     ysize = 500,
     zlim = c(-1,1))
```

### Cloud masking

``` r
# register QA_Quality bands
rsat_cloudMask(chilwa_2018)

# derive NDSI layer and truncate to -1:1 
ndsi.img <- rsat_get_raster(chilwa_2018, "landsat_ot_c2_l1", "ndsi")
ndsi.img <- clamp(ndsi.img, -1, 1)

# load function and mask results
clds.msk <- rsat_get_raster(chilwa_2018, "landsat_ot_c2_l1", "CloudMask")
```
