---
title: "Expenses with tidyverse"
output: html_notebook
---
# Overview
The goal of tidyr is to help you create tidy data. Tidy data is data where:

1. Every column is variable.
2. Every row is an observation.
3. Every cell is a single value.

Tidy data describes a standard way of storing data that is used wherever 
possible throughout the tidyverse. If you ensure that your data is tidy,
you’ll spend less time fighting with the tools and more time working on your 
analysis. Learn more about tidy data in vignette("tidy-data").


# Installing and opening packages

```{r}
#install.packages("tidyverse")
#install.packages("ggplot2")
library(ggplot2)
library(tidyverse)
```
# Navigating and setting working directory

```{r}
getwd()
setwd("/Users/madeo/OneDrive/Cloud/Learn programming/R/")
```


# Opening files

```{r}
df <- read.csv("df.csv")
```

# Dates in the dataframe

In this file there is a column that registers dates. The function  **class()** 
will check  variable's nature. 
```{r}
class(df$Date)
```
It is possible to manipulate the format of the dates using the function
**as.Date()** and giving the proper argument. From *R Cookbook - chapter 7.9*:

The format argument defines the appearance of the resulting string. Normal 
characters, such as a slash (/) or hyphen (-), are simply copied to the output 
string. Each two-letter combination of a percent sign (%) followed by another 
character has special meaning. Some common ones are:

* %b
+ Abbreviated month name (“Jan”)
* %B
+ Full month name (“January”)
* %d
+ Day as a two-digit number
* %m
+ Month as a two-digit number
* %y
+ Year without century (00–99)
* %Y
+ Year with century

See the help page for the strftime function for a complete list of 
formatting codes.

To create a column with a day and month with two digit numbers and year with
century:

```{r}
date_col <- as.Date(df$Date, "%d/%m/%Y")
#class(date_col)
```

# Create, modify, and delete columns
 
To **remove** an entire column from a data frame:

```{r}
df$Date<-NULL
```

To **append** a column stored in a variable:

```{r}
df$Date<-date_col
```

The function [mutate.()][https://dplyr.tidyverse.org/reference/mutate.html] 
will create/add variables preserving the old one. However, **transmutate.()**
will do the same but deleting the old variable.

In this example will create new columns containg the months and years separately. 


```{r}
df<-df %>%  mutate(month=format(Date,"%B"), year=format(Date,"%Y"))
```

Our objective now is to calculate **FOR EACH MONTH** the total amount earned and 
spent. To do so we will use the function summarise to do the calculations and 
store them in a new dataframe that will be grouped by months using the function 
group_by. [summarise()](https://dplyr.tidyverse.org/reference/summarise.html) 
creates a new data frame. It will have one (or more) rows for each combination 
of grouping variables; if there are no grouping variables, the output will have 
a single row summarising all observations in the input. It will contain one 
column for each grouping variable and one column for each of the summary 
statistics that you have specified. Most data operations are done on groups
defined by variables. [group_by()](https://dplyr.tidyverse.org/reference/group_by.html?q=groupby) takes an existing tbl and converts it into a 
grouped tbl where operations are performed "by group". **ungroup()** removes 
grouping.

```{r}
#2020
Total_in_out_2020 <- df %>%  group_by(month) %>% summarise(total_out=sum(Amount[InOut=="Out" & year=="2020"]),
                                                            total_in=sum(Amount[InOut=="In" & year=="2020"]))

#2021
Total_in_out_2021 <- df %>%  group_by(month) %>% summarise(total_out=sum(Amount[InOut=="Out" & year=="2021"]),
                                                           total_in=sum(Amount[InOut=="In" & year=="2021"]))
```

The months in this new data frame are not in order by month.To order them we 
will use the function [slice()](https://dplyr.tidyverse.org/reference/slice.html) to index the rows by their location and the match
function to reorder them base on a vector given:

```{r}
month_order<-c("enero","febrero","marzo","abril","mayo","junio","julio","agosto","sepriembre","octubre",
               "noviembre","diciembre")

Total_in_out_2020 <- Total_in_out_2020 %>% slice(match(month_order,month))

Total_in_out_2021 <- Total_in_out_2021 %>% slice(match(month_order,month))
```

Now we will create an additional column named as savings in which we substract 
what we have spent to of what we have earnt:
```{r}
Total_in_out_2020 <- Total_in_out_2020 %>%  mutate(savings=total_in-total_out)

Total_in_out_2021 <- Total_in_out_2021 %>%  mutate(savings=total_in-total_out)
sum(Total_in_out_2020$savings)
sum(Total_in_out_2021$savings)
```



