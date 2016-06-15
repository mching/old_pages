---
layout: post
title: Date analysis of photos with R and exiftool
---

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

Now we want to get a summary of the years of the photos in each folder. I used the mode function from [here](http://stackoverflow.com/questions/2547402/is-there-a-built-in-function-for-finding-the-mode) to find out what was the most common year of the photos in each folder.


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

Finally it'd be great if we could get a list of all the folders corresponding to each year. To do this, I created an empty list then populated it based on the most common year of the photos in each folder.


```r
folder_output <- list()

for(i in sort(unique(folder_years$year))) {
  x <- folder_years %>% filter(year == i)
  folder_output[as.character(i)][[1]] <- x[,1]
}
```

It worked great, and here's a sampling:


```r
folder_output[["2010"]]
```

```
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
```

