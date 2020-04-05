---
categories:
- Sports science
- Data science
coverImage: /img/catapult_cover.png
coverMeta: out
date: "2020-04-06"
metaAlignment: center
tags:
- GPS
- R
thumbnailImage: /img/catapult_square.jpg
thumbnailImagePosition: left
title: How to tidy Catapult 10Hz export data
---

The export format of Catapult's 10Hz GPS exports is ideal for analysis. This article teaches you how you can tidy it using R.

<!--more-->

# Tidy data

It's been suggested that ~80% of data analysis is spent on the process of cleaning and preparing the data ( [Dasu and Johnson 2003](https://onlinelibrary.wiley.com/doi/book/10.1002/0471448354) ). In  2014, [Hadley Wickham](https://twitter.com/hadleywickham) coined the term [tidy data](https://vita.had.co.nz/papers/tidy-data.pdf) to define a dataset structured to facilitate analysis. In the [R for Data Science book section on tidy data](https://r4ds.had.co.nz/tidy-data.html), for a dataset to be *tidy* it has to follow these 3 interrelated rules:

1. Each variable must have its own column.
2. Each observation must have its own row.
3. Each value must have it's own cell.

![](/img/tidy_data.png)
*Following three rules makes a dataset tidy: variables are in columns, observations are in rows, and values are in cells. Image from [R for Data Science](https://r4ds.had.co.nz/index.html) by Hadley Wickham & Garrett Grolemund.*

# Catapult data 

When you export 10Hz GPS data from Catapult's Openfield software, the `.csv` file produced from each unit looks like this:

![](/img/untidy.png)

**Far from ideal**

This isn't great because:

1. Each unit has produced it's own `.csv` file meaning to do any between-player analysis we need to find a way to combine them.
2. There are 8 rows of metadata above the actual data we're primarily interested in, preventing us from combining it in this format.

Without combining all players' data into one *tidy* table or "*dataframe*", we're unable to explore the data using the common data exploration tools such as a pivottable on Microsoft Excel or more advanced data manipulation tools like the `dplyr` R package. 

```
library(tidyverse)
library(zoo)
library(magrittr)

# Add metadata for these files (e.g. opponent or round number) that will become a column in the final output. This must exactly match the folder name containing the .csv files.

folder_name <- "Round 1 vs Bulldogs" 

# This function will read in multiple .csv files, create a column with the file name, and parse it into it's relevant components.

read_plus <- function(flnm) {
  read_csv(flnm, skip = 8)[, c(
    "Timestamp",
    "Latitude",
    "Longitude",
    "Seconds",
    "Velocity",
    "Acceleration",
    "Odometer",
    "Heart Rate",
    "Player Load",
    "Positional Quality (%)",
    "HDOP",
    "#Sats"
  )] %>%
    mutate(Filename = flnm,
           Folder = str_split(Filename, "/")[[1]][2],
           Activity_interim = str_split(Filename, "/")[[1]][3],
           Activity_interim2 = str_split(Activity_interim, " for ")[[1]][1],
           Activity = str_split(Activity_interim2, " Export")[[1]][1],
           Player_interim = str_split(Activity_interim, " for ")[[1]][2],
           Player = paste0(str_split(Player_interim, " ")[[1]][1], " ", str_split(Player_interim, " ")[[1]][2])
    ) %>%
    select(-Filename, -Activity_interim, -Activity_interim2, -Player_interim)
}

# Import data from files contained within the folder specified by `folder_name` above (folder must be in working directory)

tbl_with_sources <-
  list.files(path = paste0("./", folder_name, "/"),
             pattern = "*.csv", 
             full.names = T) %>%
    map_df(~read_plus(.))
```

Cover image and thumbnail from [catapultsports.com](www.catapultsports.com).
