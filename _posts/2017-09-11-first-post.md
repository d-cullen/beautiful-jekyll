---
layout: post
title: Creating a NFL map in R with Leaflet
tags: [nfl, map, R, gis, leaflet]
---

Inspired by [this post](https://www.reddit.com/r/CFB/comments/6s12dr/closest_school_in_each_conference_to_every_county/) in r/cfb, I wanted to see if I could a similar map for NFL teams using the leaflet package.

I first created a spreadsheet of the location of each team's stadium and colors using data from Wikipedia and a website of [team colors](http://jim-nielsen.com/teamcolors/). I also downloaded a shapefile of U.S. counties from [the U.S. Census Bureau](https://www.census.gov/geo/maps-data/data/cbf/cbf_counties.html). Full code can be seen at the bottom of the page.

First, I loaded the stadium and color data into `R` using `read.csv`. Then, I converted the latitude and longitude variables into spatial coordinates using the `sp` package.

```R
#Load csv into dataframe containing stadium locations
stadiums <- read.csv(data_path)

# Create Spatial DataFrame of Stadiums
sp.stadiums <- stadiums
coordinates(sp.stadiums) <- ~Long+Lat
```
I imported the county shapefile using `readOGR()` and removed all areas not in the contiguous United States. 

```R
# Read in us county shapefiles
us.map  <- readOGR(dsn = shape_path, layer = "cb_2016_us_county_500k", stringsAsFactors = FALSE)

#  Remove  Alaska(02), Hawaii (15), Puerto Rico (72), Guam (66), Virgin Islands (78), American Samoa (60)
#  Mariana Islands (69), Micronesia (64), Marshall Islands (68), Palau (70), Minor Islands (74)
us.map <- us.map[!us.map$STATEFP %in% c("02", "15" ,"72", "66", "78", "60", "69",
                                        "64", "68", "70", "74"),]

# Remove other outling islands 
us.map <- us.map[!us.map$STATEFP %in% c("81", "84", "86", "87", "89", "71", "76",
                                        "95", "79"),]
```


```R
# get the centroids and then convert them to a SpatialPointsDataFrame
county_centers <- SpatialPointsDataFrame(gCentroid(us.map, byid=TRUE), 
                                         us.map@data, match.ID=FALSE)
```

The `plot(county_center)` with a cross on the location of each county center.
![Map of county centers](/img/centers_plot.jpg){: .center-image }

```R
# Create matrix of distances between each county center and each stadium
# stadium in rows counties in columns
dist_matrix <- gDistance(county_centers,sp.stadiums, byid=TRUE)

# Merge stadium data to distance data each row is distance to stadium and there 
# will be a column of team names
dist_matrix <- merge(dist_matrix, stadiums, by=0, all=TRUE)

# Keep only distances and team names
drops <- c('Row.names', 'Lat', 'Long',  'Color1', 'Color2')
dist_matrix <- dist_matrix[ , !(names(dist_matrix) %in% drops)]
```
