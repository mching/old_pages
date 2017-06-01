---
layout: post
title: AAP Annual Leadership Forum Consent Calendar Resolutions Less Likely to Make Top 10 
---

Last month I went to the American Academy of Pediatrics' Annual Leadership Forum. This is a meeting of local chapter leaders from across the country where we make resolutions for the AAP to act on. Some resolutions were passed quickly as a group because they were not controversial. This process is called the "consent calendar". The other resolutions were debated, sometimes at much length, before being voted on individually. At the end of the meeting we came up with the top 10 resolutions that we wanted the AAP to work on this year. It seemed like the consent calendar resolutions should be more likely to be in the top 10 since they are not controversial.  However, it actually looks like they are underrepresented as top 10 resolutions. I decided to run an analysis to see the magnitude of this effect.

## Methods
Data were entered from records of the 2015-2017 ALF sessions. I recorded the numbers of the resolutions, whether they were on the consent calendar, whether they were passed, and whether they were on the top 10. I also recorded the reference committee that they were assigned to (Advocacy, Practice, or AAP Operations).



```r
library(dplyr)
library(ggplot2)
library(binom)

dat <- read.csv("../datasets/ALF.csv")
dat <- tbl_df(dat)
dat$id <- as.character(dat$id)
```

I looked at the pass rate overall and by committee. I also looked at whether there was an association between being on the consent calendar and being on the top 10 list at the end of the meeting.

## Results
### Consent Calendar
Approximately 34% (95% confidence interval 29-40%) of all resolutions were placed on the consent calendar.


```r
table(dat$consent1)
```

```
## 
## FALSE  TRUE 
##   210   110
```

```r
binom.confint(110, 110+210, method = "exact")
```

```
##   method   x   n    mean     lower     upper
## 1  exact 110 320 0.34375 0.2918095 0.3986143
```

### Passage Rates
Overall 93% (90-95%) of resolutions passed. 

```r
table(dat$disposition)
```

```
## 
## Defeated   Passed 
##       23      303
```

```r
binom.confint(303, 23+303, method = "exact")
```

```
##   method   x   n      mean     lower     upper
## 1  exact 303 326 0.9294479 0.8960159 0.9547533
```

If not on the consent calendar, the rate of passing was lower at 89% (84-93%).


```r
table(dat$disposition, dat$consent1)
```

```
##           
##            FALSE TRUE
##   Defeated    23    0
##   Passed     187  110
```

```r
binom.confint(187, 187+23, method = "exact")
```

```
##   method   x   n      mean     lower     upper
## 1  exact 187 210 0.8904762 0.8402022 0.9292922
```

There were not much data on pass rates per committee because of the sample size. 


```r
table(dat$disposition, dat$committee)
```

```
##           
##              A   B   C
##   Defeated   8   6   9
##   Passed   119 108  76
```

### Top 10 Resolutions and Consent Calendar
Top 10 rates by consent were as listed below. The total top 10 over the 3 years did not add to 30 because several late resolutions were placed in the top 10. In fact, there were only 6 late resolutions, and 3 of these became top 10 resolutions.


```r
table(consent=dat$consent1, top10=dat$top10)
```

```
##        top10
## consent FALSE TRUE
##   FALSE   187   23
##   TRUE    106    4
```

I looked at the probability of being on the top 10 list for consent calendar resolutions and for resolutions not on the consent calendar. The consent calendar resolutions had a 3.6% (1.0-9.0%) chance of being on the top 10. The non-consent calendar resolutions had an 11% (7.1-16%) chance of being on the top 10. 


```r
binom.confint(4, 110, method = "exact")
```

```
##   method x   n       mean       lower      upper
## 1  exact 4 110 0.03636364 0.009995204 0.09049205
```

```r
binom.confint(23, 210, method = "exact")
```

```
##   method  x   n      mean      lower     upper
## 1  exact 23 210 0.1095238 0.07070775 0.1597978
```

This difference was statistically significant (p = 0.03). The odds ratio estimate was 0.31 for being on the top 10 if a resolution was in the consent calendar versus not being on the consent calendar.


```r
fisher.test(table(consent=dat$consent1, top10=dat$top10))
```

```
## 
## 	Fisher's Exact Test for Count Data
## 
## data:  table(consent = dat$consent1, top10 = dat$top10)
## p-value = 0.03253
## alternative hypothesis: true odds ratio is not equal to 1
## 95 percent confidence interval:
##  0.07536158 0.93458320
## sample estimates:
## odds ratio 
##  0.3077351
```

## Discussion
There was a statistically significant difference in the likelihood of being placed in the top 10 for resolutions that were on the consent calendar. One reason for this could be that resolutions on the consent calendar do not have someone arguing their merits before the assembled leaders like the other ones. The result is that the odds of being successfully placed in the top 10 are about 1/3 for these non-controversial resolutions.

At the beginning of the meeting, the moderator warned us not to try to remove items from the consent calendar to discuss in front of the group. He said that this might open a resolution to being rejected. However, if being in the top 10 is a desirable position, it would seem like you have a much higher chance of being a top 10 and not that much worse a chance of failing (100% vs ~90%).

Another notable finding was that of the 6 late resolutions reviewed, 3 of them made the top 10. The reason for this is probably because late resolutions reflect more timely events. For example, this year's executive orders on traveler bans led to a couple of late resolutions that were ultimately adopted in the top 10. Having extra special attention as a late resolution is probably also helpful.

## Conclusion
ALF consent calendar resolutions are only about 1/3 as likely to make the top 10 resolutions as non-consent calendar resolutions.
