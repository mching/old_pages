# King Tides Citizen Science Project

I read in the [newspaper](http://www.staradvertiser.com/2017/05/22/hawaii-news/king-tides-summer-swells-and-high-sea-levels-could-imperil-coastal-areas/) that the [University of Hawaii](http://ccsr.seagrant.soest.hawaii.edu/king-tides) was recruiting "citizen scientists" to help document the impact of this week's "[king tides](http://ccsr.seagrant.soest.hawaii.edu/Hawaii%20Sea%20Level)" on our coastline. I uploaded some photos to [the website](https://getliquid.io) but also found that we could download the dataset. I took this as an opportunity to learn something about geographic information systems (GIS) and R, and the end result was a pretty nice map of all the places that were photographed this week.

## GIS in R
I knew nothing about GIS before starting this but there were some excellent articles on the web about it. One tutorial used the [`ggmap`](https://journal.r-project.org/archive/2013-1/kahle-wickham.pdf) package. I just wanted to plot points on a map, and my prior experience with `ggplot2` and this [cheatsheet](https://www.nceas.ucsb.edu/~frazier/RSpatialGuides/ggmap/ggmapCheatsheet.pdf) made it pretty easy.

The hardest part was all the installation requirements, specifically GDAL. There's a lot of information on the web on how to install this, but it was a lot of googling that finally got it to work on Fedora. MacOS GDAL requires you to use packages available [here](http://www.kyngchaos.com/software/frameworks).

## Making a Map of Oahu
First I wanted a map of Oahu. This couldn't be easier using `ggmap`. This produces a pretty watercolor looking map from [Stamen.com](https://stamen.com). 


```r
library(tidyverse)
library(ggmap)
myLocation <- "Oahu"
myMap <- get_map(location = myLocation,
                 source = "stamen",
                 maptype = "watercolor", 
                 crop = FALSE,
                 zoom = 10)
ggmap(myMap)
```

![](https://mching.github.io/images/tide1.png)<!-- -->

Then I wanted to put the points on the map. The dataset was in comma separated values format (albeit with this `latin1` file encoding when I did this post first in Fedora). It was easy to load the dataset.


```r
dat <- read.csv("../datasets/tides_20170526.csv", 
                # fileEncoding = "latin1", 
                colClasses = "character")
dat <- tbl_df(dat)
```

The file needed some cleaning for convenient use in the `ggmap` package.


```r
dat
```

```
## # A tibble: 818 × 20
##                                                     Dataset.Name
##                                                            <chr>
## 1  Hawai'i and Pacific Islands King Tides Project (HI Sea Grant)
## 2  Hawai'i and Pacific Islands King Tides Project (HI Sea Grant)
## 3  Hawai'i and Pacific Islands King Tides Project (HI Sea Grant)
## 4  Hawai'i and Pacific Islands King Tides Project (HI Sea Grant)
## 5  Hawai'i and Pacific Islands King Tides Project (HI Sea Grant)
## 6  Hawai'i and Pacific Islands King Tides Project (HI Sea Grant)
## 7  Hawai'i and Pacific Islands King Tides Project (HI Sea Grant)
## 8  Hawai'i and Pacific Islands King Tides Project (HI Sea Grant)
## 9  Hawai'i and Pacific Islands King Tides Project (HI Sea Grant)
## 10 Hawai'i and Pacific Islands King Tides Project (HI Sea Grant)
## # ... with 808 more rows, and 19 more variables: Dataset.Owner <chr>,
## #   User <chr>, Submission.Latitude <chr>, Submission.Longitude <chr>,
## #   Date.Created <chr>, Record <chr>,
## #   Hawai.iSea.Grant.King.Tides.Photo.Database <chr>,
## #   Location..tap.arrow.to.input.exact.location....Latitude <chr>,
## #   Location..tap.arrow.to.input.exact.location....Longitude <chr>,
## #   Share.your.image. <chr>, Island.s. <chr>, Location.Description <chr>,
## #   Orientation..check.your.compass.. <chr>, Date <chr>, Time <chr>,
## #   Tell.us.something.about.your.photo. <chr>, Photographer.s.Name <chr>,
## #   By.uploading.an.image.you.irrevocably.consent.that.it.may.be.used.for.educational..research..outreach.and.promotional.purposes..in.any.medium..in.perpetuity. <chr>,
## #   X <chr>
```

```r
# Rename latitute and longitude columns for more convenient reference
names(dat)[9] <- "latitude"
names(dat)[10] <- "longitude"

# Make latitude and longitude numeric
dat$latitude <- as.numeric(dat$latitude)
```

```
## Warning: NAs introduced by coercion
```

```r
dat$longitude <- as.numeric(dat$longitude)
```

```
## Warning: NAs introduced by coercion
```

```r
# Convert date and time columns into POSIX format
dat$date_time <- paste(dat$Date, dat$Time)
dat$date_time <- strptime(dat$date_time, "%m/%d/%y %H:%M")
dat$date_time <- as.POSIXct(dat$date_time)
dat$LatLong <- paste0(dat$latitude, ":", dat$longitude)
```


I just wanted to see where people had been active in the last two days so I filtered the dataset by date.

```r
minidat <- dat %>% filter(date_time > as.POSIXct("2017-05-25")) %>% 
  select(latitude, longitude, date_time, LatLong)
```

The resulting map was pretty sweet.


```r
ggmap(myMap) + geom_point(aes(x = longitude, y = latitude), data = minidat, color = "black", alpha = 0.3)
```

```
## Warning: Removed 81 rows containing missing values (geom_point).
```

![](https://mching.github.io/images/tide2.png)<!-- -->

I got virtually the same map using the `qmap` function. This function uses a Google Maps terrain map to create its base image.


```r
qmap("Oahu") + geom_point(aes(x = longitude, y = latitude), data = minidat, color = "red", alpha = 0.3)
```

```
## Map from URL : http://maps.googleapis.com/maps/api/staticmap?center=Oahu&zoom=10&size=640x640&scale=2&maptype=terrain&language=en-EN&sensor=false
```

```
## Information from URL : http://maps.googleapis.com/maps/api/geocode/json?address=Oahu&sensor=false
```

```
## Warning: `panel.margin` is deprecated. Please use `panel.spacing` property
## instead
```

```
## Warning: Removed 82 rows containing missing values (geom_point).
```

![](https://mching.github.io/images/tide3.png)<!-- -->

The resulting images from `ggmap` were pretty nice but I wanted more. Nowadays it's customary to be able to pan and zoom like in Google Maps. The `ggmap` method produces a static map which is nice to look at, but it would be nice to see what points were taken on other islands. Or how about that blob of points on the south shore. Could we zoom in to see more readily where those points are?

## `googleVis` version
I did some more research and found the `gvisMap` function from the `googleVis` package could provide a solution to this problem. Inspiration for how to do this came from [this tutorial](https://pakillo.github.io/R-GIS-tutorial/#googlevis).

The special thing about `gvisMap` is that it requires you to feed it a variable containing latitude and longitude values in a specific latitude:longitude format. That's why I did this above in the data cleaning steps.

It also allows you to write a label for each point. I just used the date and time label but if I were to spend a little more time on this, it would be simple to write a label that gave the record number, the time, and some brief description.


```r
library(googleVis)

M1 <- gvisMap(minidat, "LatLong", tipvar = "date_time", 
              options=list(showTip=TRUE, showLine=F, enableScrollWheel=TRUE, 
                           mapType='satellite', useMapTypeControl=TRUE, width=800,height=800))
```

The next command should be `plot(M1)` but this launches a javascript based map running locally. I have to figure out how to integrate this into a Jekyll blog like this one, but for the time being, you can see the result [here](https://www2.hawaii.edu/~mslching/map_20170526.html). I copied the code from running `print(M1, 'chart')` onto an html page and voila, [the map](https://www2.hawaii.edu/~mslching/map_20170526.html) works just like it should!

## Discussion
It wasn't too hard to plot points on a map. If I'm going to do more of this, I think the `googleVis` is probably better for dynamic presentation than `ggmap` but the `ggmap` might be nicer for static presentation.

Looking at the actual results, it comes as no surprise that urban Honolulu is well documented. There are lots of people living here and so Waikiki and Hawaii Kai were covered pretty well. Rural areas were much less well documented, so that's something for the project to work on!

## Conclusions
GIS in R is feasible for a relative newbie to GIS. Citizen scientists on the King Tides Project documented the urban Honolulu area better than rural areas.
