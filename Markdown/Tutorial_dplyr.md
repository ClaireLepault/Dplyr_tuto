---
title: "Data frames and dplyr tutorial"
date: 2023-04-13
output: html_document
---

# Let's load the tidyverse meta-package and check the output.


```{R setup, include = FALSE}
library("tidyverse")
knitr::opts_chunk$set(echo = TRUE,eval=TRUE)
```

The tidyverse actually comes with a lot more packages than those that are just loaded automatically.


```R
tidyverse_packages()
```

# Data frames


```R
?storms
```


```R
head(storms,2)
```


```R
sprintf("%i rows x %i columns", nrow(storms), ncol(storms))
```


```R
storms[1:6,c(1:8, 10:11)] %>% data.frame()
```
---
## Creation
---

```R
data.frame(var1 = 1:5, var2 = "apple", var3 = rnorm(5))
```


```R
# An object with value
tmp <- rnorm(5)
# Creating the data frame
our_df <- data.frame(var1 = 1:5, var2 = "apple", var3 = tmp)
```

## Naming


```R
names(our_df)
```


```R
# Set new names
names(our_df) <- c("name1", "name2", "name3")
names(our_df)
```


```R
# Change the name of the second variable(only)
names(our_df)[2] <- "col2"
names(our_df)
```

## Indexing
#### Option 1. Index data frames just as you index matrices in R


```R
our_df[1,1]# grabs the value in the first row of the first variable.
```


```R
our_df[2,]# returns the second row of `our_df` (as a data frame).
```


```R
our_df[,3] #returns the third column (`name3`) of `our_df` (as a vector).
```

#### Option 2. Reference values/variables using columns' names.


```R
our_df$name1 #returns the column named `name1` (as a vector). .hi[Top:] `$`
```


```R
our_df[,"name1"] #returns the column named `name1` (as a vector).
```


```R
our_df["name1"] #returns the column named `x` (as a data frame).
```


```R
our_df[,c("name1","col2")] #returns a data frame with variables `"name1"` and `"col2"`
```

## Adding variables


```R
# Add a variable to our_df
our_df$new_var <- 6:10
```


```R
# Create interaction: multi = var1 * new_var
our_df$multi <- our_df$name1 * our_df$new_var
```


```R
our_df
```

# Dplyr


```R
# Pipe example
rnorm(10) %>% mean()
```

## Select


```R
storms %>% 
  select(name:hour, wind, pressure, -day)%>%
  head(8)
```


```R
storms %>% 
  select(name, contains("diameter"))%>%
  tail(5) #tail shows you the last x (here 5) rows of the dataframe
```


```R
storms %>%
  select(alias=name, year, month, day, hour,
         wind_speed=wind, ts_diameter=tropicalstorm_force_diameter)%>%
  head(4) 
```

## Filter


```R
storms %>% 
  select(name,year,month,day)%>%
  filter(year==2008,
         month >= 6)%>%
  head(3)
```


```R
storms %>% 
  select(name,year,month,day)%>%
  filter(year==2008|
         month >= 6)%>%
  head(3)
```


```R
storms %>% 
  select(name:status)%>%
  filter(grepl("tropical", status))%>%
  head(5) 
```


```R
storms %>% 
  select(name,year,month,day,hour,
         ts_diameter=tropicalstorm_force_diameter)%>%
  filter(is.na(ts_diameter))%>%
  head(1)
```

## Arrange


```R
storms %>% 
  select(name,year,month,day,hour,
         ts_diameter=tropicalstorm_force_diameter)%>%
  filter(!is.na(ts_diameter))%>%
  arrange(ts_diameter)%>%
  head(3)
```


```R
storms %>% 
  select(name,year,month,day,hour,
         ts_diameter=tropicalstorm_force_diameter)%>%
  filter(!is.na(ts_diameter))%>%
  arrange(desc(ts_diameter))%>%
  head(3)
```

## Mutate


```R
storms %>% 
  select(name, year, month) %>%
  mutate(decade = paste0(substr(year, start = 1, stop = 3),0),
         quarter = ifelse(month %in% c(12,1,2), "Winter",
                         ifelse(month %in% 3:5, "Spring",
                                ifelse(month %in% 6:8,"Summer","Automn"))),
         text = paste0(name, " occured in the", decade,"'s"))%>%
  head(4)
```


```R
storms %>% 
  select(name:status) %>% 
  mutate(status=as.character(status)) %>% 
  mutate_if(is.character, toupper) %>%
  head(3)
```


```R
storms %>% 
  select(name:status) %>% 
  mutate_at(vars(name,status), list(UP =~ toupper(.))) %>%
  head(3)
```

## Summarize and group_by


```R
storms %>% 
  select(name, year, wind, pressure) %>%
  mutate(decade = paste0(substr(year, start = 1, stop = 3),0))%>%
  group_by(decade) %>% 
  summarize(Nobs = n(),
            mean_wind = mean(wind, na.rm = TRUE),
            max_pressure = max(pressure, na.rm = TRUE))
```


```R
storms %>%
  summarize(mean_ts_diameter = mean(tropicalstorm_force_diameter))
storms %>% 
  summarize(mean_ts_diameter = mean(tropicalstorm_force_diameter, na.rm = TRUE))
```


```R
storms %>% 
  select(name, year, wind, pressure) %>%
  mutate(decade = paste0(substr(year, start = 1, stop = 3),0))%>%
  group_by(decade) %>% 
  summarize_if(is.numeric, mean, na.rm=T) %>%
  head(4)
```


```R
storms %>% 
  select(year, wind, pressure) %>%
  mutate(decade = paste0(substr(year, start = 1, stop = 3),0))%>%
  select(-year)%>%
  group_by(decade) %>% 
  summarize_all(list(Mean=~mean(.,na.rm=TRUE), 
                     Min=~min(.,na.rm=TRUE),
                     Max=~max(.,na.rm=TRUE)))
```
