---
title: Testing Website 
Date: 2025-12-04
---

# Parking Violations in NYC

## Data

For this assignment, we are going to investigate data on parking violations in NYC. 

### Parking violations in 2020/21

NYC Open Data has data on all [parking violations issued in NYC](https://data.cityofnewyork.us/City-Government/Parking-Violations-Issued-Fiscal-Year-2021/pvqr-7yc4) since 2014. The updated dataset provided for 2025 currently includes about 26 million observations. To make the assignment manageable, I have reduced it to a subset of tickets issued in Jan 2025 and to Manhattan precincts only, yielding about 200 thousand tickets.

Two support files are also included in the `parking` sub folder:

  - the **descriptions of all variables**
  - the **dictionary of violation codes**
  
### Police Precincts

A second data source is the [shape files of police precincts in NYC](https://www1.nyc.gov/site/planning/data-maps/open-data/districts-download-metadata.page). 

## Exercise

### 1. Data exploration

Before focusing on the spatial part of the data, let's explore the basic patterns in the data. 

```{r}
#Libraries 
library(tidyverse)
library(leaflet) # interactive maps 
library(sf) # loading shapefiles 
library(DT) # interactive charts 
library(readxl)
library(RColorBrewer)
library(patchwork)

#setwd
setwd("~/Desktop/Spring 2025/Data Viz/DataViz_Course_Repo/Exercises/07_parking_GRADED/data/parking")

#loading data
park_vio <- read.csv("parkingNYC_Jan2025.csv")

#column names & descriptions 
#col_des <- read.csv("parking_variable_descriptions.csv")

#fines & types of violations codebook 
viol_n_fine <- read.csv("Parking_ViolationCodes_January2020.csv")
```

*color pallet*
```{r}
palette <- c('#f0f9a1', '#d9ed92', '#99d98c', '#76c893', '#52b69a', '#34a0a4', '#168aad', '#1a759f', '#1e6091', '#184e77')

#modified palette
palette_heat <- c('#b5e48c', '#99d98c', '#52b69a', '#168aad', '#1a759f', '#1e6091')
```

#### a) Violation Code and Fine Amounts

Add the violation code descriptions and fine amounts to the data file. For simplicity, ignore the differing fine amounts above and below 96th street. _Simply use the fine amount for below 96th street for all violations._

Provide a visual overview of the top 10 most common types of violations. Compare how this ranking differs if we focus on the total amount of revenue generated.

```{r}
#selecting & renaming necessary vars from viol_n_fine
viol_n_fine <-  viol_n_fine %>% 
  select(VIOLATION.CODE, VIOLATION.DESCRIPTION, Manhattan..96th.St....below..Fine.Amount...) %>%
  rename(Violation.Code = VIOLATION.CODE,
         violation.descr = VIOLATION.DESCRIPTION,
         fine_blw96th = Manhattan..96th.St....below..Fine.Amount...)

#merged dataframe 
df_vio_fines <- left_join(park_vio, viol_n_fine, by = "Violation.Code")
df_vio_fines

#SELECTING NECESSARY VARS FROM MAIN DF 
df_vio_fines <- df_vio_fines %>% 
  dplyr::select(Violation.Code, violation.descr, fine_blw96th, Vehicle.Color, Vehicle.Year, Plate.ID, everything())

```
