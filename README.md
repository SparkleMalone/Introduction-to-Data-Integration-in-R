# Introduction-to-data-Integration-in-R

Data analysis often requires combining data from multiple sources, such as files, databases, APIs, or web scraping. R is a powerful and flexible tool for data integration, but it can also pose some challenges and pitfalls. In this workshop, you will learn some of the best ways to integrate data from multiple sources in R.

#### Workshop Goals: 

1. Understand techniques for data Integration. 
2. Obtain various data from different sources.
3. Combine tabular and vector data.
4. Combine tabular and raster data. 
4. Combine tabular data by an ID and time.

#### Choose the right package
R has many packages that can help you import, merge, and manipulate data from different sources. Some of the most popular and useful ones for tables include readr, dplyr, tidyr, and purrr. These packages are part of the tidyverse, a collection of packages that share a consistent and coherent syntax and philosophy for data analysis. For spatial data, the sf and terra packages are useful.

Type of data| Library
|------:|-----------|
|Tabular| tidyverse|
| Vector| sf|
| Raster | terra|

#### Use pipes and functions
One of the best features of the tidyverse is the pipe operator (%>%), which allows you to chain multiple functions together and pass the output of one function as the input of the next one. This way, you can create a data integration pipeline that is clear and logical, and that avoids intermediate variables and nested functions.

#### Check and validate data

Data integration can introduce errors or inconsistencies in your data, such as duplicates, mismatches, outliers, or invalid values. Therefore, it is important to check and validate your data before and after you integrate it from multiple sources. You can use various tools and techniques to do this, such as summary statistics, data visualization, data profiling, or data quality rules.

In this workshop you will begin to extract information about the FLUXNET CH4 tower sites.

# Load Libraries:
```{r, include=T}
library(sf)
library(terra)
library(tidyverse)
library(ggplot2)
library(AOI)
```

# Integrating information from simple features

Import the file FluxNet_Sites_2024.csv and call it FluxNet

```{r, include=T}
FluxNet <- read.csv('data/FluxNet_Sites_2024.csv')
```
This dataset includes:

 Column Name | Description |
|------:|-----------|
|SITE_ID| Unique site id|
|SITE_NAME|Site name|
|FLUXNET2015|License information for the data for the two FLUXNET Products|
|FLUXNET-CH4|License information for the data for the two FLUXNET Products|
|LOCATION_LAT|Location information|
|LOCATION_LONG|Location information|
|LOCATION_ELEV|Elevation in meters|
|IGBP|Vegetation type|
|MAT| Mean annual temperature in Celsius|
|MAP| Mean annual precipitation in mm|

Subset file to incluse only sites measuring methane:
```{r, include=T}
FLUXNET.CH4 <- FluxNet %>% filter( FLUXNET.CH4 == "CC-BY-4.0" )
```
Now that you have all of the sites measuring methane, convert FLUXNET.CH4 to a sf and call it FLUXNET.CH4.shp:

```{r, include=T}
FLUXNET.CH4.shp <- st_as_sf(x = FLUXNET.CH4,                         
           coords = c("LOCATION_LONG",  "LOCATION_LAT"),
           crs = 4326)

ggplot(data=FLUXNET.CH4.shp ) + geom_sf()
```
check the class and that the geometry is valid:
```{r, include=T}
class(FLUXNET.CH4.shp)
st_is_valid(FLUXNET.CH4.shp)
```

Create a global sf and extract the country into the sf

```{r, include=T}

global <- aoi_get(country= c("Europe","Asia" ,"North America", "South America", "Australia","Africa", "New Zealand"))
```

Make the CRS match:
```{r, include=T}

FLUXNET.CH4.shp = st_transform(FLUXNET.CH4.shp, crs= '+init=epsg:4087')

global = st_transform(global, crs= '+init=epsg:4087') %>% st_make_valid()

ggplot() + geom_sf(data = global) + geom_sf(data = FLUXNET.CH4.shp) 

```

Use the st_intersect() to extract the country of each tower site:
```{r, include=T}
FLUXNET.CH4.shp$Country <- st_intersection( global, FLUXNET.CH4.shp)$name

FLUXNET.CH4.shp$Country 
```

# Integrating information from rasters
Import the file GlobalSoil_grids.tif :

```{r, include=T}
soil <- terra::rast("data/GlobalSoil_grids.tif" )
soil
```
Transform FLUXNET.CH4.shp to the same CRS as soils:
```{r, include=T}
FLUXNET.CH4.shp = st_transform(FLUXNET.CH4.shp, crs= crs(soil))
FLUXNET.CH4.shp
```
Extract soil information to FLUXNET.CH4.shp:
```{r, include=T}
FLUXNET.CH4.shp$SOIL_BulkDensity = terra::extract(soil, FLUXNET.CH4.shp)$BulkDensity
FLUXNET.CH4.shp$SOIL_BulkDensity

FLUXNET.CH4.shp$SOIL_PH = terra::extract(soil, FLUXNET.CH4.shp)$PH
FLUXNET.CH4.shp$SOIL_PH

FLUXNET.CH4.shp$SOIL_Nitrogen = terra::extract(soil, FLUXNET.CH4.shp)$Nitrogen
FLUXNET.CH4.shp$SOIL_Nitrogen
```

Import the climate information (GlobalClimate.tif) :
```{r, include=T}
climate <- terra::rast("data/GlobalClimate.tif" )
climate
```
Transform FLUXNET.CH4.shp to the same CRS as climate:
```{r, include=T}
FLUXNET.CH4.shp
FLUXNET.CH4.shp = st_transform(FLUXNET.CH4.shp, crs= crs(climate))
FLUXNET.CH4.shp
```
Look at the data that is available in climate:
```{r, include=T}
names(climate)
```
Extract climate information to FLUXNET.CH4.shp:
```{r, include=T}
FLUXNET.CH4.shp$MAP = terra::extract(climate, FLUXNET.CH4.shp)$MAP
FLUXNET.CH4.shp$TMIN = terra::extract(climate, FLUXNET.CH4.shp)$TMIN
FLUXNET.CH4.shp$TMAX = terra::extract(climate, FLUXNET.CH4.shp)$TMAX
FLUXNET.CH4.shp$MAT = terra::extract(climate, FLUXNET.CH4.shp)$MAT

# Look at the data
FLUXNET.CH4.shp$MAP
FLUXNET.CH4.shp$TMIN
FLUXNET.CH4.shp$TMAX
FLUXNET.CH4.shp$MAT
```
Import elevation information (Elevation.tif):
```{r, include=T}
elevation <- terra::rast("data/Elevation.tif" )
elevation
```
Transform FLUXNET.CH4.shp to the same CRS as elevation:
```{r, include=T}
FLUXNET.CH4.shp = st_transform(FLUXNET.CH4.shp, crs= crs(elevation))
FLUXNET.CH4.shp
```
Look at the data that is available in elevation:
```{r, include=T}
names(elevation)
```
Extract elevation information to FLUXNET.CH4.shp:
```{r, include=T}
FLUXNET.CH4.shp$ELEVATION = terra::extract(elevation, FLUXNET.CH4.shp)$wc2.1_2.5m_elev
FLUXNET.CH4.shp$ELEVATION
```
# Joining tables
We can combine columns from two (or more) tables together. This can be achieved using the join family of functions in dplyr. There are different types of joins that will result in different outcomes.

inner_join() includes all rows that appear in both the first data frame (x) and the second data frame (y).

left_join() returns all rows from x  based on matching rows on shared columns in y.
right_join() is the companion to left_join(), but returns all rows included in y based on matching rows on shared columns in x.

Import APPEEARS file where I requested MODIS NDVI and EVI data for all FLUXNET_sites (data/ENV720-MOD13A3-061-results.csv):
```{r, include=T}
FLUXNET <- read.csv("data/ENV720-MOD13A3-061-results.csv")
names(FLUXNET)
```

Subset the FLUXNET dataset to include only the columns of interest and rename them:
```{r, include=T}
FLUXNET.sub <- FLUXNET %>% select( "ID",
"Date",
"MOD13A3_061__1_km_monthly_EVI", "MOD13A3_061__1_km_monthly_NDVI", "MOD13A3_061__1_km_monthly_VI_Quality") %>% 
rename( SITE_ID = ID,
EVI = MOD13A3_061__1_km_monthly_EVI,
NDVI = MOD13A3_061__1_km_monthly_NDVI, 
QAQC = MOD13A3_061__1_km_monthly_VI_Quality ) %>% filter( QAQC > 0)

names(FLUXNET.sub)
```
Make the FLUXNET.CH4.shp vector a dataframe:
```{r, include=T}
FLUXNET_CH4 <- as.data.frame( FLUXNET.CH4.shp)
```
Identify the column that you should use to join the datasets FLUXNET_CH4 and FLUXNET.sub:
```{r, include=T}
FLUXNET_CH4$SITE_ID
FLUXNET.sub$SITE_ID
```
Use left_join because you want to keep all the data from FLUXNET_CH4 and only the sites in FLUXNET.sub that match the sites in FLUXNET_CH4:
```{r, include=T}
FLUXNET_CH4_final <- FLUXNET_CH4 %>% left_join( FLUXNET.sub, by= 'SITE_ID')
```
Check to make sure the list of sites matches:
```{r, include=T}
length( unique(FLUXNET_CH4$SITE_ID))
length(unique(FLUXNET_CH4_final$SITE_ID))
```
Import the monthly FLUX data:
```{r, include=T}

load( "data/FLUXNET_FLUXES.Rdata")
```
Look at the flux file "FLUXNET.flux":
```{r, include=T}
summary( FLUXNET.flux)
names(FLUXNET.flux)

# Make SITE ID:
FLUXNET.flux$SITE_ID <- FLUXNET.flux$SITE

# This is a good column to join based on:
FLUXNET.flux$YearMon

```
In your site file, convert the date to Year-Month and use this column to join it with the flux file "FLUXNET.flux":

```{r, include=T}
names(FLUXNET_CH4_final)
summary( FLUXNET_CH4_final$Date)
FLUXNET_CH4_final$Date

# Format as a Date:
FLUXNET_CH4_final$Date.f <-FLUXNET_CH4_final$Date %>% as.Date(format='%Y-%m-%d')

# Format as a Yearmon:
library(zoo)
FLUXNET_CH4_final$YearMon <- FLUXNET_CH4_final$Date.f %>% zoo::as.yearmon( "%m-%Y")

class(FLUXNET_CH4_final$YearMon)
```
Join the two files: 
```{r, include=T}
fluxes_month <- FLUXNET.flux %>% left_join(FLUXNET_CH4_final , by = c ('YearMon', 'SITE_ID'))

```
Save your file:
```{r, include=T}
save(fluxes_month, file="data/Monthly_Fluxes.Rdata" )
```
You are now prepared to take data from different sources to build a file to explore patterns in methane infrastructure. Take note of the difference between joining site based static data versus site based data the changes over time. In your next in class assessment, you will build a file with all of the data you want to use to develop your methane model. To prepare for this, find and obtain your data sources. 