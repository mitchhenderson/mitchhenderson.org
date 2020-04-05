---
title: How to tidy Catapult 10Hz export data
author: Mitch Henderson
date: '2020-04-05'
categories:
  - Data science
  - Sports science
tags:
  - GPS
  - R
coverImage: /img/catapult_cover.png
coverMeta: out
metaAlignment: center
thumbnailImage: /img/catapult_square.jpg
thumbnailImagePosition: left

---

The export format of Catapult's 10Hz GPS exports is ideal for analysis. This article teaches you how you can tidy it using R.

<!--more-->

# Tidy data

It is often claimed that ~80% of data analysis is spent on the process of cleaning and preparing the data ( [Dasu and Johnson 2003](https://onlinelibrary.wiley.com/doi/book/10.1002/0471448354) ). In  2014, [Hadley Wickham](https://twitter.com/hadleywickham) coined the term [tidy data](https://vita.had.co.nz/papers/tidy-data.pdf) to define a dataset structured to facilitate analysis. In the [R for Data Science book's section on tidy data](https://r4ds.had.co.nz/tidy-data.html), for a dataset to be *tidy* it has to follow these 3 interrelated rules:

1. Each variable must have its own column.
2. Each observation must have its own row.
3. Each value must have it's own cell.

![](/img/tidy_data.png)
*Following three rules makes a dataset tidy: variables are in columns, observations are in rows, and values are in cells. Image from [R for Data Science](https://r4ds.had.co.nz/index.html) by Hadley Wickham & Garrett Grolemund.*

# Catapult data 

When you export 10Hz GPS data from Catapult's Openfield software, the `.csv` file produced from each unit looks like this:

![](/img/untidy.png)

![](/img/batman_facepalm.gif)

It isn't great because:

1. Each unit has produced it's own `.csv` file meaning to do any analysis using all players data we need to find a way to combine all the files.
2. There are 8 rows of metadata above the actual data we're primarily interested in, preventing us from combining in it's current format.

Without combining all players' data into one *tidy* table or "*dataframe*", we can't explore the data using the common data exploration tools such as a PivotTable on Microsoft Excel or more advanced data manipulation tools like the `dplyr` R package. 

# Solution

Thankfully for us, we don't need to manually go through each file, deleting the top 8 rows, creating new columns for metadata like player name and match, and then copy pasting them all into the one file. The `tidyverse` suite of R packages can help us to largely automate these manual tasks! 

![](/img/batman_thinking.gif)

I will explain how to do it here (and provide you with the R code), and also show you in the YouTube video below.

## Step 1

Make sure you have [R](https://cran.r-project.org/) and [RStudio](https://rstudio.com/products/rstudio/download/) downloaded and installed on your machine (both free!). 

Open RStudio, copy and paste the code below into a script (you can create a new R script by clicking the symbol directly under 'File' on the top-left of the window, and selecting 'R Script'). 

Save the file as a `.R` file with an appropriate name (e.g. "tidy_catapult_data.R").

```
# Script to combine and tidy multiple Catapult Openfield 10Hz export .csv files by Mitch Henderson
# www.mitchhenderson.org

library(tidyverse)
library(zoo)
library(magrittr)

# Set working directory (ideally you will learn project based workflow in RStudio and this part becomes redundant)

setwd("C:/Users/Mitch.Henderson/folder_containing_folders_of_csv_files")

# Add metadata for these files that will become a column in the final output. This must exactly match the folder name containing the .csv files and MUST be 2 words separated by a space (e.g. "Round 1" or "vs Bulldogs" or "Week 2").

folder_name <- "Round 1" 

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

combine_and_tidy <-
  list.files(path = paste0("./", folder_name, "/"),
             pattern = "*.csv", 
             full.names = T) %>%
    map_df(~read_plus(.))
    
# Output csv into working directory (folder containing the folders of csv files)

write.csv(combine_and_tidy, file = paste0(folder_name, "_tidy.csv"), row.names = F)
```

## Step 2

Export the files from Openfield **without changing their filename** and save them in a folder with an appropriate name **that has 2 parts to it separated by a blank space**. A good example is "Round 1" or "vs Bulldogs" or "Week 2" (these are the examples I've used in the code below). 

The name is important as the code is looking for a 2-part name to become a column in the final output.

## Step 3

Put the folder (or folders) created in Step 1 into a parent folder. Name this anything you like. 

This folder needs to become your working directory when you run the code. If you aren't familiar with a working directory, it is just a file path on your computer that sets the default location of any files you read into R, or save out of R.

In my code below I've manually set the working directory to my Desktop. This is actually poor practice, because no one else will have the same file system as me, but I've done it this way so it's clearer to those new to R how to manually set your working directory. 

I highly recommend you become familiar with [project-oriented workflow](https://www.tidyverse.org/blog/2017/12/workflow-vs-script/) and RStudio's Project capabilities because this essentially shifts working directories for you.

## Step 4

Change the code's `folder_name` variable to the exact folder name (in brackets/parentheses like is currently in the code) containing the `.csv` files you would like to combine and tidy. 

In my code, I've called it "Round 1". 

Like I mentioned in Step 2, for the code to work the name **MUST** be 2 parts separated by a space. 

## Step 5

Highlight the entire code and press Ctrl + Enter (Cmd + Return on a Mac). This will run the code and carry out the tasks of:

1. Pulling in each `.csv` file's data within the folder you've referenced with `folder_name` (assuming that folder is within your working directory).
2. Remove the 8 rows of metadata from the top of each file.
3. Create new columns for player name and activity derived from the `.csv`'s file name generated by Openfield, and a new column for the `folder_name` variable derived from the R code you entered.
4. Combine all files into one *dataframe*.
5. Export this dataframe into a new `.csv` called "`folder_name`_tidy.csv" in your working directory. Where the `folder_name` part of the new file name is what you entered in the code (e.g. the name of my file would be "Round 1_tidy.csv").

![](/img/batcomputer.gif)

# Conclusion & tutorial

Now the data is in *tidy* format and is easy to manipulate and analyse!

![](/img/tidy_gps.png)

![](/img/batman_thumbsup.gif)

You can see me going through all the steps outlined here in this video with some dummy data:

VID HERE

Let me know if this post has helped you or if there's anything else you're interested in learning that I can help with. I'm keen to hear!

**Keep up to date with anything new from me on [my Twitter](https://twitter.com/mitchhendo_).**


Cheers,

Mitch


*Cover image and thumbnail from [catapultsports.com](www.catapultsports.com).*
