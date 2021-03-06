---
output: html_document
editor_options: 
  chunk_output_type: console
---
# Zero-fill Matrix {#Zero3}



Many analyses require that you have records not only of when/where species were detected, but also where they were not detected. PFW contains records of species presence, however, we can infer species absence when a species was not detected during a survey, if the survey was done within the bounds of the species range. 

> Let's start this Chapter by reloading our libraries and resetting our working directories in the event you took a break and are starting a new session with your newly filtered and cleaned data set. 


```r
require(tidyverse)
require(reshape)
require(data.table)
require(tibble)
require(raster)
require(sp)
require(rgeos)
require(rgdal)

out.dir <- paste("Output/")
dat.dir <- paste("Data/")
```

## Load Filtered Data {#Zero3.1}

First we will load each of the filtered and cleaned PFW dataset into your working Environment under a different dateframe name.  


```r
#Old data
dat1<-fread("Data/PFW_Canada_1988-1995.csv")
dat2<-fread("Data/PFW_Canada_1995-2000.csv")
dat3<-fread("Data/PFW_Canada_2000-2005.csv")
dat4<-fread("Data/PFW_Canada_2005-2010.csv")
dat5<-fread("Data/PFW_Canada_2010-2015.csv")
dat6<-fread("Data/PFW_Canada_2015-2020.csv")

#New data
dat7<-fread("Data/PFW_Canada_2020-2021.csv")
dat8<-fread("Data/PFW_Canada_2021-2022.csv")
```

Notice that the newest dateset provided by Cornell Labs is missing the `plus_code` (i.e., dat7 has one fewer column). We will add this column in the right spot to make the bind possible. 


```r
dat7<-add_column(dat7, plus_code = 0, .after = 12)

#Data directly from Cornell formatted slightly differently. 
dat8<-dat8 %>% dplyr::select(-alt_full_spp_code, -user_id, -observer_id, -housing_density, -no_birds)
```

Bind and write your full Canadian PFW dataset to the `Data` directory for use later in the analysis


```r
data<-rbind(dat1, dat2, dat3, dat4, dat5, dat6, dat7, dat8)

write.table(data, paste(dat.dir,"PFW_Canada_All.csv", sep=""), row.names = FALSE, col.name = TRUE, append = FALSE, quote = FALSE, sep = ",")
```

## Species Range {#Zero3.2}

We will zero-fill the data two different ways. Why? because one way is used for standardized reporting by Cornell and Birds Canada, and the second way is more biologically appropriate and should be  adopted for  research purposes. 

For reporting purposes, we will zero-fill the data matrix based on Province (or State). Specifically, if a species was detected >10 time in the past 10 years within a specific Province, it will be considered in range.  

First, lets create the zero-fill range file for each Province. First, we creating a master list that links all `loc_id` from PFW to Province, the we determine if a species was detected >=10 times. This is done using a loop, which creates a Range output table. Note that we use the `REPORT_AS` for the species ID field, *not* `species_code` (which would at first glance seems to make sense, but in the long run it doesn't work out. Trust me!).


```r
# Using the past 10 years of data to assign range limits for species
dat<-data %>% filter(year>=2010)

# Remove duplicated and NA
master<-dat %>% dplyr::select(Prov) %>% distinct() 

# Write table to your output director for safe keeping. You will use the master list later. 
write.table(master, file = paste(out.dir, "master_prov.csv", sep=""), row.names = FALSE, sep = ",")

sp.list<-unique(data$REPORT_AS) # n = 303 in this example

# Create a species loop
for(m in 1:length(sp.list)) {
  
 # m<-1 #for testing each species
   
  sp.data <-NULL 
  sp.data <- filter(dat, REPORT_AS == sp.list[m]) %>%
      droplevels()

# Count number of observation in each Atlas Block  
  sp.data<-sp.data %>% group_by(Prov) %>%
    dplyr::summarize(nobs=length(how_many)) %>% 
    mutate(count=ifelse(nobs>=5, 1, 0)) %>% 
   dplyr::select(-nobs)

  colnames(sp.data)[colnames(sp.data) == "count"] <- sp.list[m]

#Optional: if there are less than 2 Atlas Blocks containing a record of a species, remove. Considered rare and/or out of range. 
  
# if(nrow(sp.data)<2){
#    sp.data<-NULL
#  }else{
master<-left_join(master, sp.data, by = "Prov" )
#  } #end if statement
  
} #end sp.list loop

master[is.na(master)]=0

# Write your new range table to an output file
write.table(master, file = paste(out.dir, "Range_prov.csv", sep=""), row.names = FALSE, sep = ",")
```

Each row under the species id indicates if the Atlas Block is within range (1) or is out of range (0). 


## Sampling Events {#Zero3.3}

Now that we have a handle on the winter distribution/range of each species, we need to know when each PFW site was sampled. This way we only add a zero-count for a site that was being watched. This is done for each `sub_id` which covers the two-day count period.  

First we create new effort fields. The first sums the number of half days the feeder site was watched (max = 4) and the second changes the effort hours into a factor.  


```r
# Create the full data table. This step is repetitive. 
data<-rbind(dat1, dat2, dat3, dat4, dat5, dat6, dat7, dat8)

# Create effort days field (max = 4)
data<-data %>% mutate(Effort_days = (day1_am +day1_pm +day2_am +day2_pm)/2)

# Create effort hours field that is a factor
data<-data %>% mutate(Effort_hrs = as.factor(effort_hrs_atleast))
levels(data$Effort_hrs)<-c("0_1", "1_4", "4_8", ">8")
```

Now we are ready to create the full sampling events layer using the filtered Canadian PFW dataset.  


```r
event.data <- data %>%
  dplyr::select(loc_id, sub_id, latitude, longitude, month, day, year, Period, Effort_days, Effort_hrs, Prov, region) %>%
  group_by(loc_id, sub_id, latitude, longitude, month, day, year, Period, Effort_days, Effort_hrs, Prov, region) %>%
  distinct() %>%
  ungroup() %>%
  as.data.frame()

# write your new sampling events table to an Output folder
write.table(event.data, paste(out.dir,"Events.csv", sep=""), row.names = FALSE, col.name = TRUE, append = FALSE, quote = FALSE, sep = ",")
```

Now you have the tables that you need to zero-fill your data matrix. We can now start creating data summaries in [Chapter 4](#Sum4). 
