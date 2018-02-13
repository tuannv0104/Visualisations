Untitled
================
FC\_rSTATS
13/02/2018

Scatter Plots & Crosshairs with ggPlot2
=======================================

Across R’s many visualisation libraries, you will find several ways to create scatter plots. ggPlot2, being one of the fundamental visualisation libraries, offers perhaps the simplest way to do so. In a few lines, we will be able to create scatter plots that show the relationship between two variables. It also offers easy ways to customise these charts, through adding crosshairs, text, colour and more.

This article will plot goals for and against from a season, taking you through the initial creation of the chart, then some customisation that ggplot offers. Import the packages and off we go.

``` r
require(rvest)
```

    ## Loading required package: rvest

    ## Loading required package: xml2

``` r
require(ggplot2)
```

    ## Loading required package: ggplot2

The Data
--------

Using the web scrapping lessons that we learn in a previous tutorial we can grab live data for the Premier League season. I won't go into details here but running the code below will grab the data we need.

``` r
page <- "https://uk.soccerway.com/national/england/premier-league/20172018/regular-season/r41547/?ICID=SN_01_01"
scraped_page <- read_html(page)
Teams  <- scraped_page %>% html_nodes("#page_competition_1_block_competition_tables_6_block_competition_league_table_1_table .large-link") %>% html_text() %>% as.character()
Teams <- gsub("\n","",Teams)
Teams <- trimws(Teams)
GoalsFor <- scraped_page %>% html_nodes(".team_rank .total_gf") %>% html_text() %>% as.numeric()
GoalsAgainst <- scraped_page %>% html_nodes(".team_rank .total_ga") %>% html_text() %>% as.numeric()
df <- data.frame(Teams, GoalsFor, GoalsAgainst)
```

Plotting with ggplot2
---------------------

ggplot2 is a very powerful plotting package in R which follows a 'grammar' of graphics. It's my go to solution when wanting to plot anything in R. It's quick to get working. Firstly, let's plot a basic version of a scatter plot.

``` r
ggplot(data = df, aes(x=GoalsFor, y=GoalsAgainst)) + 
geom_point() 
```

![](Untitled_files/figure-markdown_github/unnamed-chunk-3-1.png)

![Plot Version 1](/Users/Joe/plot1.png)

Great, it's that easy. But let's build on this base plot. Let's add the crosshairs, which are essentially the mean of each axis. This allows us to say anything above the line is 'above average' and anything below the line is 'below average'.

``` r
ggplot(data = df, aes(x=GoalsFor, y=GoalsAgainst)) + 
geom_point() +
geom_vline(xintercept=mean(GoalsFor)) +
geom_hline(yintercept=mean(GoalsAgainst))
```

![](Untitled_files/figure-markdown_github/unnamed-chunk-4-1.png)

![Plot Version 2](/Users/Joe/plot2.png)

That work's but the lines are a little brash and heavy. Let's make the line a dashed line, change the colour to red and make it a little see-through.

``` r
ggplot(data = df, aes(x=GoalsFor, y=GoalsAgainst)) + 
geom_point() +
geom_vline(xintercept=mean(GoalsFor), linetype="dashed", alpha = 0.4, colour = "red") +
geom_hline(yintercept=mean(GoalsAgainst), linetype="dashed", alpha = 0.4, colour = "red")
```

![](Untitled_files/figure-markdown_github/unnamed-chunk-5-1.png)

![Plot Version 3](/Users/Joe/plot3.png)

This now works pretty well, although it's hard to know which team is which! So we let's add some labels.

``` r
ggplot(data = df, aes(x=GoalsFor, y=GoalsAgainst, label = Teams)) + 
geom_point() +
geom_vline(xintercept=mean(GoalsFor), linetype="dashed", alpha = 0.4, colour = "red") +
geom_hline(yintercept=mean(GoalsAgainst), linetype="dashed", alpha = 0.4, colour = "red") +
geom_text(size = 2, nudge_y = -0.5)
```

![](Untitled_files/figure-markdown_github/unnamed-chunk-6-1.png)

![Plot Version 4](/Users/Joe/plot4.png)

Now we want to create two labels to go in the top left and the bottom right corners to help the reader understand the plot better. To do this we create a new dataframe for the annotations and use that to position them in the corner.

``` r
annotations <- data.frame(
   xpos = c(-Inf,Inf),
   ypos =  c(Inf,-Inf),
   annotateText = c("Poor Attack, Poor Defense","Strong Attack, Strong Defense"),
   hjustvar = c(0,1) ,
   vjustvar = c(1,0))

ggplot(data = df, aes(x=GoalsFor, y=GoalsAgainst, label = Teams)) + 
  geom_point() + 
  geom_vline(xintercept=mean(GoalsFor), linetype="dashed", alpha = 0.4, colour = "red") +
  geom_hline(yintercept=mean(GoalsAgainst), linetype="dashed", alpha = 0.4, colour = "red") +
  geom_label(data = annotations, aes(x=xpos,y=ypos,hjust=hjustvar, vjust=vjustvar,label=annotateText, colour = "red")) +
  geom_text(size = 2.5, nudge_y = -0.5)
```

![](Untitled_files/figure-markdown_github/unnamed-chunk-7-1.png)

![Plot Version 5](/Users/Joe/plot5.png)

This works pretty well however, it covers up the Stoke City data point. So we change the ordering so the data point is on top of the annotations. We can also remove the legend and add a title.

``` r
annotations <- data.frame(
   xpos = c(-Inf,Inf),
   ypos =  c(Inf,-Inf),
   annotateText = c("Poor Attack, Poor Defense","Strong Attack, Strong Defense"),
   hjustvar = c(0,1) ,
   vjustvar = c(1,0))

ggplot(data = df, aes(x=GoalsFor, y=GoalsAgainst, label = Teams)) + 
  geom_vline(xintercept=mean(GoalsFor), linetype="dashed", alpha = 0.4, colour = "red") +
  geom_hline(yintercept=mean(GoalsAgainst), linetype="dashed", alpha = 0.4, colour = "red") +
  geom_label(data = annotations, aes(x=xpos,y=ypos,hjust=hjustvar, vjust=vjustvar,label=annotateText, colour = "red", size = 1)) +
  geom_text(size = 2.5, nudge_y = -0.5) +
  geom_point() +
  theme(legend.position="none") +
  ggtitle("Goals For & Against : Premier League 2017/18")
```

![](Untitled_files/figure-markdown_github/unnamed-chunk-8-1.png)

![Plot Version 6](/Users/Joe/plot6.png)

Have a play around with the settings of the plots to discover how it works and make it your own.

The main thing I would improve here is that some team labels overlap, we could use the package ggrepel to help us better distribute the team labels.

``` r
require(ggrepel)
```

    ## Loading required package: ggrepel

``` r
ggplot(data = df, aes(x=GoalsFor, y=GoalsAgainst)) + 
  geom_vline(xintercept=mean(GoalsFor), linetype="dashed", alpha = 0.4, colour = "red") +
  geom_hline(yintercept=mean(GoalsAgainst), linetype="dashed", alpha = 0.4, colour = "red") +
  geom_label(data = annotations, aes(x=xpos,y=ypos,hjust=hjustvar, vjust=vjustvar,label=annotateText, colour = "red", size = 1)) +
  geom_text_repel(aes(GoalsFor, GoalsAgainst, label = Teams)) +
  geom_point() +
  theme(legend.position="none") +
  ggtitle("Goals For & Against : Premier League 2017/18")
```

![](Untitled_files/figure-markdown_github/unnamed-chunk-9-1.png)

![Plot Final Version](/Users/Joe/plot7.png)

Final Code
----------

``` r
require(rvest)
require(ggplot2)
require(ggrepel)

page <- "https://uk.soccerway.com/national/england/premier-league/20172018/regular-season/r41547/?ICID=SN_01_01"
scraped_page <- read_html(page)
Teams  <- scraped_page %>% html_nodes("#page_competition_1_block_competition_tables_6_block_competition_league_table_1_table .large-link") %>% html_text() %>% as.character()
Teams <- gsub("\n","",Teams)
Teams <- trimws(Teams)
GoalsFor <- scraped_page %>% html_nodes(".team_rank .total_gf") %>% html_text() %>% as.numeric()
GoalsAgainst <- scraped_page %>% html_nodes(".team_rank .total_ga") %>% html_text() %>% as.numeric()
df <- data.frame(Teams, GoalsFor, GoalsAgainst)

annotations <- data.frame(
   xpos = c(-Inf,Inf),
   ypos =  c(Inf,-Inf),
   annotateText = c("Poor Attack, Poor Defense","Strong Attack, Strong Defense"),
   hjustvar = c(0,1) ,
   vjustvar = c(1,0))

ggplot(data = df, aes(x=GoalsFor, y=GoalsAgainst)) + 
  geom_vline(xintercept=mean(GoalsFor), linetype="dashed", alpha = 0.4, colour = "red") +
  geom_hline(yintercept=mean(GoalsAgainst), linetype="dashed", alpha = 0.4, colour = "red") +
  geom_label(data = annotations, aes(x=xpos,y=ypos,hjust=hjustvar, vjust=vjustvar,label=annotateText, colour = "red", size = 1)) +
  geom_text_repel(aes(GoalsFor, GoalsAgainst, label = Teams)) +
  geom_point() +
  theme(legend.position="none") +
  ggtitle("Goals For & Against : Premier League 2017/18")
```

![](Untitled_files/figure-markdown_github/unnamed-chunk-10-1.png)