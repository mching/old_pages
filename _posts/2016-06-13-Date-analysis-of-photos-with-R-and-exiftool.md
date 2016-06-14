# Date analysis of photos with R and exiftool

We were fans of Google's Picasa photo organization software, but when it was discontinued, I decided that I would like to transfer all my digital photos onto an external drive and then into Adobe Lightroom. To do so, however, my "research" told me that it would be best to group them into catalogs based on year, mainly because I have on the order of 10^5 photos.

Right now I have my pictures organized into about 300-400 directories by event. I decided I wanted to organize all of my folders as directories within year directories. So all the 2016 folders would need to be sorted into a 2016 directory, all the 2015 folders would go into a 2015 directory, etc.

```
-2016
--Mom's birthday
--Valentine's Day
--Big Island Trip
-2015
--Schofield Days
--Bellows Camp
--etc...
```

The problem was most of the folders are titled things like "Kindergarten Graduation" or "Anacapa". Some of them had the date in them (Maui 2012), or I could remember the date (Our Wedding). But for about 350 of them, there was no date in the folder title, and to figure out when these pictures were from, I would have to manually look at each directory, check the date from the EXIF data, and then sort those into the proper folder. Even if it took 1 minute per folder, that would take me 6 hours. So I decided I could spend 6 hours doing this manually or 6 hours learning some way to automate it. 

A quick search online brought me to [`exiftool`](http://www.sno.phy.queensu.ca/~phil/exiftool/). This is a command line program that allows for reading and writing photo metadata. It has lots of awesome quotes on the webpage, and the one that sealed the deal was this one:

> "... it is total f***ing gibberish to me." - [Reddit Linux Questions](https://www.reddit.com/r/linuxquestions/comments/2yiked/i_want_to_batch_extract_the_exif_datetime_from_10/)

After some fooling around, I discovered that I could extract the date at the command line and send it to a .csv (in Mac OS X). Adding other EXIF attributes allows you to extract more data, but since I didn't really want anything else, I just pulled the filename and the DateTimeOriginal attributes.

```
> exiftool -DateTimeOriginal -S -s -csv ./*/ > all_photos_dates.csv
```

Sweet! Now that's something I can work with!

First, I needed to read in the data then let's take a look:


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
library(stringr)
library(tidyr)
photos <- read.csv(file = "~/Dropbox/Mike/photo_analysis/all_photo_dates.csv")
photos <- tbl_df(photos)
photos
```

```
## Source: local data frame [24,129 x 2]
## 
##                           SourceFile     DateTimeOriginal
##                               <fctr>               <fctr>
## 1         ./2010a/OwenAki2010_06.JPG  2010:06:28 19:29:37
## 2    ./2010a/OwenAkiKCS_Sept2010.jpg                     
## 3        ./BOB Stroller/CIMG2414.JPG  2007:07:16 15:12:32
## 4        ./BOB Stroller/CIMG2415.JPG  2007:07:16 15:12:46
## 5        ./BOB Stroller/CIMG2416.JPG  2007:07:16 15:12:54
## 6        ./BOB Stroller/CIMG2417.JPG  2007:07:16 15:13:41
## 7  ./Blueberry Festival/P7220001.JPG 2006:07:22 13:31:47Z
## 8  ./Blueberry Festival/P7220002.JPG 2006:07:22 13:31:53Z
## 9  ./Blueberry Festival/P7220003.JPG 2006:07:22 13:40:04Z
## 10 ./Blueberry Festival/P7220004.JPG 2006:07:22 13:40:10Z
## ..                               ...                  ...
```

We can see that the `exiftool` command read in the file names in the format "./directoryname/filename". I need to split out that directory name and the component files. We can do it using the `separate` function from `tidyr`.


```r
photos <- photos %>% separate(SourceFile, c("dot", "dname", "fname"), sep = "/", remove = TRUE)
photos <- photos[,2:4]
photos
```

```
## Source: local data frame [24,129 x 3]
## 
##                 dname                   fname     DateTimeOriginal
##                 <chr>                   <chr>               <fctr>
## 1               2010a      OwenAki2010_06.JPG  2010:06:28 19:29:37
## 2               2010a OwenAkiKCS_Sept2010.jpg                     
## 3        BOB Stroller            CIMG2414.JPG  2007:07:16 15:12:32
## 4        BOB Stroller            CIMG2415.JPG  2007:07:16 15:12:46
## 5        BOB Stroller            CIMG2416.JPG  2007:07:16 15:12:54
## 6        BOB Stroller            CIMG2417.JPG  2007:07:16 15:13:41
## 7  Blueberry Festival            P7220001.JPG 2006:07:22 13:31:47Z
## 8  Blueberry Festival            P7220002.JPG 2006:07:22 13:31:53Z
## 9  Blueberry Festival            P7220003.JPG 2006:07:22 13:40:04Z
## 10 Blueberry Festival            P7220004.JPG 2006:07:22 13:40:10Z
## ..                ...                     ...                  ...
```

Ok, next thing we need to do is to take care of that date column and put it into a form that R can work with. I used POSIXct because POSIXlt caused problems when trying to add it to the data frame. This is because POSIXlt is a list, and POSIXct represents the number of seconds since the beginning of 1970.


```r
photos$DateTimeOriginal <- as.POSIXct(strptime(photos$DateTimeOriginal, format = "%Y:%m:%d %H:%M:%S"))
```

One of the problems with POSIXct is that it's not as easy to get the year out compared to POSIXlt. No problem, we can just temporarily convert to POSIXlt and add 1900 (the start date of POSIXlt years).


```r
photos <- photos %>% mutate(year = as.POSIXlt(DateTimeOriginal)$year + 1900)
photos
```

```
## Source: local data frame [24,129 x 4]
## 
##                 dname                   fname    DateTimeOriginal  year
##                 <chr>                   <chr>              <time> <dbl>
## 1               2010a      OwenAki2010_06.JPG 2010-06-28 19:29:37  2010
## 2               2010a OwenAkiKCS_Sept2010.jpg                <NA>    NA
## 3        BOB Stroller            CIMG2414.JPG 2007-07-16 15:12:32  2007
## 4        BOB Stroller            CIMG2415.JPG 2007-07-16 15:12:46  2007
## 5        BOB Stroller            CIMG2416.JPG 2007-07-16 15:12:54  2007
## 6        BOB Stroller            CIMG2417.JPG 2007-07-16 15:13:41  2007
## 7  Blueberry Festival            P7220001.JPG 2006-07-22 13:31:47  2006
## 8  Blueberry Festival            P7220002.JPG 2006-07-22 13:31:53  2006
## 9  Blueberry Festival            P7220003.JPG 2006-07-22 13:40:04  2006
## 10 Blueberry Festival            P7220004.JPG 2006-07-22 13:40:10  2006
## ..                ...                     ...                 ...   ...
```

Now we want to get a summary of the years of the photos in each folder.


```r
Mode <- function(x) {
  ux <- unique(x)
  ux[which.max(tabulate(match(x, ux)))]
}

folder_years <- photos %>% group_by(dname) %>% summarize(year = Mode(year))
folder_years
```

```
## Source: local data frame [332 x 2]
## 
##                      dname  year
##                      <chr> <dbl>
## 1                    2010a  2010
## 2       Blueberry Festival  2006
## 3             BOB Stroller  2007
## 4           Boston Day Out  2006
## 5   Boston Police T-shirts  2007
## 6  Braden's Birthday Party  2009
## 7                  Bubbles  2006
## 8                  Bunnies  2006
## 9                 Cabrillo  2002
## 10             Camp Milken  2005
## ..                     ...   ...
```

Finally it'd be great if we could get a list of all the folders corresponding to each year.


```r
table(folder_years$year, useNA = "ifany")
```

```
## 
## 2001 2002 2003 2004 2005 2006 2007 2008 2009 2010 2011 2012 2013 2014 2015 
##    1   18    4    9   23   53   48   19   23   20   29   26   15    5    2 
## 2016 <NA> 
##    1   36
```


```r
folder_output <- list()

for(i in sort(unique(folder_years$year))) {
  x <- folder_years %>% filter(year == i)
  folder_output[as.character(i)][[1]] <- x[,1]
}

folder_output
```

```
## $`2001`
## Source: local data frame [1 x 1]
## 
##       dname
##       <chr>
## 1 Las Vegas
## 
## $`2002`
## Source: local data frame [18 x 1]
## 
##                      dname
##                      <chr>
## 1                 Cabrillo
## 2           Chicken Bridge
## 3           Corona del Mar
## 4                   dinner
## 5              Dodger Game
## 6     Greg's Holiday Party
## 7        Ho'ike Hula night
## 8           Hollywood Bowl
## 9                 Holy Jim
## 10        Jimmy Graduation
## 11             Joshua Tree
## 12 Malibu Creek State Park
## 13                    Misc
## 14                Mt. Lowe
## 15                 Na Mamo
## 16           Redondo Beach
## 17  Su-Yu's farewell lunch
## 18               Surfliner
## 
## $`2003`
## Source: local data frame [4 x 1]
## 
##                   dname
##                   <chr>
## 1 Graduation Med School
## 2     Sturtevant Canyon
## 3          Trail Canyon
## 4    Vancouver Marathon
## 
## $`2004`
## Source: local data frame [9 x 1]
## 
##                         dname
##                         <chr>
## 1                  Fruit Tart
## 2            Gus Fawn Wedding
## 3    Leng and Nan's Birthdays
## 4             Mike's Birthday
## 5 Miscellaneous Family Photos
## 6  Namphol and Vivian Wedding
## 7                  Nisei Week
## 8                    seaworld
## 9              Virgin Islands
## 
## $`2005`
## Source: local data frame [23 x 1]
## 
##                        dname
##                        <chr>
## 1                Camp Milken
## 2                   Car seat
## 3             Dad's Pictures
## 4  Demetra's Christmas Party
## 5   El Capitan Canyon-Milken
## 6                  Feb Bdays
## 7            First D50 shots
## 8             Fourth of July
## 9                Hanauma Bay
## 10                    Hawaii
## ..                       ...
## 
## $`2006`
## Source: local data frame [53 x 1]
## 
##                                      dname
##                                      <chr>
## 1                       Blueberry Festival
## 2                           Boston Day Out
## 3                                  Bubbles
## 4                                  Bunnies
## 5                                 Cape Cod
## 6                                Car Seats
## 7  Childrens' Book Festival and Greek Fest
## 8                               Ching Ming
## 9                                   Crafts
## 10                                Descanso
## ..                                     ...
## 
## $`2007`
## Source: local data frame [48 x 1]
## 
##                                         dname
##                                         <chr>
## 1                                BOB Stroller
## 2                      Boston Police T-shirts
## 3                                       Cards
## 4            Childrens' Museum with Alamillos
## 5                                 Christopher
## 6                              Day in Waltham
## 7                            DeCordova Museum
## 8       Drumlin Farm - Sap to Syrup Breakfast
## 9                      Emerson's 1st Birthday
## 10 Eric Carle Museum - Allen Say Book Signing
## ..                                        ...
## 
## $`2008`
## Source: local data frame [19 x 1]
## 
##                           dname
##                           <chr>
## 1      Emerson's Birthday Party
## 2          Flat Stanley's Visit
## 3          Frog Went A Courtin'
## 4      Grant's Birthday Party_2
## 5                         Knits
## 6    Melissa and Michelle visit
## 7        Mike Passport Pictures
## 8                       Molokai
## 9             Mom's Club Brunch
## 10                 Owen at CCMS
## 11 Phoebe Announcement Pictures
## 12                 Phoebe-Birth
## 13         Phoebe's Impressions
## 14             Printed Pictures
## 15                        Raffi
## 16         Ronak's 2nd Birthday
## 17      Sheep Shearing Festival
## 18           Thida-Matt Wedding
## 19                      To sell
## 
## $`2009`
## Source: local data frame [23 x 1]
## 
##                              dname
##                              <chr>
## 1          Braden's Birthday Party
## 2                     CDs in box A
## 3                    Dunns Visit_1
## 4   Edaville - Day Out with Thomas
## 5                     Eric's Visit
## 6          Farewell BBQ at Ferry's
## 7                      Ferry Visit
## 8                  Furlough Friday
## 9  Honolulu Holiday Lights Trolley
## 10       Jake's 1st Birthday Party
## ..                             ...
## 
## $`2010`
## Source: local data frame [20 x 1]
## 
##                             dname
##                             <chr>
## 1                           2010a
## 2           Dayton's 5th birthday
## 3          Gus Ryan Phillip Visit
## 4               Hannah's Birthday
## 5                         Ihilani
## 6  Jeff and Lynne Wedding Weekend
## 7           Jenica's 4th birthday
## 8                    Kathy Photos
## 9            Kawaiaha'o Beach Day
## 10                      La Pietra
## 11            La Pietra - MS Camp
## 12            La Pietra Class Day
## 13                 LAMC fieldtrip
## 14                 Lyon Arboretum
## 15                Owen's drawings
## 16     Science 7 - Animal Project
## 17          Shellie Bridal Shower
## 18                Shellie Wedding
## 19               Stegosaurus Walk
## 20           Toren's 3rd birthday
## 
## $`2011`
## Source: local data frame [29 x 1]
## 
##                                      dname
##                                      <chr>
## 1                       Dunn's baby shower
## 2                  Elijah's Birthday Party
## 3                    Hannie's 3rd Birthday
## 4              Hannie's 3rd Birthday Party
## 5                        Home Improvements
## 6                Iolani Christmas Activity
## 7               Iolani co2024 Beach Picnic
## 8  Iolani co2024 Fieldtrip to Fire Station
## 9               Iolani co2024 Thanksgiving
## 10        Iolani Kindergarten Beach Picnic
## ..                                     ...
## 
## $`2012`
## Source: local data frame [26 x 1]
## 
##                                              dname
##                                              <chr>
## 1                            Den 8-Honolulu Museum
## 2                                 Den 8-KITV Visit
## 3                          Evan 4th birthday party
## 4                              Evan's 4th Birthday
## 5                                Hannie's Birthday
## 6                   Hero Factory Geo Tank Breakout
## 7                               Hydroponic Lettuce
## 8                        Iolani 100 days of school
## 9  Iolani co2024 Fieldtrip Foster Botanical Garden
## 10                         Iolani co2024 headshots
## ..                                             ...
## 
## $`2013`
## Source: local data frame [15 x 1]
## 
##                                 dname
##                                 <chr>
## 1              Camping 2024 2nd grade
## 2                   Cub Scout Bowling
## 3  Cub Scouts Pacific Aviation Museum
## 4                Disney Alaska Cruise
## 5                  EFMP holiday party
## 6                 Evan's 5th Birthday
## 7                 Fairy Tale Festival
## 8                         Girl Scouts
## 9                       KCS Ho'omoana
## 10                 Nan MEd Graduation
## 11                Owen's 7th Birthday
## 12            Pack325 Aiea Loop Trail
## 13           Pack325 Kaena Point Hike
## 14              Sarah Andrea Ceremony
## 15                  Snooze at the Zoo
## 
## $`2014`
## Source: local data frame [5 x 1]
## 
##                               dname
##                               <chr>
## 1                  co2026 Hoolaulea
## 2 Evan and Aki's 6th Birthday Party
## 3              Gloria-Katrina Visit
## 4             Nick-Michaela Wedding
## 5               Owen's 8th Birthday
## 
## $`2015`
## Source: local data frame [2 x 1]
## 
##                            dname
##                            <chr>
## 1          Den 8 Obstacle Course
## 2 Matthew Tom 1st Birthday Party
## 
## $`2016`
## Source: local data frame [1 x 1]
## 
##                  dname
##                  <chr>
## 1 Owen's 10th Birthday
```

