---
layout: post
title: Pitchers with More Saves than Strikeouts
---

On my favorite fantasy baseball podcast, the host mentioned the phenomenon of pitchers who had more saves than strikeouts. This seemed like it would be fairly uncommon since it seems that many closers are power pitchers (e.g., Aroldis Chapman). I decided to investigate further using the Lahman database.

# Methods
I used the Lahman database again, which is perfect for answering this kind of questions. It has a Pitching table that contains season long stats for every pitcher from 1871 to 2015 (as of this post).

```r
library(Lahman)
library(tidyverse)
data(Pitching)
data(Master)
x <- tbl_df(Pitching)
```

It was a simple enough query using the `tidyverse`. I decided to only look at the last 40 years since the modern closer is a relatively recent concept. 

## Results
The list of pitchers with more saves than strikeouts encompassed 23 players, of whom four had only 2 saves in that particular season. Todd Jones had more saves than strikeouts 3 years in a row. The biggest discrepancy was Danny Kolb in 2004 when he 39 saves with only 21 strikeouts. 

This may be an increasingly rare phenomenon. In the last 6 years, there has only been one year where a pitcher had more saves than strikeouts. 


```r
x %>% select(playerID, yearID, teamID, IPouts, SV, SO) %>% 
  filter(SV>SO, yearID>1977) %>% 
  mutate(IP = round(IPouts/3, 1)) %>%
  inner_join(Master) %>% 
  select(yearID, nameFirst, nameLast, teamID, IP, SO, SV) %>%
  as.data.frame()
```

```
## Joining, by = "playerID"
```

```
##    yearID nameFirst    nameLast teamID    IP SO SV
## 1    1980       Don   Stanhouse    LAN  25.0  5  7
## 2    1982      Dave     Gumpert    DET   2.0  0  1
## 3    1983       Don      Carman    PHI   1.0  0  1
## 4    1984       Dan Quisenberry    KCA 129.3 41 44
## 5    1987      Gene      Garber    KCA  14.3  3  8
## 6    1987    Dickie       Noles    DET   2.0  0  2
## 7    1990      Rich      Garces    MIN   5.7  1  2
## 8    1991      Dave       Smith    CHN  33.0 16 17
## 9    2000       Bob     Wickman    CLE  26.7 11 14
## 10   2002      Mike    Williams    PIT  61.3 43 46
## 11   2003      Mike    Williams    PIT  37.3 20 25
## 12   2004     Danny      Graves    CIN  68.3 40 41
## 13   2004     Danny        Kolb    MIL  57.3 21 39
## 14   2004      Jose        Mesa    PIT  69.3 37 43
## 15   2005     Danny      Graves    CIN  18.3  8 10
## 16   2005    Dustin   Hermanson    CHA  57.3 33 34
## 17   2005    Braden      Looper    NYN  59.3 27 28
## 18   2005       Bob     Wickman    CLE  62.0 41 45
## 19   2006      Todd       Jones    DET  64.0 28 37
## 20   2007      Todd       Jones    DET  61.3 33 38
## 21   2008      Todd       Jones    DET  41.7 14 18
## 22   2009     Brian     Fuentes    LAA  55.0 46 48
## 23   2012       Jim     Johnson    BAL  68.7 41 51
```

## Conclusion
It isn't common to have more saves than strikeouts. This has happened about 23 times in the last 40 years, or about once every other year.
