---
layout: post
title: Scale Model of Solar System
---

One of my son's Cub Scouts "adventures" requires him to create a scale model of the solar system. We sketched the calculations on the back of a receipt over lunch. We started with the distance we figured that we would use (length of our street, 387 feet) and then calculated the distances to each planet based on this. I figured it would be neat to create a function in R that would return a table of the relative distances based on a given model solar system radius.

Here are the relative distances from the sun for each planet.

```r
solar_system <- data.frame(
  solar_object = c("Sun", "Mercury", "Venus", "Earth", "Mars", "Jupiter",
                   "Saturn", "Uranus", "Neptune"),
  distance_km = c(0, 57.9e6, 108.2e6, 149.6e6, 227.8e6, 778.3e6, 
                  1427e6, 2871e6, 4497.1e6),
  diam_km = c(1.392e6, 4878, 12104, 12756, 6787, 142792, 120660, 
              51118, 48600)
)
solar_system
```

```
##   solar_object distance_km diam_km
## 1          Sun           0 1392000
## 2      Mercury    57900000    4878
## 3        Venus   108200000   12104
## 4        Earth   149600000   12756
## 5         Mars   227800000    6787
## 6      Jupiter   778300000  142792
## 7       Saturn  1427000000  120660
## 8       Uranus  2871000000   51118
## 9      Neptune  4497100000   48600
```

We can use some data manipulation from the `dplyr` package to calculate the new distance and diameters. I based the model solar system distance (radius) on my street length, which is approximately 387 feet.


```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
solar_system <- tbl_df(solar_system)
model_total_distance <- 387 # feet
actual_total_distance <- solar_system$distance_km[9]
solar_system <- solar_system %>% 
  mutate(model_distance = distance_km * model_total_distance /
           actual_total_distance, 
         model_diam = diam_km * model_total_distance /
           actual_total_distance)
solar_system
```

```
## Source: local data frame [9 x 5]
## 
##   solar_object distance_km diam_km model_distance   model_diam
##         (fctr)       (dbl)   (dbl)          (dbl)        (dbl)
## 1          Sun           0 1392000       0.000000 0.1197891975
## 2      Mercury    57900000    4878       4.982611 0.0004197785
## 3        Venus   108200000   12104       9.311201 0.0010416153
## 4        Earth   149600000   12756      12.873897 0.0010977234
## 5         Mars   227800000    6787      19.603433 0.0005840584
## 6      Jupiter   778300000  142792      66.976963 0.0122880310
## 7       Saturn  1427000000  120660     122.801139 0.0103834516
## 8       Uranus  2871000000   51118     247.065220 0.0043989829
## 9      Neptune  4497100000   48600     387.000000 0.0041822953
```

We can put everything into a function for more automatic usage.


```r
solar_model <- function(distance) {
  require(dplyr)
  solar_system <- data.frame(
    solar_object = c("Sun", "Mercury", "Venus", "Earth", "Mars", "Jupiter",
                   "Saturn", "Uranus", "Neptune"),
    distance_km = c(0, 57.9e6, 108.2e6, 149.6e6, 227.8e6, 778.3e6, 
                  1427e6, 2871e6, 4497.1e6),
    diam_km = c(1.392e6, 4878, 12104, 12756, 6787, 142792, 120660, 
              51118, 48600)
    )
  solar_system <- tbl_df(solar_system)
  model_total_distance <- distance
  actual_total_distance <- solar_system$distance_km[9]
  solar_system <- solar_system %>% 
    mutate(model_distance = distance_km * model_total_distance /
             actual_total_distance, 
           model_diam = diam_km * model_total_distance /
             actual_total_distance)
  print(solar_system)
}
```

Here's the model for 100 meters.

```r
solar_model(100)
```

```
## Source: local data frame [9 x 5]
## 
##   solar_object distance_km diam_km model_distance   model_diam
##         (fctr)       (dbl)   (dbl)          (dbl)        (dbl)
## 1          Sun           0 1392000       0.000000 0.0309532810
## 2      Mercury    57900000    4878       1.287496 0.0001084699
## 3        Venus   108200000   12104       2.405995 0.0002691512
## 4        Earth   149600000   12756       3.326588 0.0002836495
## 5         Mars   227800000    6787       5.065487 0.0001509195
## 6      Jupiter   778300000  142792      17.306709 0.0031752018
## 7       Saturn  1427000000  120660      31.731560 0.0026830624
## 8       Uranus  2871000000   51118      63.841142 0.0011366881
## 9      Neptune  4497100000   48600     100.000000 0.0010806964
```

We can see that the diameter for the planets is too small to really draw (Earth is 0.0002 m or 0.2 mm in diameter). 

We can trial and error to see how much space we would need to make the Earth a decent size (like 5 mm). We can try 1000 m.


```r
solar_model(1000)
```

```
## Source: local data frame [9 x 5]
## 
##   solar_object distance_km diam_km model_distance  model_diam
##         (fctr)       (dbl)   (dbl)          (dbl)       (dbl)
## 1          Sun           0 1392000        0.00000 0.309532810
## 2      Mercury    57900000    4878       12.87496 0.001084699
## 3        Venus   108200000   12104       24.05995 0.002691512
## 4        Earth   149600000   12756       33.26588 0.002836495
## 5         Mars   227800000    6787       50.65487 0.001509195
## 6      Jupiter   778300000  142792      173.06709 0.031752018
## 7       Saturn  1427000000  120660      317.31560 0.026830624
## 8       Uranus  2871000000   51118      638.41142 0.011366881
## 9      Neptune  4497100000   48600     1000.00000 0.010806964
```

Still too small. That's only 2 mm. Let's try 2000 m.


```r
solar_model(2000)
```

```
## Source: local data frame [9 x 5]
## 
##   solar_object distance_km diam_km model_distance  model_diam
##         (fctr)       (dbl)   (dbl)          (dbl)       (dbl)
## 1          Sun           0 1392000        0.00000 0.619065620
## 2      Mercury    57900000    4878       25.74993 0.002169398
## 3        Venus   108200000   12104       48.11990 0.005383025
## 4        Earth   149600000   12756       66.53176 0.005672989
## 5         Mars   227800000    6787      101.30973 0.003018390
## 6      Jupiter   778300000  142792      346.13418 0.063504036
## 7       Saturn  1427000000  120660      634.63121 0.053661248
## 8       Uranus  2871000000   51118     1276.82284 0.022733762
## 9      Neptune  4497100000   48600     2000.00000 0.021613929
```

With that, the Sun is 62 cm in diameter and the Earth is about 0.5 cm. That's at a scale of 2000 meters, or about 1.25 miles. Wow! 

Interestingly enough we can also try to place the nearest star (other than the Sun) on our scale. Proxima Centauri is 4.246 light years away, and 1 light year is 9.461 x 10^12 kilometers. Its radius is 0.141 the radius of the sun (0.141 * 695,700 km).


```r
solar_model_PC <- function(distance) {
  require(dplyr)
  solar_system <- data.frame(
    solar_object = c("Sun", "Mercury", "Venus", "Earth", "Mars", "Jupiter",
                   "Saturn", "Uranus", "Neptune", "Proxima Centauri"),
    distance_km = c(0, 57.9e6, 108.2e6, 149.6e6, 227.8e6, 778.3e6, 
                  1427e6, 2871e6, 4497.1e6, 9.461e12*4.246),
    diam_km = c(1.392e6, 4878, 12104, 12756, 6787, 142792, 120660, 
              51118, 48600, 0.141*2*6.957e5)
    )
  solar_system <- tbl_df(solar_system)
  model_total_distance <- distance
  actual_total_distance <- solar_system$distance_km[9]
  solar_system <- solar_system %>% 
    mutate(model_distance = distance_km * model_total_distance /
             actual_total_distance, 
           model_diam = diam_km * model_total_distance /
             actual_total_distance)
  print(solar_system)
}
```

After adding in the distance to Proxima Centauri and is diameter to the model, we can recalculate on a scale where the Sun to Neptune is 100 m.

```r
solar_model_PC(100)
```

```
## Source: local data frame [10 x 5]
## 
##        solar_object  distance_km   diam_km model_distance   model_diam
##              (fctr)        (dbl)     (dbl)          (dbl)        (dbl)
## 1               Sun 0.000000e+00 1392000.0   0.000000e+00 0.0309532810
## 2           Mercury 5.790000e+07    4878.0   1.287496e+00 0.0001084699
## 3             Venus 1.082000e+08   12104.0   2.405995e+00 0.0002691512
## 4             Earth 1.496000e+08   12756.0   3.326588e+00 0.0002836495
## 5              Mars 2.278000e+08    6787.0   5.065487e+00 0.0001509195
## 6           Jupiter 7.783000e+08  142792.0   1.730671e+01 0.0031752018
## 7            Saturn 1.427000e+09  120660.0   3.173156e+01 0.0026830624
## 8            Uranus 2.871000e+09   51118.0   6.384114e+01 0.0011366881
## 9           Neptune 4.497100e+09   48600.0   1.000000e+02 0.0010806964
## 10 Proxima Centauri 4.017141e+13  196187.4   8.932736e+05 0.0043625314
```

So our nearest star turns out to be 890 kilometers away on this scale. Put another way, if we had this 100 meter model in Los Angeles, where the Earth is again 0.2 mm in size, the nearest star would be around Salt Lake City!
