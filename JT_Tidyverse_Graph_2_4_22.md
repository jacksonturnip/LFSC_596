Managing Data and Creating a Graph in Tidyverse
================
Jackson Turner
2/4/2022

This R markdown file provides a brief tutorial on how to visualize some
data using tidyverse.

For this exercise, we’ll be using data from Kivlin et al. (2020) to
create a preliminary figure demonstrating how soil microbial communities
of several sites near and within the Great Smoky Mountains National Park
respond along a wildfire burn gradient (high, medium, and none) and
human development (natural v. urban).

## Loading in tidyverse and our data

Because we’ll be using ggplot2, dplyr, and tidyr to visualize our data,
we’ll go ahead and load the tidyverse package that contains all three.
Some of the functions will conflict with each other but it won’t affect
our analysis.

    ## -- Attaching packages --------------------------------------- tidyverse 1.3.1 --

    ## v ggplot2 3.3.5     v purrr   0.3.4
    ## v tibble  3.1.6     v dplyr   1.0.7
    ## v tidyr   1.1.4     v stringr 1.4.0
    ## v readr   2.1.1     v forcats 0.5.1

    ## -- Conflicts ------------------------------------------ tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

We’ll also go ahead and load in our data and save it as a data frame.

``` r
smokies_dat<-read.csv("Smokies_ENV.csv")
```

Let’s use tidyverse to get a sneak peek at our data frame. It’s worth
noting that each row in this output frame corresponds to a column in our
data frame.

``` r
glimpse(smokies_dat)
```

    ## Rows: 59
    ## Columns: 19
    ## $ ï..Sample    <chr> "BGB-AMPH-AMFb", "BGB-PAQU-AMFb", "FCM-LACT-AMFb", "FCM-P~
    ## $ Site         <chr> "BGB", "BGB", "FCM", "FCM", "FCM", "LCM", "LCM", "LGB", "~
    ## $ PlantSpecies <chr> "AMBR", "PAQU", "LACT", "PANI", "SMIL", "KALA", "LACT", "~
    ## $ PlantTaxa    <chr> "Amphicarpaea_bracteata", "Parthenocissus_quinquefolia", ~
    ## $ PlantFamily  <chr> "Fabaceae", "Vitaceae", "Asteraceae", "Poaceae", "Smilaca~
    ## $ Duration     <chr> "Annual", "Perennial", "Annual", "Annual", "Perennial", "~
    ## $ LifeForm     <chr> "Forb", "Vine", "Forb", "Grass", "Vine", "Tree", "Forb", ~
    ## $ Burn.Status  <chr> "None", "None", "High", "High", "High", "High", "High", "~
    ## $ Burn         <chr> "No", "No", "Yes", "Yes", "Yes", "Yes", "Yes", "Yes", "Ye~
    ## $ Burnbinomial <int> 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 1, 1, 0, 0, ~
    ## $ UN           <chr> "Natural", "Natural", "Natural", "Natural", "Natural", "N~
    ## $ Unbinomial   <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ~
    ## $ Latitude     <dbl> 35.68, 35.68, 35.70, 35.70, 35.70, 35.70, 35.70, 35.68, 3~
    ## $ Longitude    <dbl> -83.53, -83.53, -83.53, -83.53, -83.53, -83.53, -83.53, -~
    ## $ NMDS1        <dbl> 0.13, 0.12, -0.63, -0.66, -0.46, -0.55, -0.52, -0.20, 0.0~
    ## $ NMDS2        <dbl> -0.28, -0.19, -0.27, 0.16, -0.17, 0.50, 0.02, 0.09, 0.06,~
    ## $ Shannon      <dbl> 3.61, 3.67, 1.41, 2.28, 2.81, 2.03, 3.24, 3.17, 3.62, 2.7~
    ## $ Simpson      <dbl> 0.95, 0.96, 0.58, 0.84, 0.89, 0.83, 0.94, 0.94, 0.96, 0.8~
    ## $ Richness     <dbl> 87, 99, 22, 23, 41, 16, 54, 57, 76, 46, 48, 28, 21, 47, 2~

Oh no\! The first column of our data looks strange\!

The columns saved weirdly when this file was buildin Excel, so we’ll
make the first column our row names instead using the `row.names()`
function. Now that our row names are the same as our first column, we’ll
go ahead and delete that column to make our data concise.

``` r
row.names(smokies_dat)<-smokies_dat[,1]
smokies_dat<-smokies_dat[,-1]
```

How does our data look now?

``` r
glimpse(smokies_dat)
```

    ## Rows: 59
    ## Columns: 18
    ## $ Site         <chr> "BGB", "BGB", "FCM", "FCM", "FCM", "LCM", "LCM", "LGB", "~
    ## $ PlantSpecies <chr> "AMBR", "PAQU", "LACT", "PANI", "SMIL", "KALA", "LACT", "~
    ## $ PlantTaxa    <chr> "Amphicarpaea_bracteata", "Parthenocissus_quinquefolia", ~
    ## $ PlantFamily  <chr> "Fabaceae", "Vitaceae", "Asteraceae", "Poaceae", "Smilaca~
    ## $ Duration     <chr> "Annual", "Perennial", "Annual", "Annual", "Perennial", "~
    ## $ LifeForm     <chr> "Forb", "Vine", "Forb", "Grass", "Vine", "Tree", "Forb", ~
    ## $ Burn.Status  <chr> "None", "None", "High", "High", "High", "High", "High", "~
    ## $ Burn         <chr> "No", "No", "Yes", "Yes", "Yes", "Yes", "Yes", "Yes", "Ye~
    ## $ Burnbinomial <int> 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 1, 1, 0, 0, ~
    ## $ UN           <chr> "Natural", "Natural", "Natural", "Natural", "Natural", "N~
    ## $ Unbinomial   <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ~
    ## $ Latitude     <dbl> 35.68, 35.68, 35.70, 35.70, 35.70, 35.70, 35.70, 35.68, 3~
    ## $ Longitude    <dbl> -83.53, -83.53, -83.53, -83.53, -83.53, -83.53, -83.53, -~
    ## $ NMDS1        <dbl> 0.13, 0.12, -0.63, -0.66, -0.46, -0.55, -0.52, -0.20, 0.0~
    ## $ NMDS2        <dbl> -0.28, -0.19, -0.27, 0.16, -0.17, 0.50, 0.02, 0.09, 0.06,~
    ## $ Shannon      <dbl> 3.61, 3.67, 1.41, 2.28, 2.81, 2.03, 3.24, 3.17, 3.62, 2.7~
    ## $ Simpson      <dbl> 0.95, 0.96, 0.58, 0.84, 0.89, 0.83, 0.94, 0.94, 0.96, 0.8~
    ## $ Richness     <dbl> 87, 99, 22, 23, 41, 16, 54, 57, 76, 46, 48, 28, 21, 47, 2~

There, our data are now ready to explore\!

## Subsetting our data

Oh no\! The botanist who’s quality checking our samples just called, and
they said that the violets we sampled were actually cardboard cuttouts\!

We can’t include them in the data anymore since there was no plant in
those samples to affect microbial diversity. Therefore, we’ll have to
remove each row of our data set that corresponds to a violet (with the
`PlantSpecies` code `VIOL`). Tidyverse makes this easy by allowing us to
filter out all the cardboard cuttouts we mistook for violets in our
data.

``` r
(smokies_dat<-smokies_dat %>%
  filter(PlantSpecies!="VIOL"))
```

Now that we’ve fixed this mistake, let’s go ahead and remove any empty
or duplicated rows of data.

``` r
smokies_dat %>%
  na.omit() %>%
  distinct()
```

It looks like our data is in order. To make this analysis easy on
ourselves, we’ll go ahead and subset it, creating a smaller data frame
with only the information we need to answer our question: `UN`
(development), `Burn.Status` (burn status), and `Shannon` (Shannon
diversity).

``` r
smokies_dat_subset<- smokies_dat %>%
  select(UN,Burn.Status,Shannon)
```

Now that we’ve checked and subset our data, we’re ready to visualize our
relevant data\!

## visualizing our data

The `ggplot2` package within `tidyverse` allows us to plot our data in
any way of our choosing. For this particular graph, we’d like to use a
boxplot with human development on the X-axis, Shannon diversity on the
Y-axis, and a seperate boxplot for each burn status.

``` r
ggplot(smokies_dat_subset,aes(x=UN,y=Shannon,fill=Burn.Status)) +
  geom_boxplot() +
  xlab("Development Status") +
  ylab("Shannon Divsersity") +
  labs(fill="Burn Status")+
  theme(panel.border = element_rect(linetype = "solid", fill = NA),
        panel.background = element_blank(),
        axis.title = element_text(face = "plain", color = "black", size = 18),
        axis.text.y = element_text(face = "plain", color = "black", size = 10),
        axis.text.x = element_text(face = "plain", color = "black", size = 10),
        axis.line.y = element_line(size = 0.5),
        axis.line.x = element_line(size = 0.5),
        legend.title=element_text(size=16),
        legend.text=element_text(size=20))
```

![](JT_Tidyverse_Graph_2_4_22_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

From this exploratory graph we can infer that natural, unburned sites in
this system are somewhat more diverse than those of urban sites or those
that have been affected by wildfires. Despite the inferences provided by
this graph, this is just an exploratory graph and is under no
circumstances a substitute for actual analyses used to answer our
question.

This concludes the tutorial – I hope it was helpful and I wish you the
best in your journey creating graphs\!

References:

  - Stephanie N. Kivlin, V. Rosanne Harpe, Jackson H. Turner, Jessica A.
    M. Moore, Leigh C. Moorhead, Kendall K. Beals, Mali M. Hubert,
    Monica Papeş, Jennifer A. Schweitzer; Arbuscular mycorrhizal fungal
    response to fire and urbanization in the Great Smoky Mountains
    National Park. Elementa: Science of the Anthropocene 21 January
    2021; 9 (1): 00037. doi:
    <https://doi.org/10.1525/elementa.2021.00037>
