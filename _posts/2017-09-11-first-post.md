---
layout: post
title: Creating a NFL map in R with Leaflet
image: /img/hello_world.jpeg
tags: [nfl, map, R, gis, leaflet]
---

Inspired by [this post](https://www.reddit.com/r/CFB/comments/6s12dr/closest_school_in_each_conference_to_every_county/) in r/cfb, I wanted to see if I could do the same for NFL teams using the leaflet package.

I first created a spreadsheet of the location of each team's stadium and colors using data from Wikipedia and a website of [team colors](http://jim-nielsen.com/teamcolors/). I also downloaded a shapefile of U.S. counties from [the U.S. Census Bureau](https://www.census.gov/geo/maps-data/data/cbf/cbf_counties.html). 

First, I loaded the stadium and color data into `R` using `read.csv`. Then, I converted the latitude and longitude variables into spatial coordinates using the `sp` package.

```
#Load csv into dataframe containing stadium locations
stadiums <- read.csv(data_path)

# Create Spatial DataFrame of Stadiums
sp.stadiums <- stadiums
coordinates(sp.stadiums) <- ~Long+Lat
```
I imported the county shapefile using `readOGR()`

```
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

