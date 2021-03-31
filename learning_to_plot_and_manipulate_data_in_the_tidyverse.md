# Learning to plot and manipulate data in the tidyverse

**[Mark Stenglein](https://www.stengleinlab.org/)**
**March, 2021**

## Plotting with ggplot2

This exercise will introduce basic plotting skills using the [ggplot2](https://ggplot2.tidyverse.org/) in R.

In this exercise, we will import data into R and use ggplot2 functions to plot it different ways to answer questions about the data.

The data we will be analyzing represents historical weather data from Fort Collins, Colorado.

This data was downloaded from the US Government's [National Centers for Environmental Information](https://www.ncdc.noaa.gov/).  It contains montly average temperatures and precipitation data captured at a weather station in Fort Collins from 1893 - early 2021. 


#### Connect to the cctsi-103 server to use RStudio Server

You can use [RStudio Desktop](https://www.rstudio.com/products/rstudio/) as a stand alone app on your computer or you can use [RStudio Server](https://www.rstudio.com/products/rstudio/#rstudio-server) hosted on a server via a web interface.  For class today we will be using RStudio Server on the `cctsi-103.cvmbs.colostate.edu` server.  To connect to this server, you will need to either be on the CSU campus or connected to the CSU VPN using the Pulse Secure desktop vpn client.  

You could also do this exercise on your own computer but it'll be necessary to have [RStudio Desktop installed](https://www.rstudio.com/products/rstudio/download/#download) and in R the [tidyverse package](https://ggplot2.tidyverse.org/#installation).  

Open a browser and enter this address in the address bar: `http://cctsi-103.cvmbs.colostate.edu:8787/`

Enter the login credentials that I tell you to use.

If you are running this exercise as a stand-alone with RStudio Desktop, you will need [the weather data file](./FoCo_weather_data.csv) on your computer and can ignore the above instructions re: connecting to the cctsi-103 server.


#### Creating a new script file and initializing the R environment 

First, you'll need to create an R script in RStudioServer and initialize the R environment. 

To do this:

- select `File->New File->R Script`.  A new blank R script file should appear.
- save this file, by selecting `File->Save` and give the file a name something like `your_name_weather_script.R` (replace `your_name` with your name).
- make sure the R session is 'working' in the same directory as your script by selecting `Session->Set Working Directory->To Source File Location`.  Many problems experienced by folks new to R relate to the R session not having the same working directory as their data files.  [See this post for more information](https://www.stat.ubc.ca/~jenny/STAT545A/block01_basicsWorkspaceWorkingDirProject.html#working-directory)


#### Load the tidyverse libraries 

To use the [tidyverse](https://www.tidyverse.org/) functions, including those in ggplot2, you'll need to load the tidyverse libraries into your R environment.  To do this, type (or copy-paste) these lines at the top of your R script file and press `Cmd-Return` (in Mac OSX) or `Ctrl-Enter` (in Windows) to run them.  We'll need the [lubridate library](https://lubridate.tidyverse.org/) to work with date data.

```
library(tidyverse)
library(lubridate)
```

This should produce output like:
```
── Attaching packages ──────
✓ ggplot2 3.3.3     ✓ purrr   0.3.4
✓ tibble  3.1.0     ✓ dplyr   1.0.5
✓ tidyr   1.1.3     ✓ stringr 1.4.0
✓ readr   1.4.0     ✓ forcats 0.5.1
── Conflicts ───────────────
x dplyr::filter() masks stats::filter()
x dplyr::lag()    masks stats::lag()
```

The conflicts section here are just telling you that there are functions in the tidyverse (filter() and lag()) that mask (hide) functions with the same name from the stats package.  That's OK.  

Using the library() function loads a particular library into R.  If a library is not already installed on your computer, you will get an error message like `Error in library(tidyverse) : there is no package called ‘tidyverse’`.  In this case, you will need to install the library, using the `install.packages()` function.  For example: `install.packages("tidyverse")`. You only need to install packages once, but you need to load them every time you want to use them.


#### Importing the weather data

Now, you'll need to load (import) the historical weather data into R.  Importing data into R is daunting at first, but there are some of helpful functions for this, including:.

- [read.table()](https://www.rdocumentation.org/packages/utils/versions/3.6.2/topics/read.table) is a function to read comma-delimited (csv) tab-delimited (tsv) or any delimited plain text data file. The data downloaded from NOAA is in csv format.
- [read_excel()](https://www.rdocumentation.org/packages/readxl/versions/1.3.1/topics/read_excel) is a function in the [readxl package](https://readxl.tidyverse.org/) that will read data in from an Excel spreadsheet.  Being able to use data directly from Excel is a nice option.  You can write data to Excel using the [openxlsx package](https://cran.r-project.org/web/packages/openxlsx/index.html).

We will use read.table() to read in our weather data, which is in comma-delimited format (csv = comma-separated values).  Copy and paste the following lines into your R script and run them.  Note that lines in R beginning with `#` indicate comment lines, ignored by R:

```
# read in the data using read.delim a function to read (comma or tab) delimited data files
df <- read.table("FoCo_weather_data.csv", sep=",", header=T, stringsAsFactors = F)

# now, we need to tell R that the DATE column contains date data (when importing the data, it interpreted this column as character (text) data, but there are advantages to R recognizing it as actual dates).  
df$DATE <- as.Date(df$DATE)

# we will also create 2 new columns in our data that store the month and year of each observation.  The year function extracts the year from a date variable.
df$YEAR <- year(df$DATE)

# extract the month and make it a new column in our data frame (table)
# label=TRUE and abbr=TRUE, tells R to store the months as abbreviate month names instead of numbers ("Jan" instead of 1)
df$MONTH <- month(df$DATE, label=TRUE, abbr=TRUE)
```

The data is now stored in R in an object named `df` (short for data frame ~ data table).  Let's inspect this data frame by running this command:
```
View(df)
```

(You can also double click in the upper right Environment pane to make this happen).

What does this data frame look like?  What columns (variables) are present in the data?  How many observations are there?  How does this # of observations relate to the weather data we have (~128 years of monthly averages).

Importing data and then manipulating data like we have done here (aka "data wrangling") can be challenging.  Some people say that it represents 90% of the work of data science.  See [here for an introduction to this topic](https://r4ds.had.co.nz/data-import.html)


## Plotting the data

#### Has average temperature increased in Fort Collins over the past 128 years?

Let's begin by doing some exploratory plotting to answer the question of whether average temperatures have changed over the past century in Fort Collins.  Based on what you know about the earth's climate, what would you predict?

To create a plot in ggplot2, you use the `ggplot()` function.  This will create a plot.

So run this command to create a plot:

```
ggplot()
```

Nothing happened!!  Well actually something happened: a grey empty box appeared in the plots pane in the bottom right corner. That's because you've told R to create a plot but you told it nothing about the plot you want to create so it just created a blank plot. You haven't specified a data source, for instance.  Let's specify the data source.

Do this by typing and running:

```
ggplot(data=df) 
```

Now we've told R that we want to plot the data in the df dataframe, but we've still not told it anything about *how* we want to portray the data.  

To add additional layers to a plot in ggplot2, you simply use the `+` symbol to add a layer in the form of a [ggplot2 function](https://ggplot2.tidyverse.org/reference/index.html).   Let's start out by adding a simple scatter plot layer using the [geom_point](https://ggplot2.tidyverse.org/reference/geom_point.html) geometry.

```
ggplot(data=df) +
  geom_point(mapping = aes(x=DATE, y=TAVG)) 
```

The `+` symbol can be in the middle of a line between two ggplot2 functions or at the end of a line (doesn't work at the beginning of a line). 

Note that in the above example, we've specified an *aesthetic mapping* using the `mapping = aes()` construction.  "Aesthetic mappings describe how variables in the data are mapped to visual properties" in the plot. [See here for more information](https://ggplot2.tidyverse.org/reference/aes.html). 

In this case, we are defining the x and y positions of the data points as one of their visual properties.  We are telling ggplot to map the DATE variable to the x-axis and the TAVG (average monthly temperate) to the y axis.  DATE and TAVG are the names of two of the columns of data in our df data frame.

You should see a plot showing the 1518 data point in this data set plotted as individual points.

This is fine, but let's make it a little fancier.  Let's color the data points differently.  One way to do this is to specify a color for the points outside of the aes() aesthetic mapping:

```
ggplot(data=df) +
  geom_point(mapping = aes(x=DATE, y=TAVG), color="red") 
```

Now you should see a plot with all the points colored red.  But what if you want to color the points according to another property of the data (another variable).  You can do this by assigning a variable to the color in the aesthetic mappings:

```
ggplot(data=df) +
  geom_point(mapping = aes(x=DATE, y=TAVG, color=MONTH)) 
```

We have made the color aesthetic of each point to be a function of the MONTH variable.  The points should be colored according to each observation's month, with Dec and January having the coldest average temperatures, as expected.  When defining aesthetics it is possible to use categorical variables like Month, with 12 categories, or continuous variables, like temperature, with ~infinite possible values.

#### Faceting

Faceting splits the plot up into sub-plots based on the value of some variable in the data.  Faceting is a nice feature of ggplot2 that doesn't exist in Excel.  

For instance, we can divide the plot up according to the month of each obsevation.  To do this, we can use the [`facet_wrap()` function](https://ggplot2.tidyverse.org/reference/facet_wrap.html)

Enter and run the following lines into your R script:

```
ggplot(data=df) +
  geom_point(mapping = aes(x=DATE, y=TAVG, color=TAVG))  +
  facet_wrap(~MONTH)
```

Note that again we have added another feature to the plot using the `+` symbol.  Also note the tilde character `~` before the variable name MONTH.

You should now see 12 plots representing the average temperature for each month.  Now we've colored the points in each plot by the average temperature too. 


#### Labels and other theme elements

Note that the x and y axis labels are not great: the y axis is labeled TAVG, for instance, the name of the variable we have mapped to the y axis position.  There are a great many ways you can "decorate" your plot to make it more informative and better looking.  For instance, we can add x and y axis labels using the [xlab and ylab function](https://ggplot2.tidyverse.org/reference/labs.html)

```
ggplot(data=df) +
  geom_point(mapping = aes(x=DATE, y=TAVG, color=TAVG))  +
  facet_wrap(~MONTH) +
  xlab("") +
  ylab("Average monthly temperature in Fort Collins (F)")
```

We've left the x axis label blank because the year labels are self explanatory.

[See here](https://ggplot2.tidyverse.org/reference/theme.html) for more way to "to customize the non-data components of your plots: i.e. titles, labels, fonts, background, gridlines, and legends".  This page is so useful that I have it bookmarked in my browser menu bar.  There are also a number of [pre-defined complete themes](https://ggplot2.tidyverse.org/reference/ggtheme.html).


#### Back to the question: is average temperature increasing?

OK, let's return to the question of whether average temperature has increased over the past century in Fort Collins.  The scatter plots we've just generated seem to be sloping upwards, consistent with an increase in temperature, but the data is noisy: there are a lot of points and average temperature is pretty variable.  We can use the [geom_smooth](https://ggplot2.tidyverse.org/reference/geom_smooth.html) geometry to plot smoothed mean values of data.  This can aid "the eye in seeing patterns in the presence of overplotting".

Enter and run this in your script to create a version of the plot with a smoothed mean line:

```
ggplot(data=df) +
  geom_point(mapping = aes(x=DATE, y=TAVG))  +
  geom_smooth(mapping = aes(x=DATE, y=TAVG))  +
  xlab("") +
  ylab("Average monthly temperature in Fort Collins (F)")
```

It seems like the trendline created by geom_smooth has an upwards slope, but let's plot it without all the points to make it even clearer:

```
ggplot(data=df) +
  # geom_point(mapping = aes(x=DATE, y=TAVG))  +
  geom_smooth(mapping = aes(x=DATE, y=TAVG))  +
  xlab("") +
  ylab("Average monthly temperature in Fort Collins (F)")
```

Note that we just commented out the geom_point() line to avoid plotting the points in this instance.  

With the y axis focused on a smaller range, you can see that the average monthly temperature has increased in Fort Collins from ~46.5F in 1900 to ~51F today.  The gray shaded area represents the 95% confidence interval, indicating that there is some uncertainty in this estimate.


#### Let's ask a different question now: how warm is it expected to be in any month in Fort Collins?  

Let's now explore a new geometry: the [geom_boxplot](https://ggplot2.tidyverse.org/reference/geom_boxplot.html), which summarizes the distribution of the data or subsets of data.  The boxplot depicts the median value of a distribution, the 25th and 75th percentiles, and outliers (see [here](https://ggplot2.tidyverse.org/reference/geom_boxplot.html#summary-statistics) for more info).   

Let's use boxplots to display average daily minimum and maximum temperatures in each month.

```
ggplot(data=df) +
  geom_boxplot(aes(x=MONTH, y=TMAX), color="red") +
  geom_boxplot(aes(x=MONTH, y=TMIN), color="blue") +
  theme_bw() + 
  ylab("Monthly average min. and max. temperatures in Fort Collins (F)") +
  xlab("") 
```

What's the hottest month in Fort Collins?  What months have the most variable or least variable maximum and minimum temperatures?


##### [Time permitting] Exercise 1 

Create a plot that shows the average monthly precipitation in Fort Collins.

Our data frame contains a variable (column) named PRCP, which captures average monthly precipitation in inches.

Which is the rainiest month in Fort Collins?  How many inches does it rain on average in that month? 

##### [Time permitting] Exercise 2 

Use [geom_jitter](https://ggplot2.tidyverse.org/reference/geom_jitter.html) instead of geom_boxplot in the above code.  Jitter can be used to represent distributions of data points, especialy when they have identical values for one of their positional variables (all the x positions for the data points for each month would totally overlap if you used geom_point in our example).  

You could also try [geom_violin](https://ggplot2.tidyverse.org/reference/geom_violin.html) to make a violin plot, or change the [alpha aesthetic](https://ggplot2.tidyverse.org/reference/aes_colour_fill_alpha.html#alpha) to add transparency to be able to see overlapping points.



#### Saving a plot

Once you've created a plot you'll want to save it, of course, so you can use it in your presentation or paper.  One way to do this in RStudio is via the plot pane using the Export menu.  This is OK, but because it involves user interaction (mouse clicks), it's less reproducible than using code to save the plot.  A more reproducible way to save the plot is using the [ggsave function](https://ggplot2.tidyverse.org/reference/ggsave.html)

Run this command to save a PDF version of your plot:

```
ggsave("my_plot.pdf", width=4.5, height=3, units="in")
```

ggsave by default will save the most recently created plot.  You can save any plot by assigning your plot to a variable in R.  When you do this, RStudio will not automatically display the plot in the plot pane, but you can do that by just typing the name of that variable as a line of R code and running it.  For instance:

```
# create a plot and assign to a variable
monthly_precip_plot <- ggplot(data=df) +
  geom_boxplot(aes(x=MONTH, y=TMAX), color="red") +
  geom_boxplot(aes(x=MONTH, y=TMIN), color="blue") +
  theme_bw() + 
  ylab("Monthly average min. and max. temperatures in Fort Collins (F)") +
  xlab("") 

# display the plot in the plots pane
monthly_precip_plot

# save the plot to a file
ggsave("monthly_precipitation_plot.pdf", plot=monthly_precip_plot, width=4.5, height=3, units="in")
```

It is often possible to create a publication-quality plot using this approach.



## Additional information and exercises

The above was just a brief introduction to plotting in ggplot2 and to using the tidyverse in general.  Here are some additional vignettes of useful things you can do using these packages.


### Data manipulation using dplyr

The [dplyr library](https://dplyr.tidyverse.org/) in the tidyverse allows you to manipulate your data in a variety of useful ways: you can extract subsets of the data (rows or columns), summarize and calculate summary statistics of the data, join (merge) tables together, etc.  Here are some examples of using dplyr to do this:


#### Filter observations

The [filter function](https://dplyr.tidyverse.org/reference/filter.html) can be used to extract subsets of the rows of your data.  For instance, to only show the most recent years of weather data, you could run:

```
filter(df, YEAR > 2010)
```

You can assign this filtered data to a new data frame if you'd like:

```
last_ten_years_df <- filter(df, YEAR > 2010)
``` 

#### Sort observations

What was the hottest recorded month in Fort Collins?  Use the [arrange function](https://dplyr.tidyverse.org/reference/arrange.html) to answer this:

```
arrange(df, -TAVG)
```

#### Subset columns

You can keep only some of the columns in a data frame if you want using the [select function](https://dplyr.tidyverse.org/reference/select.html)

```
# I don't care about precipitation data!
just_temp_df <- select(df, -PRCP, -SNOW)

# show the first rows of this new df
head(just_temp_df)
```

#### Chaining together dplyr commands using the pipe

A powerful feature of tidyverse functions is that you can chain them together using a [pipe operator (`%>%`)](https://style.tidyverse.org/pipes.html)

For instance, to identify the hottest month in just the most recent decade, you could run:

```
filter(df, YEAR > 2010) %>% arrange(-TAVG)
```

#### Grouping and summarizing data

dplyr can be used to group and summarize data using the [group_by](https://dplyr.tidyverse.org/reference/group_by.html) and [summarize](https://dplyr.tidyverse.org/reference/summarise.html) functions.  

For instance, to collapse our historical weather data from monthly averages to yearly averages, we could run this code:

```
# could use medians instead too
# use na.rm = TRUE here because some months don't have data available (have NA values), which will make the mean value NA unless removed 
df_by_year <- df %>%
  group_by(YEAR) %>%
  summarize(yearly_TAVG = mean(TAVG, na.rm = TRUE),
            yearly_TMIN = mean(TMIN, na.rm = TRUE),
            yearly_TMAX = mean(TMAX, na.rm = TRUE))
```

The group_by(YEAR) function above is grouping the observations based on their year value.  So all 12 observations for each year will be grouped together, and the mean monthly average of those 12 months will be calculated and stored in a new variable, yearly_TAVG (and similarly for min and max temps).  Note that the summarize() function reduces the number of observations in the resulting data table to the number of groups. 

Then we could plot this: 

```
ggplot(data=df_by_year) +
  geom_point(aes(x=YEAR, y=yearly_TAVG), color="orange") +
  geom_point(aes(x=YEAR, y=yearly_TMIN), color="blue") +
  geom_point(aes(x=YEAR, y=yearly_TMAX), color="red") +
  geom_smooth(aes(x=YEAR, y=yearly_TAVG)) + 
  theme_classic() + 
  ylab("Average yearly temperature (F)") +
  xlab("")
```

This is actually a clearer way to visualize the temperature trends than when we plotted all the points above.

Note that the incomplete data for 2021 (only the early winter months are present for 2021 in this dataset) is pulling down the yearly averages for 2021.  We can use dplyr functions within our ggplot2 functions to omit 2021 data, as follows:

```
ggplot(data=filter(df_by_year, YEAR < 2021)) +
  geom_point(aes(x=YEAR, y=yearly_TAVG), color="orange") +
  geom_point(aes(x=YEAR, y=yearly_TMIN), color="blue") +
  geom_point(aes(x=YEAR, y=yearly_TMAX), color="red") +
  geom_smooth(aes(x=YEAR, y=yearly_TAVG)) + 
  theme_classic() + 
  ylab("Average yearly temperature (F)") +
  xlab("")
```

There, that looks better.

Note the use of filter() in the first line of the above code.  

#### Creating new variables using mutate

You can create new columns of data using the [mutate function](https://dplyr.tidyverse.org/reference/mutate.html) in dplyr

For example, say we wanted to calculate the spread between the minimum and maximum average temperature for each month.

```
df_with_a_new_column <- mutate(df, TSPREAD = TMAX - TMIN)
```

Or we could create a new column that contained the global average of yearly averages.  We could use this new column to answer the question of how the average temperature in each year compares to the global average of all the years of recorded data.  

```
df_by_year <- mutate(df_by_year, 
                     global_TAVG = mean(yearly_TAVG),
                     global_TMAX = mean(yearly_TMAX),
                     global_TMIN = mean(yearly_TMIN))
```

Here, we are using mean() to calculate mean values using the mutate function.  Because we have not grouped the data, this does not reduce the number of observations as happened above when using group_by() and summarize() to create df_by_year.  This is a subtle but important difference.

We are also over-writing the df_by_year dataframe with the output of the mutate function, which is a data frame with three new columns.  We could also give this df a new name. 

Let's plot the difference in yearly average temperature to the global average using the [geom_col geometry](https://ggplot2.tidyverse.org/reference/geom_bar.html).  Here, we will do the necessary calculation in the ggplot code:

```
ggplot(data=filter(df_by_year, YEAR < 2021)) +
  geom_col(mapping = aes(x=YEAR, y=(yearly_TAVG - global_TAVG), fill=(yearly_TAVG - global_TAVG)), color="black", size=0.25)+
  scale_fill_gradient2(low="blue", high="red") +
  theme_bw() +
  xlab("") +
  ylab("Yearly average temperature - overall average temperature in Fort Collins (F)")
``` 

You can see that years since 1950 have tended to have warmer than average temperatures and years before that cooler than average.  This is consistent with the trend data we plotted earlier, but is just another, possibly clearer, way to show the same thing.  

Note the calculations performed *within* the aes() function on the second line of code.  An alternative (and cleaner) way to have done this would be to have used mutate to create a new variable representing this calculated value and use that variable name in the aes function.

Note also the use of [scale_fill_gradient2 function](https://ggplot2.tidyverse.org/reference/scale_gradient.html).  This allowed us to color the bars in the plot according to how different the yearly average temperature was from the global average, with larger positive values being redder and larger negative values being blue.  For some geometries in ggplot, the color of the object is defined by the "fill" aesthetic and the color of the surrounding line is defined by the "color" aesthetic.  This can be a little confusing at first and is something to be aware of.

[More information about ggplot2 scales here](https://ggplot2.tidyverse.org/reference/index.html#section-scales)



### More information

This was just a brief introduction to the power of the tidyverse.  For more information, I would refer you to:

- [R for Data Science](https://r4ds.had.co.nz/): I found this book to be extremely helpful when I was learning about dplyr and ggplot2.  It was co-written by the creator of ggplot2, Hadley Wickham.  
- [Learning ggplot2 resources](https://ggplot2.tidyverse.org/#learning-ggplot2)
- [The on-line version of work-in-progress 3rd edition of “ggplot2: elegant graphics for data analysis”](https://ggplot2-book.org/).  Also written by Wickham.

