---
layout: post
title: Reverse-In Parking in Hawaii and California
---

One of my mainland-born friends recently expressed to me how he was annoyed that Hawaii drivers like to reverse their cars into parking stalls more than drivers on the mainland. My wife, who is also from the mainland, agreed with him enthusiastically. Whether this is appropriate parking behavior or not, I had never noticed it before, so I figured it was something I could test with some observations. I had a trip coming up to the mainland, and I was able to collect some data to test whether the Hawaii drivers were really so different than their California counterparts.

## Methods

I took photographs of parked vehicles in the parking lot of the [99 Ranch mall](https://www.google.com/maps/place/99+Ranch+Market/@33.8411535,-117.9435128,15z/data=!4m5!3m4!1s0x0:0x867c7acd905366a!8m2!3d33.8411535!4d-117.9435128) in Anaheim, California. This is a mixed use mall with a variety of restaurants and small retail and service businesses. The comparison Hawaii photographs were taken in [Koko Marina](https://www.google.com/maps/place/Koko+Marina+Center/@21.2780102,-157.707062,17z/data=!3m1!4b1!4m5!3m4!1s0x7c001265ac2c9e61:0xf24af58b58bfd296!8m2!3d21.2780052!4d-157.7048733) and the [Ward area parking structure](https://www.google.com/maps/place/Nordstrom+Rack+Ward+Village+Shops/@21.2938104,-157.8553068,17z/data=!4m12!1m6!3m5!1s0x7c006dfb62829bb5:0xaac1f817df7b6e43!2sWard+Village!8m2!3d21.2948051!4d-157.8560471!3m4!1s0x0:0x47553222252dff70!8m2!3d21.2933142!4d-157.8514713), which also has a variety of restaurants and retail businesses. 

I recorded how many vehicles were facing in and how many were facing out in each photograph. 

I included all vehicles that were parked in stalls that were perpendicular to the traffic direction. I excluded non-passenger vehicles such as semi-trucks or commercial vans. 

I estimated beforehand that the proportion of reversed-in cars in Hawaii and California would be about 30% and 15% respectively. With a Type 1 error rate of 5% and a Type 2 error rate of 20%, the sample size required was 118 cars per group. (https://select-statistics.co.uk/calculators/sample-size-calculator-two-proportions/)

I analyzed the data using Fisher’s exact test, with a two sided probability. Epidemiologic risk based estimates were calculated using the [`epiR`](https://cran.r-project.org/web/packages/epiR/epiR.pdf) package of R.

## Results
The following are links to representative photos: [Anaheim (California)](https://mching.github.io/images/anaheim99ranch.JPG), [Koko Marina](https://mching.github.io/images/KokoMarina.JPG) (Hawaii), and [Ward Village](https://mching.github.io/images/Ward.JPG) (Hawaii).

I obtained data on 172 vehicles in California and 197 vehicles in Hawaii. In California, 18/172 (10.4%, 95% confidence interval 6.6-16.0%) vehicles were reversed in. In Hawaii, 68/197 (34.5%, 95% CI 28.2-41.4%) vehicles were reversed in.

Vehicles in Hawaii were 3.3 (2.0-5.3) times more likely to be reversed in than in California. The two-sided p-value was less than 0.0001.

Here's the code for the analysis.

```r
library(dplyr)
library(tidyr)
library(epiR)

# Load files
fileURL <- "https://mching.github.io/datasets/parking_data.csv"
download.file(fileURL, destfile = "parking_data.csv", method = "curl")
dat <- read.csv("parking_data.csv")

# Clean data. I had summarized each photo by number of cars facing out and facing in. These had to be put into a long form that R could work with, where each row is one observation
dat <- tbl_df(dat)
dat <- dat %>% gather(Direction, n, Reverse_in:Forward_in)
dat <- dat[rep(1:nrow(dat), dat$n), 1:3]

dat$State <- factor(dat$State, levels = c("Hawaii", "California"))
dat$Direction <- factor(dat$Direction, levels = c("Reverse_in", "Forward_in"))

table1 <- table(dat$State, dat$Direction)
table1
```

```r
fisher.test(table1)
```

```
## 
## 	Fisher's Exact Test for Count Data
## 
## data:  table1
## p-value = 3.357e-08
## alternative hypothesis: true odds ratio is not equal to 1
## 95 percent confidence interval:
##  2.488736 8.458416
## sample estimates:
## odds ratio 
##   4.492008
```

Here's the summary of the observed parking behavior in Hawaii vs California.

```
##             
##              Reverse_in Forward_in
##   Hawaii             68        129
##   California         18        154
```

```r
epi.2by2(table1, method = "cross.sectional")
```

And here is the data analyzed as an epidemiologic cross sectional table. The exposure is Hawaii, and the outcome is reverse-in parking.

```
##              Outcome +    Outcome -      Total        Prevalence *
## Exposed +           68          129        197                34.5
## Exposed -           18          154        172                10.5
## Total               86          283        369                23.3
##                  Odds
## Exposed +       0.527
## Exposed -       0.117
## Total           0.304
## 
## Point estimates and 95 % CIs:
## -------------------------------------------------------------------
## Prevalence ratio                             3.30 (2.05, 5.32)
## Odds ratio                                   4.51 (2.55, 7.97)
## Attrib prevalence *                          24.05 (15.99, 32.12)
## Attrib prevalence in population *            12.84 (6.55, 19.13)
## Attrib fraction in exposed (%)              69.68 (51.12, 81.19)
## Attrib fraction in population (%)           55.10 (34.26, 69.33)
## -------------------------------------------------------------------
##  X2 test statistic: 29.721 p-value: < 0.001
##  Wald confidence limits
##  * Outcomes per 100 population units
```


## Discussion
It wasn’t even close. The observed data supported the hypothesis that Hawaii drivers like to reverse into stalls much more than California drivers. The effect size was dramatic, with more than 3 times more cars reversed in Hawaii than California. 

The reason for this behavior is unknown but there is much online speculation about it. The [Hawaii Driver Manual, page 64](https://hidot.hawaii.gov/highways/files/2015/11/mvso-HawaiiDrivers-Manual09.2015.pdf) recommends reversing in so that drivers can enter traffic in a forward direction. However, this appears to be mainly about entering a roadway rather than a parking lot lane.

More on the controversy from the [Huffington Post](http://www.huffingtonpost.com/2015/03/17/reverse-parking-hawaii_n_6887760.html)
and [Slate](http://www.slate.com/articles/life/transport/2011/02/youre_parking_wrong.html)
and NPR ([a study done in China versus the USA](http://www.npr.org/2014/08/27/343623220/parking-behavior-may-reflect-economic-drive))
 
Limitations of the study include the convenience sample. Clustering was not taken into account in the analysis. In addition, there may be other factors that affect the parking in specific areas such as the neighborhoods, retail mix, time of day, width of parking stalls, percentage of stalls filled, etc. While I tried to match the types of businesses, most of the Hawaii cars came from a nearly full, enclosed parking garage with one-way lanes, while the California lot was virtually empty. 

Like this current study, any future work would be frivolous and gratuitous, but possibilities include collecting data from other parts of the mainland, limiting observations to cars parked up against a barrier (like a sidewalk or building), and matching more closely the stores and neighborhoods (i.e., comparing Safeway parking lots in suburban locations).

## Conclusion
In Hawaii drivers reverse into parking stalls at a higher rate than in California. No comment is made on the appropriateness of reversing in, but I, for one, like doing it.
