For coding club tutorial
================
Giada Leone
04/12/2021

------------------------------------------------------------------------

#### This tutorial is aimed at people who have some experience in using R and would like to try **spatial data visualisation** by using R as a GIS like software using specific packages. <br> <br> R is very powerful due to its wide applications in statistics, modelling, data visualisation and spatial data. In this tutorial we will be focussing on a specific type of spatial data: *point data*, and different ways to visualise it without diving into the theory behind more complex spatial objects such as line polygon and raster data. If you are interested in other types of spatial data have a look at the supplementary material at the end of the tutorial.

------------------------------------------------------------------------

### Tutorial Aims

##### 1. Data preparation and wrangling for spatial visualisation

##### 2. Creating basic *static* maps with point data using ggplot2

##### 3. Creating *interactive* maps using the leaflet package

##### 4. Challenge yourself try it on another dataset

------------------------------------------------------------------------

#### Here is a quick introduction to spatial data

[![IMAGE ALT TEXT
HERE](https://img.youtube.com/vi/F7U-nJj9yUg/0.jpg)](https://www.youtube.com/watch?v=F7U-nJj9yUg)

------------------------------------------------------------------------

#### Let’s set up your project folder/ directory

Once you open Rstudio you need to create a project to include all of the
files related to this tutorial by clicking on the create a project
symbol ![](../other/Rproj_screenshot.png) in the global toolbar at the
top of an Rstudio window. Click on ‘new directory’ and ‘new project’,
save the folder on your desktop and give it a descriptive name such as
‘maps\_tutorial’.

Your Rstudio project will open up and now you can create a new script by
typing ‘ctrl + shift + N’ on windows or ‘command + shift + N’ on Mac or
simply by using the global toolbar.

------------------------------------------------------------------------

#### Downloading the files

To download the necessary dataset you can clone or download the
[repository](https://github.com/EdDataScienceEES/tutorial-giadaleone99)
by clicking on the green ‘Code’ button and then clicking on ‘download
ZIP’. Next you can unzip the folder and move the data file
‘beachwatch\_data.csv’ into the project folder/directory that you
created previously. This data is provided by the [Marine Conservation
Society](https://www.mcsuk.org/) as part of the Great British Beach
Clean initiative.

------------------------------------------------------------------------

#### Required packages

We will be using the `tidyverse` to do data wrangling and preparing the
dataset, it also includes the `ggplot2` package which we will use for
visualisation of point data.

The `janitor` package is a useful tool to clean datasets which will come
in handy when handling public datasets which may have unorganised column
names.

The `leaflet` package allows us to create interactive and informative
maps.

If you have never used these packages before you will need to install
them by using the `install.packages("packagename")` function and include
the names within the "" before loading them with the
`library(packagename)` function.

``` r
# Libraries ----
# install.packages("tidyverse")

library(tidyverse)  # Data wrangling & includes ggplot2
library(janitor)    # Cleaning datasets
library(ggthemes)   # Themes for ggplot2
library(leaflet)    # Interactive map
```

------------------------------------------------------------------------

### Importing the dataset

Here we are importing the file ‘beachwatch\_data.csv’ which you
previously moved to your directory. We use the `read.csv()` function and
assign the dataframe to object called ‘beaches’.

Note that if you previously saved the data file in a subfolder you will
have to specify the filepath to import the data. For example, if you
moved the csv file to a folder names ‘data’ within the repo, you should
write `beaches <- read.csv(data/beachwatch_data.csv")`

``` r
beaches <- read.csv("beachwatch_data.csv")
```

------------------------------------------------------------------------

### Preparing the data

Here we are organising the data by firstly cleaning the column names
with the clean\_names() function from the janitor package. Next we are
deleting the unnecessary columns from the dataframe as well as dropping
columns which have NA values.

``` r
# cleaning data frame column names
beaches <- beaches %>% clean_names()

# deleting unused columns
beaches <- select(beaches,-seq(12, 109), -seq(112, 170))

# dropping columns which have NA values
beaches <- drop_na(beaches)
```

Changing data types: if variables are characters we are converting them
to factors or ‘categorical’ data types, and if variables are integers we
convert them to numeric. This is important for subsequent steps to work.
R often requires variables to be a specific data type.

``` r
beaches <- beaches %>% mutate_if(is.character, as.factor) %>%
                       mutate_if(is.integer, as.numeric)
```

------------------------------------------------------------------------

### Creating a static map with ggplot2

Let’s use ggplot to create a map to show us all the beaches around the
UK which have data from Great British Beach Clean surveys.

We will gradually make this map step by step so you can follow the map
making process.

``` r
# Basic world map
ggplot() +                                # Calling ggplot()
   borders("world", colour = "black") +   # Plotting world borders in black
   theme_map()                            # map theme (no long & lat, white background )
```

![](rmdtomdfile_files/figure-gfm/unnamed-chunk-6-1.png)<!-- --> <br>
Next, we will zoom in to the UK by setting the x and y limits within the
`coord_cartesian()` function.

``` r
# Zoom in to UK
ggplot() +
   borders("world", colour = "black") +
   coord_cartesian(xlim = c(-10, 3), ylim = c(50.3, 59)) +  # specifying coordinates to zoom
   theme_map()
```

![](rmdtomdfile_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

<br>

Now that we have zoomed into our area of interest, we can plot the
points of the UK beaches surveyed in the dataset by specifying the
coordinates (longitude and latitude) of our points within the `aes()`
function.

We also filled the land with a lightgrey colour to make it stand out
more and coloured the background blue.

``` r
# Adding spatial data points (beaches)
ggplot(beaches, aes(x = beach_longitude,
                    y = beach_latitude)) +
   borders("world", colour = "black", fill = "lightgrey") +  # changing the colour of the map
   coord_cartesian(xlim = c(-10, 3), ylim = c(50.3, 59)) +
   geom_point(size = 1) +
   theme_map() +
   theme(panel.background = element_rect(fill = "aliceblue"))
```

![](rmdtomdfile_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

In the code below we are telling ggplot to colour the points depending
on beach region for which we have a column in the dataset.

``` r
# Colouring points based on beach region
ggplot(beaches, aes(x = beach_longitude,
                    y = beach_latitude,
                    colour = beach_region)) +                  # Colouring by beach region
   borders("world", colour = "black", fill = "lightgrey") +
   coord_cartesian(xlim = c(-10, 3), ylim = c(50.3, 59)) +
   geom_point(size = 1) +
   theme_map() +
   theme(panel.background = element_rect(fill = "aliceblue"))  # Changing background colour
```

![](rmdtomdfile_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

We can see that the default location of the legend overlaps with the
map, so we move the legend by specifying the `legend.position()` withing
the theme function. Here we also add an informative title and change the
legend title and box.

``` r
ggplot(beaches, aes(x = beach_longitude,
                    y = beach_latitude,
                    colour = beach_region)) +
   borders("world", colour = "black", fill = "lightgrey") +
   coord_cartesian(xlim = c(-10, 3), ylim = c(50.3, 59)) +
   geom_point(size = 1) +
   theme_map() +
   theme(plot.title = element_text(size = 12, hjust = 0.5, face = 'bold'),  # title characteristics
         panel.background = element_rect(fill = "aliceblue"),
         legend.position = c(0.77, 0.45),
         legend.box.background = element_rect(color = 'grey', size = 0.5)) +  # Adding a border
   labs(colour = "UK Regions",  # Changing legend title
        title = "Great British Beach Clean surveys in the UK")  # Informative title
```

![](rmdtomdfile_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

That looks much better than it did before! Now, you have been equipped
with some of the basic tools to create static maps with point data in
ggplot.

While maps like this can be very useful for reports, interactive maps
offer the ability to make maps much more informative by including
specifications for each of the points!
