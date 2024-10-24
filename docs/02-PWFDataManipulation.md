# Data Manipulation {#Start2}



In this Chapter, we will import the raw data and wrangle it into a format suitable for creating summary statistics. 

## Installing packages {#Start2.1}

The functionality of some packages may require updated versions of [R](https://www.r-project.org/) and [RStudio](https://www.rstudio.com). To avoid errors, please ensure you are using the most recent releases of R and RStudio, and update your R packages.


```r
update.packages()       

# may need to run this as an R administrator if you run into 'permission denied' issues
```

You need to load the package each time you open a new R session. 


```r
#If you need to install the package, use the following script before loading the package. You only need to install the package once on your computer, and then it will always be available. For example: 

#install.package("tidyverse")

require(tidyverse)
require(lubridate)
require(data.table)
require(reshape)
require(tibble)
```

## Working Directory {#Start2.2}

First, lets set up your working directory. If you don't know what your current working directory is you can check with the following script. 


```r
getwd()
```

If you want to save your data in a particular location on your computer, set your directory with the following script. 


```r
#This is just an example that will need to be modified to the path on your computer

setwd("C:/Users/username/Documents/foldername/")
```

Alternatively, you can set the working directory using the `Session` tab > `Set Working Directory` > `Choose Directory` 

Now we will a create folder to save the raw data and store any outputs. We will also set shortcuts to these folders for future use. 


```r
dir.create("Data")
dir.create("Output")

out.dir <- paste("Output/")
dat.dir <- paste("Data/")
```

## Data download and filtering {#Start2.3}

Researchers seeking to conduct formal analyses using PFW data are invited to download the raw data from the Cornell Lab PFW [website](https://feederwatch.org/explore/raw-dataset-requests/). As with the use of any data set, knowing the data structure, understanding the metadata, grasping the data collection protocols, and being cognizant of the unique aspects of the program are all critical for conducting analyses and interpreting results in ways that provide meaningful insights. Although the data are freely available, we invite researchers to consult with researchers at the Cornell Lab of Ornithology or Birds Canada to ensure that the data are being handled and analyzed in a meaningful way.

> Save the raw data in the "Data" folder you created in the previous step. In this example, we are going to work with all the Canadian PFW data. This is a *big* dataset, so we will process it in batches, otherwise R gets cranky.

**Step 1: Load Data**

Now that you have all the raw data saved in your `Data` folder, we are going to process each file sequentially to extract what we need. Each time you load a new dataframe you will overwrite the old 'canada.pfw' dataframe. 

> Notice we are using `fread` instead of `read.csv`. It is faster for big data files. 


```r
#Process sequentially. Load one file and move to Step 2.

#Old data
canada.pfw<-fread("Data/PFW_1988_1995_public.csv")
canada.pfw<-fread("Data/PFW_1996_2000_public.csv")
canada.pfw<-fread("Data/PFW_2001_2005_public.csv")
canada.pfw<-fread("Data/PFW_2006_2010_public.csv")
canada.pfw<-fread("Data/PFW_2011_2015_public.csv")
canada.pfw<-fread("Data/PFW_2016_2020_public.csv")

#New data
canada.pfw<-fread("Data/PFW_2021_public.csv")
#New data 2021-2022 from Cornell. Just Canadian data for analysis. 
canada.pfw<-fread("Data/PFW_Canada_2021_2022.csv")
```

**Step 2: Filter Data** 

Now that you have loaded your first dataframe filled with PFW data. Now we are going to filter and clean the data to ensure it is ready for future processing and analysis. 

Change the uppercase headers to lower case, since it appears that there is a mix of both, depending on the year. 


```r
names(canada.pfw)<-tolower(names(canada.pfw))
```

Filter out the U.S. data and retain the Canadian data only. I also remove unwanted data columns. You can change this code to filter for just U.S. data, or to remove/ maintain any data column you would like for your analytical purposes. 


```r
canada.pfw <- canada.pfw %>% 
  filter(subnational1_code %in%c("CA-ON","CA-SK","CA-BC","CA-QC","CA-MB","CA-AB","CA-NB","CA-NS","CA-NL","CA-PE","CA-YT","CA-NT")) %>%
  dplyr::select(-entry_technique, -data_entry_method) %>% 
  collect()
```

Here I separate the provincial code into its own column and remove 'pfw' from the Period id


```r
canada.pfw<-canada.pfw %>% 
  separate(subnational1_code, c("del1", "Prov"), sep="-", remove=FALSE) %>% 
  dplyr::select (-del1, -subnational1_code) %>% 
  separate(proj_period_id, c("del2", "Period"), sep="_", remove=FALSE) %>%
  dplyr::select(-del2, -proj_period_id)
```

To eliminate biases created by extending the PFW season in some years, data are truncated to after Nov 8 (doy=312) (the earliest possible 2nd Saturday) and end Apr 3 (doy=93) (earliest possible end date). 


```r
 canada.pfw <- canada.pfw %>%
  mutate(date = ymd(paste(year, month, day, sep = "/")), doy = yday(date)) %>% 
  filter(doy <= 93 | doy >= 312)
```

Now we assign a `floor.week` variable which starts on Saturday (i.e., +6)


```r
canada.pfw <- canada.pfw %>% mutate(floor.week=floor_date(canada.pfw$date,unit="week")+6)
```

We also assign `region` to the Canadian data set since several provinces are grouped for the purpose of summary statistics. 


```r
canada.pfw <- canada.pfw %>% 
  mutate(region = ifelse(Prov=="ON", "ON", ifelse (Prov=="BC", "BC", ifelse(Prov == "AB", "PR", ifelse(Prov=="SK", "PR", ifelse(Prov=="MB", "PR", ifelse(Prov=="QC", "QC", ifelse(Prov=="NB", "AT", ifelse(Prov=="NS", "AT", ifelse(Prov=="PE", "AT", ifelse(Prov=="NL", "AT", ifelse(Prov=="YT", "North", ifelse(Prov=="NT", "North", NA)))))))))))))
```

Now, we need to remove invalid records as defined by [Bonter and Greig 2021](https://www.frontiersin.org/articles/10.3389/fevo.2021.619682/full). 

Flagged observations are identiﬁed in the database as “0” in the VALID ﬁeld and their status in the review process is described using a combination of the VALID ﬁeld and the REVIEWED ﬁeld as deﬁned here:

VALID = 0; REVIEWED = 0; Interpretation: Observation triggered a ﬂag by the automated system and awaits the review process. Note that such observations are removed.

VALID = 0; REVIEWED = 1; Interpretation: Observation triggered a ﬂag by the automated system and was reviewed; insuﬃcient evidence was provided to conﬁrm the observation. Note that such observations are removed.

VALID = 1; REVIEWED = 0; Interpretation: Observation did not trigger the automatic ﬂagging system and was accepted into the database without review.

VALID = 1; REVIEWED = 1; Interpretation: Observation triggered the ﬂagging system and was approved by an expert reviewer.

Based on these descriptions, we remove all VALID == 0, keep all VALID == 1. We do not use the REVIEW column at this time. 


```r
canada.pfw <- canada.pfw %>% filter(valid==1)
```

**Step 3: Assign Species**

Before we proceed we are going to add some data columns that will help us with reporting. Specifically, we want to include the `REPORT_AS`, `CATEGORY` and `PRIMARY_COM_NAME` fields in the `PFW_species_code.csv`, which is part of the [PFW Data Dictionary](https://drive.google.com/file/d/1kHmx2XhA2MJtEyTNMpwqTQEnoa9M7Il2/view). 
This csv file is located in the `Data` folder on GitHub or can be accessed directly at the link provided. However, note that the csv file provided on Github has been cleaned to only retain the worksheet/ data columns we need. 


```r
# Load the PFW species list
sp<-read.csv("Data/PFW_species_codes.csv")
sp<- sp %>% dplyr::select(SPECIES_CODE, REPORT_AS, CATEGORY, PRIMARY_COM_NAME)

# Join the tables by species_code
canada.pfw<-left_join(canada.pfw, sp, by=c("species_code"="SPECIES_CODE"))
```

**Step 4: Save Data**

Now that you have processed the data that you want, you can save it locally in your `Data` folder. Since we process each raw data file in sequence, each will have its own unique file name. 


```r
min.yr<-min(canada.pfw$year)
max.yr<-max(canada.pfw$year)
  
write.table(canada.pfw, file = paste(dat.dir,"PFW_Canada_",min.yr,"-",max.yr, ".csv", sep=""), row.names = FALSE, col.name = TRUE, append = FALSE, quote = FALSE, sep = ",")
```

**Step 5: Start Again**

Assuming you want to work with more than one data file, start back at [Step One](#Start2.3) and process the next raw data file. Once you are done processing your data, you can move to [Chapter 3](#Zero3).
