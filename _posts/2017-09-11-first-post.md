---
layout: post
title: Creating a NFL map in R with Leaflet
css: "/css/nfl.css"
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
I use the `gCentroid` function in the `rgeos` package to find the centroid of every county in the shapefile.

```R
# get the centroids and then convert them to a SpatialPointsDataFrame
county_centers <- SpatialPointsDataFrame(gCentroid(us.map, byid=TRUE), 
                                         us.map@data, match.ID=FALSE)
```

The `plot(county_center)` with a cross on the location of each county center.
![Map of county centers](/img/centers_plot.jpg){: .center-image }
Using `gDistance` I calculate the distance between each centroid and each stadium in the data. Then I merge the team data onto the distance data so that each row gives the distance to each stadium.
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
I create a function that finds the minimum distance in each column (county) and writes the team name to every cell in the column.
```R
closest_f <- function(df) {
  i <- 1
  n <- ncol(df)
  while (i < n){
    df$c.team <- df[which(df[,i] == min(df[,i])), which(colnames(df)=="Team")]
    df[,i]    <- df$c.team
    i <- i + 1
  }
  
  # Drop Team and c.team columns
  drops  <- c('Team', 'c.team')
  df     <- df[ , !(names(df) %in% drops)]
  return(df)
  
}
```
After running the function on the distance matrix, I merge the data onto the spatial points data frame using apply with the min option (the min option shouldn't do anything because every entry in each column is the same). Then I merge the data back to the spatial polygons data frame. Now I have the closest stadium associated with every county in the contiguous United States. 

```R
closest <- closest_f(dist_matrix)

# Add closest team name to county_centers @data
county_centers@data$TEAM_0 <- apply(closest,2,min)

# Merge team data back to shapefile
us.map <- merge(us.map, county_centers, by.x="NAME", by.y="NAME")
```


```R
## Create color palette 
# Read CSV that contains first and second colors
colors <- read.csv(data_path, stringsAsFactors = FALSE)
# Keep only columns team and color1 and color2
keep.c <- c('Color1', 'Team', 'Color2')
colors <- colors[,(names(colors) %in% keep.c) ]

# Sort colors alphabetically by team so colorfactor uses correct order
colors <- colors[order(colors$Team),]

c1 <- colors[["Color1"]]
c2 <- colors[["Color2"]]
t  <- colors[["Team"]]

# Create two color palettes for factor variables
c1pal <- colorFactor(c1, t)
c2pal <- colorFactor(c2, t)


################
## Create popup
county_popup <- paste0( us.map@data$NAME,"<br>", us.map@data$TEAM_0)


## Create map of closest team to each county 
# color and weight to change outline
leaflet() %>%
  addPolygons(data = us.map, weight = 1.5, color = ~c2pal(us.map@data$TEAM_0) , smoothFactor = 0.5, fillOpacity = 1,
              fillColor = ~c1pal(us.map@data$TEAM_0), popup = county_popup) %>% 
  addProviderTiles(providers$CartoDB.Positron)
  11
 ```
 
 ```{r fig.height=2.5}
m <- leaflet() %>% setView(lng = -71.0589, lat = 42.3601, zoom = 12)
m %>% addTiles()
```
  
