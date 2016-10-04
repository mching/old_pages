# Most Yards of Offense by a College Football Team in a Shutout Loss
Michael Ching  

## Introduction
The purpose of this document is to explore the most yards in a shutout loss for a college football team. It was inspired after the University of Hawai\`i gave up 462 yards in shutting out San Jose State on November 15, 2014. I wanted to see if I could use the `dplyr` package to explore this question.


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

## Data Source
To obtain a source of data, I downloaded a database of football game statistics from 2000-2013, from http://www.repole.com/sun4cast/data.html. I unzipped this to the working directory.


```r
download_url <- "http://www.repole.com/sun4cast/stats/cfbstats.zip"
download.file(download_url, "cfbstats.zip")
unzip("cfbstats.zip")
```

## Data Processing
The working directory now contained xml and csv versions of each year. I combined the csv versions into one large file. To do this, I read in each csv file and then use `rbind_all` to combine them.


```r
# Get a list of just the csv files
files <- list.files()
files <- files[grep(".csv", files)]

# Initialize a data list to hold the files when I read them in
n_files <- length(files)
data_list <- vector("list", n_files)

# Read them in one by one
for(i in seq_along(files)) {
  data_list[[i]] <- read.csv(files[i])
}

# Combine them into one data frame
dat <- rbind_all(data_list)
```

```
## Warning in rbind_all(data_list): Unequal factor levels: coercing to
## character

## Warning in rbind_all(data_list): Unequal factor levels: coercing to
## character

## Warning in rbind_all(data_list): Unequal factor levels: coercing to
## character

## Warning in rbind_all(data_list): Unequal factor levels: coercing to
## character
```

```r
rm(data_list)
```


## Identifying and Ranking Shutout Games
Now we just wanted to keep the games in which one of the teams scored 0 points. The pertinent variables are `ScoreOff` and `ScoreDef`. The variables are somewhat misleadingly categorized as belonging to Offense (the team in the TeamName variable) or Defense (the team in the Opponent variable).


```r
shutouts <- filter(dat, ScoreDef == 0 | ScoreOff == 0)
```

We created variables for total yards for "Offense" and for "Defense". 


```r
shutouts <- mutate(shutouts, TotalYdsOff = PassYdsOff + RushYdsOff,
       TotalYdsDef = PassYdsDef + RushYdsDef)
```

We first ranked by most yards the games in which the "Offense" was shutout.


```r
filter(shutouts, ScoreOff == 0) %>% 
  select(Date, TeamName, ScoreOff, TotalYdsOff) %>% 
  arrange(desc(TotalYdsOff))
```

```
## Source: local data frame [406 x 4]
## 
##          Date           TeamName ScoreOff TotalYdsOff
##         (chr)              (chr)    (int)       (int)
## 1          NA                 NA        0         397
## 2  10/20/2007     San Jose State        0         394
## 3  11/26/2011           U.C.L.A.        0         385
## 4  11/03/2001        Texas A & M        0         372
## 5  10/30/2004      Arizona State        0         363
## 6  10/30/2004 Southern Methodist        0         356
## 7          NA                 NA        0         355
## 8  11/20/2010         Iowa State        0         332
## 9  09/22/2012            Arizona        0         332
## 10 11/18/2006     Louisiana Tech        0         331
## ..        ...                ...      ...         ...
```

We then ranked by most yards the games in which the "Defense" was shutout.


```r
filter(shutouts, ScoreDef == 0) %>% 
  select(Date, Opponent, ScoreDef, TotalYdsDef) %>% 
  arrange(desc(TotalYdsDef))
```

```
## Source: local data frame [533 x 4]
## 
##          Date           Opponent ScoreDef TotalYdsDef
##         (chr)              (chr)    (int)       (int)
## 1  09/08/2012   Stephen F Austin        0         466
## 2          NA           Kentucky        0         397
## 3  10/20/2007     San Jose State        0         394
## 4  11/26/2011           U.C.L.A.        0         385
## 5  11/03/2001        Texas A & M        0         372
## 6  10/30/2004      Arizona State        0         363
## 7  10/30/2004 Southern Methodist        0         356
## 8          NA     Michigan State        0         355
## 9  11/20/2010         Iowa State        0         332
## 10 09/22/2012            Arizona        0         332
## ..        ...                ...      ...         ...
```

Looking at the results we see a quirk in the data in that some games seem to be represented twice. It would be preferable to have the games all together so that all the shut out teams can be ranked instead of doing it by "offense" or "defense".

To do this I generated a list of all the games in which the "offense" was shutout and renamed the variables so they could be matched later. I did the same thing with "defense" shutouts.


```r
shutouts_Off <- filter(shutouts, ScoreOff == 0) %>% 
  select(Date, TeamName, RushYdsOff, PassYdsOff, ScoreOff, TotalYdsOff, 
         Opponent) %>% 
  mutate(RushYds = RushYdsOff, PassYds = PassYdsOff, Score = ScoreOff, 
         TotalYds = TotalYdsOff, SO_team = TeamName, WinTeam = Opponent) %>%
  select(Date, SO_team, WinTeam, RushYds, PassYds, Score, TotalYds) %>%
  arrange(desc(TotalYds))

shutouts_Def <- filter(shutouts, ScoreDef == 0) %>% 
  select(Date, TeamName, RushYdsDef, PassYdsDef, ScoreDef, TotalYdsDef, 
         Opponent) %>% 
  mutate(RushYds = RushYdsDef, PassYds = PassYdsDef, Score = ScoreDef, 
         TotalYds = TotalYdsDef, WinTeam = TeamName, SO_team = Opponent) %>%
  select(Date, SO_team, WinTeam, RushYds, PassYds, Score, TotalYds) %>%
  arrange(desc(TotalYds))
```

We can then combine the two lists into one list and then only keep the unique games.


```r
shutouts_all <- rbind(shutouts_Off, shutouts_Def) %>% 
  arrange(desc(TotalYds)) %>% 
  unique()

rm(shutouts_Off); rm(shutouts_Def)

select(shutouts_all, -RushYds, -PassYds, -Score)
```

```
## Source: local data frame [592 x 4]
## 
##          Date            SO_team         WinTeam TotalYds
##         (chr)              (chr)           (chr)    (int)
## 1  09/08/2012   Stephen F Austin           S-M-U      466
## 2          NA                 NA Louisiana State      397
## 3          NA           Kentucky              NA      397
## 4  10/20/2007     San Jose State    Fresno State      394
## 5  11/26/2011           U.C.L.A.    Southern Cal      385
## 6  11/03/2001        Texas A & M      Texas Tech      372
## 7  10/30/2004      Arizona State      California      363
## 8  10/30/2004 Southern Methodist    Fresno State      356
## 9          NA                 NA        Michigan      355
## 10         NA     Michigan State              NA      355
## ..        ...                ...             ...      ...
```

Looking more at the data we see that there are some games that seem like the could be joined manually, like the game in rows 2 and 3 where Louisiana State beat an unidentified team while giving up 397 yards and Kentucky lost to an unidentified team while gaining 397 yards. The same holds for 9 and 10.

It appears that the 462 yards that San Jose State gained in total offense on November 15, 2014 are more than any other shutout team from 2000-2013 with the exception of Stephen F. Austin (http://scores.espn.go.com/ncf/boxscore?gameId=322522567) on 9/8/2012! 

It is also interesting that San Jose State's poor luck against Hawaii was not their only time experiencing such a historic feat. The Spartans gained 394 yards in being shut out by Fresno State on 10/20/2007, which is the #3 most yards gained in a shutout on my list (excluding the 11/15/2014 game). 

## Relationship between Yards and Points Scored

Another question is to see if there is a relationship between points scored and the number of yards gained. We can fit a linear regression model with points scored as the outcome and total yards as the predictor. Let's again create a list that combines the "offense" and "defense" teams.


```r
# Create TotalYdsOff and TotalYdsDef 
dat <- mutate(dat, TotalYdsOff = RushYdsOff + PassYdsOff, 
              TotalYdsDef = RushYdsDef + PassYdsDef)

dat_off <- filter(dat) %>% 
  select(Date, TeamName, RushYdsOff, PassYdsOff, ScoreOff, TotalYdsOff, 
         Opponent) %>% 
  mutate(RushYds = RushYdsOff, PassYds = PassYdsOff, Score = ScoreOff, 
         TotalYds = TotalYdsOff) %>%
  select(Date, TeamName, Opponent, RushYds, PassYds, Score, TotalYds) %>%
  arrange(desc(TotalYds))

dat_def <- filter(dat) %>% 
  select(Date, TeamName, RushYdsDef, PassYdsDef, ScoreDef, TotalYdsDef, 
         Opponent) %>% 
  mutate(RushYds = RushYdsDef, PassYds = PassYdsDef, Score = ScoreDef, 
         TotalYds = TotalYdsDef, TeamName_temp = TeamName, 
         TeamName = Opponent) %>%
  mutate(Opponent = TeamName_temp) %>%
  select(Date, TeamName, Opponent, RushYds, PassYds, Score, TotalYds) %>%
  arrange(desc(TotalYds))

dat_all <- rbind(dat_off, dat_def) %>% 
  arrange(desc(TotalYds)) %>%
  unique()
```

This represents 23402 individual team performances.

Let's plot to see what is the relationship between total yards and score.


```r
smoothScatter(dat_all$TotalYds, dat_all$Score)
model1 <- lm(Score ~ TotalYds, data = dat_all)
abline(model1, col = "red")
```

![](shutout_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

There are a couple of obvious outliers there with > 2000 total yards of offense. Which ones are they?


```r
as.data.frame(dat_all[which(dat_all$TotalYds > 2000),])
```

```
##         Date TeamName Opponent RushYds PassYds Score TotalYds
## 1 10/18/2003    Akron    U-C-F    1314    1550    38     2864
## 2 10/18/2003    U-C-F    Akron     654    1697    24     2351
```

On 10/18/2003 Akron and University of Central Florida played each other, and reportedly both had > 2000 yards of total offense in a game that was only 38-24. I could not find the box score for this so I deleted these observations and reran the plot and model.


```r
dat_all <- dat_all[-which(dat_all$TotalYds > 2000), ]
smoothScatter(dat_all$TotalYds, dat_all$Score)
model1 <- lm(Score ~ TotalYds, data = dat_all)
abline(model1, col = "red")
```

![](shutout_files/figure-html/unnamed-chunk-13-1.png)<!-- -->

```r
summary(model1)
```

```
## 
## Call:
## lm(formula = Score ~ TotalYds, data = dat_all)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -64.235  -6.461  -0.624   5.805  67.306 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept) -8.626074   0.210074  -41.06   <2e-16 ***
## TotalYds     0.093484   0.000531  176.06   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 9.571 on 22874 degrees of freedom
##   (524 observations deleted due to missingness)
## Multiple R-squared:  0.5754,	Adjusted R-squared:  0.5754 
## F-statistic: 3.1e+04 on 1 and 22874 DF,  p-value: < 2.2e-16
```

Based on the simplistic model, which only accounts for 57% of the variance, we can predict how many points a team that gains 462 yards should score.

```r
predict(model1, newdata = data.frame(TotalYds = 462), interval = "pred")
```

```
##        fit      lwr     upr
## 1 34.56348 15.80386 53.3231
```

The 95% confidence interval for the prediction is 15.8-53.3 points, and 0 points is well outside of this confidence interval.

## Next Steps
Clearly the performances are not all independent (same team may score at a more similar rate per yard than different teams). Also, there is some censoring at the bottom, suggesting that perhaps tobit regression might be a better model since you can't score fewer than 0 points yet you can have fewer than 0 total yards...

There are some outliers where the team scored > 20 points with overall negative yards for the game. These games would be interesting to look at.
