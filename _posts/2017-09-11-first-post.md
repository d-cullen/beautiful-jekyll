---
layout: post
title: Creating a NFL map in R with Leaflet
image: /img/hello_world.jpeg
tags: [nfl, map, R, gis, leaflet]
---

Inspired by [this post](https://www.reddit.com/r/CFB/comments/6s12dr/closest_school_in_each_conference_to_every_county/) in r/cfb, I wanted to see if I could do the same for NFL teams using the leaflet package.

I first created a spreadsheet of the location of each team's stadium and colors using data from Wikipedia and a website of [team colors](http://jim-nielsen.com/teamcolors/). I also downloaded a shapefile of U.S. counties from [the U.S. Census Bureau](https://www.census.gov/geo/maps-data/data/cbf/cbf_counties.html). 

<ul>
{% for x in site.data.NFL_Stadiums_Colors %}
  <li>{{ x.Team" }}-{{ x.Color2 }}</li>
{% endfor %}
</ul>

First, I loaded the stadium and color data into `R` using `read.csv`. Then, I converted 


```{r eval=FALSE}
#Load csv into dataframe containing stadium locations
stadiums <- read.csv(data_path)

# Create Spatial DataFrame of Stadiums
sp.stadiums <- stadiums
coordinates(sp.stadiums) <- ~Long+Lat
```
