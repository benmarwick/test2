Persistence of Public Interest in Gun Control After US Mass Shooting Events: Is the Stoneman Douglas High School Event Different?
================
Ben Marwick
25 February, 2018

Introduction
------------

Reflecting on the mass shooting event at the Stoneman Douglas High School in Florida in Feb 2018, I thought I'd take a quick look over the weekend to see how it compares to previous events in the US. In particular, I want to see if we can detect any differences in the public reaction to this event, compared to previous events. Part of the motivation comes from a story on *Vox* ['Teenagers are doing the impossible: keeping America's attention on guns'](https://www.vox.com/policy-and-politics/2018/2/21/17033308/florida-shooting-media-gun-control) that claims we seeing a sustained reaction to a mass shooting that we haven't seen before. A similar story appeared on [*Mashable*](https://mashable.com/2018/02/19/parkland-students-gun-control-debate-google-trends/#fAY2Ogo8Jkqb). I was also inspired by tweets from [Nate Silver](https://twitter.com/NateSilver538/status/965352547383959552) and [Alexander Agadjanian](https://twitter.com/A_agadjanian/status/965362281105231873) that included screenshots from Google Trends queries to support the same claim made by the Vox article.

Here we'll look at three sources of data to investigate public reactions:

-   Google search volume for 'gun control' in the time before and after the events
-   page views of Wikipedia articles relating to gun control and mass shootings in the time before and after the events
-   TV news mentions of gun control in the time before and after the events

One key question is: which events have generated sustained public interest in gun control, and has the Feb 2018 event generated more interest than previous events?

Reproducibility of this analysis
--------------------------------

This analysis is conducted entirely in R, and this document contains all the code used to collect, analyse and visualise the data. The analysis is reproducible and so it will be easy to update to see how the Stoneman Douglas High School event compares with previous events as time passes. The code and data are openly available at <https://github.com/benmarwick/test2>. In a month we can run it again to see how public interest has changed as we move further in time away from the Stoneman Douglas High School. Should another event occur in the future, we can add that event to this comparison to investigate public reactions.

The repository includes configuration files to open and run this document in RStudio in your web browser, making the code immediately reproducible by anyone, anywhere, without installing anything on your computer. We use the [Binder](https://mybinder.org/) project to create a sharable, reproducible, executible environment for this document. The `runtime.txt` file specifies the date of [MRAN](https://mran.microsoft.com/documents/rro/reproducibility) snapshot that we download the packages from, and the `install.R` installs these packages that our code in this document depends on.

Click on this button to open RStudio and interact with this document to reproduce the analysis in your browser: [![Binder](http://mybinder.org/badge.svg)](http://beta.mybinder.org/v2/gh/benmarwick/test2/master?urlpath=rstudio)

After you click on the button, Binder will take a few minutes to load the packages we depend on. Once you see RStudio in your browser like the figure below, click on README.Rmd in the lower right pane to open this file and interact with it:

![](figures/rstudio.png)

There are three R packages contributed by the community of R users that are central to this project because they provide easy access to data sources:

-   **[gtrendsR](https://github.com/PMassicotte/gtrendsR)** by Philippe Massicotte and Dirk Eddelbuettel to perform and display [Google Trends](https://trends.google.com/trends/) quries.
-   **[pageviews](https://github.com/Ironholds/pageviews/)** by Oliver Keyes search and download [article pageviews of Wikipedia](https://wikitech.wikimedia.org/wiki/Analytics/AQS/Pageviews) and its sister projects
-   **[newsflash](https://github.com/hrbrmstr/newsflash)** by Bob Rudis for tools to work with the [Internet Archive and GDELT Television Explorer](https://blog.gdeltproject.org/the-new-television-explorer-launches/). In fact this package did not work for me, but it provided a lot of useful information and was essential for getting me started with getting data on broadcast television news coverage.

To clean, tidy and manipulate the data obtained by these pacakges I used the [tidyverse](https://www.tidyverse.org/).

Note that many of the plots here are interactive - you can hover over the data points and lines to get more information. You can zoom in and out, and you can select and deselect events in the legend to make comparisons easier. These plots were made with the [ggplot2](http://ggplot2.tidyverse.org/) and [plotly](https://plot.ly/r/) packages. If you don't see the interactive plots below, [click here](https://cdn.rawgit.com/benmarwick/test2/97119b4c/README.html).

Data Source for Mass Shooting Events in the US
----------------------------------------------

A mass shooting is defined as an indiscriminate rampage in public places resulting in three or more victims killed by the attacker. We've obtained data from the *Mother Jones* article [US Mass Shootings, 1982-2018: Data From Mother Jones' Investigation](https://www.motherjones.com/politics/2012/12/mass-shootings-mother-jones-full-data/). They provide a Google spreadsheet with a CSV download link. This appears to be reputable, and is used by the *Washington Post* in their report '[The terrible numbers that grow with each mass shooting](https://www.washingtonpost.com/graphics/2018/national/mass-shootings-in-america/?utm_term=.d2b76eede4da)'. Other sources that could be useful, but I have not used here, include the [Mass Shooting Tracker](https://www.massshootingtracker.org/data/all) and <http://www.gunviolencearchive.org/mass-shooting/>.

``` r
# We just get it once from the web and save locally
# us_mass_shootings <- 
#  readr::read_csv("https://docs.google.com/spreadsheet/pub?key=0AswaDV9q95oZdG5fVGJTS25GQXhSTDFpZXE0RHhUdkE&output=csv")

# format the date column as a date class
library(tidyverse)
library(lubridate)
library(glue)
# us_mass_shootings$Date <- mdy(us_mass_shootings$Date)

# write_csv(us_mass_shootings, "us_mass_shootings.csv")
us_mass_shootings <- read_csv("data/us_mass_shootings.csv")

# tidy a few variables 
us_mass_shootings <- 
  us_mass_shootings %>% 
  mutate(Venue = str_trim(Venue)) %>% 
  mutate(Case = glue('{Case} ({year(Date)})')) 
```

In the *Mother Jones* data there are 97 mass shooting events, from 1982 to 2018.

The total number of mass shootings per year is increasing.

``` r
# Little bit of EDA
library(plotly)
theme_set(theme_minimal(base_size = 14))

p0 <- 
  ggplot(us_mass_shootings,
         aes(Year)) +
  geom_bar() +
  ylab("Number of mass shooting events per year")
ggplotly(p0)
```

<img src="figures/unnamed-chunk-3-1.png" style="display: block; margin: auto;" />

The average number of fatalities per event does not appear to be increasing over time, but there are more outlier events with higher numbers of fatalities in the last ten years compared to before.

``` r
p2 <- 
ggplot(us_mass_shootings,
       aes(Date, 
           Fatalities, 
           label = Case)) +
  geom_point() +
  geom_smooth() +
    ylab("Number of fatalities per mass shooting event") +
  xlab("Date")
ggplotly(p2)
```

<img src="figures/unnamed-chunk-4-1.png" style="display: block; margin: auto;" />

The data are sparse for venues, but mass shootings at schools appear to be clustered through time, relative to other types of venues. This suggests that a copy-cat effect may be important for understanding school shootings.

``` r
p1 <- 
ggplot(us_mass_shootings,
       aes(Date, 
           Venue, 
           label = Case)) +
  geom_point() +
  xlab("Date")
ggplotly(p1)
```

<img src="figures/unnamed-chunk-5-1.png" style="display: block; margin: auto;" />

Persistence of Public Interest After Mass Shooting Events
---------------------------------------------------------

To investigate the persistence of public interest in gun control after mass shooting events, we will look at three sources of data to determine the magnitude of public interest directly after an event, and how long that interest persists for in the days after the event. We will look at the 50 most recent events, subset these for events with 10 or more fatalities, and look for persistence over two months after each event. These events with &gt;10 fatalities are widely spaced through time, so cross-contamination of public reaction from one event to another is not a significant factor here. We can be relatively confident that the effects we observe relate to the event we have attributed them to.

``` r
n_events <- 50 # gets back to Sandy Hook
last_ten_shooting_dates <- us_mass_shootings$Date[1:n_events]

# get date before event to show baseline
Query_start_date <- last_ten_shooting_dates - weeks(1)

# get date after event
Query_end_date <- last_ten_shooting_dates + months(2)

# if the date is in the future, set it to today
Query_end_date <- if_else(Query_end_date >  today(),  today(), Query_end_date)

# prepare data frame
search_intervals <- 
data_frame(Case = us_mass_shootings$Case[1:n_events],
           Query_start_date = Query_start_date,
           Event_date = last_ten_shooting_dates, 
           Query_end_date = Query_end_date) %>% 
  filter(!is.na(Event_date),
         !is.na(Query_start_date),
         !is.na(Query_end_date))
```

### Google search volume for 'gun control'

First we will look at Google search volume for the search term 'gun control'. We can make a single query to the Google trends data service to see how many 'hits' this search term has received. The result is a normalised score from 1 to 100, where 100 is the maximum search volume during the query period. Simon Rogers, a Data Editor at Google explains this further [in an essay on Medium](https://medium.com/google-news-lab/what-is-google-trends-data-and-what-does-it-mean-b48f07342ee8). A basic query for 'gun control' on the Google trends service gives us one data point per month, going back to 2004.

``` r
library(gtrendsR)

# we only get one data point per month, ok for long term trends, 
# but not for daily 
google.trends = gtrends(c("gun control"), 
                        gprop = "web", 
                        time = "all")
 
search_results_over_time <- google.trends$interest_over_time
search_results_over_time$the_date <- as.character(search_results_over_time$date)
```

We see a major spike at the end of 2012 and the beginning of 2013, this is the time just after the Sandy Hook Elementary massacre. This is double the search volume at any other event.

``` r
# prepare vertical lines to show when the events occurred
vlines <- 
  search_intervals %>% 
  filter(Case %in% us_mass_shootings$Case[us_mass_shootings$Fatalities >= 10]) %>% 
  mutate(x = Event_date,
         y = seq(50, 100, length.out = nrow(.)),
        label = Case) 

p4 <- 
ggplot(search_results_over_time,
       aes(date,
           hits, 
           label = the_date)) +
  geom_vline(xintercept = vlines$Event_date,
             colour = "red") +
  geom_point() +
  geom_line() +
 #geom_text(data = vlines,
 #          aes(x, 
 #              y, 
 #              label = label), 
 #              hjust = 1) +
  ylab("Google search volume for 'gun control'") +
  xlab("Date") +
  ggtitle("The Sandy Hook event resulted in the most Google search volume for 'gun control'")

# get a halo on the text https://stackoverflow.com/a/10691826/1036500
theta <- seq(pi/8, 2*pi, length.out=16)
xo <- diff(range(vlines$x))/200
yo <- diff(range(vlines$y))/200
for(i in theta) {
  p4 <- p4 + geom_text( data = vlines,
    aes_q(x = bquote(x+.(cos(i)*xo)),
          y = bquote(y+.(sin(i)*yo)),
          label = ~label), 
    size=4, colour='white', hjust = 1)
}

p4 + geom_text(data = vlines,
          aes(x, 
              y, 
              label = label), 
          hjust = 1,
          size = 4,
          colour = "black")
```

<img src="figures/unnamed-chunk-9-1.png" style="display: block; margin: auto;" />

Let's now get finer-grained data with one data point per day for the two months after each mass shooting event. We will query Google trends separately for each mass shooting event the the two months after it. An important limitation here is that the 0-100 scale on the y-axis is no longer comparable between plots for each event. As noted above, the Google trends service scales the 'hits' so that maximum search volume in a given query is scaled to 100. There have been [statistical methods proposed to calibrate across multiple queries](https://link.springer.com/chapter/10.1007/978-3-642-40495-5_11), but I have not used them here. This means that the height of each line is not directly comparable, only the shape and slope.

``` r
# get search results for each shooting event, one data point per day
# much more useful for looking at persistence of interest after event
# this may take a few minutes...
library(glue)
library(purrrlyr)
search_results_over_time_with_cases <- 
by_row(search_intervals,
    ~gtrends(c("gun control"), 
             gprop = "web", 
             time = glue('{.x$Query_start_date} {.x$Query_end_date}'))
    ) 

# extract interest-over-time data frames and 
# unnest into one big df, and
# get days since event
search_results_over_time_with_cases_iot <- 
search_results_over_time_with_cases %>% 
  mutate(interest_over_time = map(.out, ~.x$interest_over_time)) %>% 
  unnest(interest_over_time) %>% 
  mutate(days_relative_to_event = ymd(date) - ymd(Event_date))
```

We can plot some events in a grid of small multiples to get a glance at how the google search volume for 'gun control' drops off after each event. Here we show only events with more than 10 fatalities. The vertical red line indicates the day of the event so we can see the search volume prior to the event, and a background level. We can see that the Fort Hood and Washington Navy Yard events generated few searches for 'gun control' above the background level. For Sandy Hook and San Bernadino we see a substantial spike in search activity after the event, these relate to speeches ([1](https://www.nytimes.com/2016/01/06/us/politics/obama-gun-control-executive-action.html), [2](https://www.nytimes.com/2016/01/06/us/politics/obama-gun-control-executive-action.html)) given by President Obama asking Congress for tougher gun control laws directly following these events. The Orlando event appears to have the most sustained volume of searches for 'gun control' following the event. The Las Vegas event is remarkable for how quickly search volume dropped, given the high number of fatalities at that event. Remember that the y-axis scale is not directly comparable between events for these Google search volume data. This is because the search data result from separate queries to Google Trends for each event. The 100 value on the y-axes of the plots below is only the local maximum for each query, not an absolute value shared by all queries.

``` r
# small mulitples
ggplot(search_results_over_time_with_cases_iot %>% 
           filter(Case %in% us_mass_shootings$Case[us_mass_shootings$Fatalities >= 10]) %>% 
           mutate(Case = str_wrap(Case, 20)) %>% 
           mutate(Case = fct_reorder(Case, year(Event_date))),
       aes(days_relative_to_event,
           hits,
           colour = Case)) +
  geom_line() +
  geom_vline(xintercept = 0, 
             colour = "red") +
  geom_smooth(alpha=0.3, 
              linetype=0) +
  stat_smooth(geom="line", 
              alpha=0.3, 
              size = 2) +
  facet_wrap( ~Case) +
  guides(colour="none") +
  ylim(0, 100) +
  ylab("Google search volume for 'gun control'\n(100 is local maximum for each event)") +
  xlab("Days relative to event (red vertical line is the day of the event)") +
  theme(strip.text.x = element_text(size = 8))
```

<img src="figures/unnamed-chunk-11-1.png" style="display: block; margin: auto;" />

We can overlay all the events on one plot to compare the slopes in the trend lines indicating how many 'hits' were received each day. This plot is a little crowded, but we can make it easier to read by double-clicking on an event in the legend to show only that event, and then single-clicking on other events to make them visible also.

To improve readability, we only show events with more than 10 fatalities. Public interest via Google searches for gun control surrounding these stand-out events seems to have faded by about 20 days after the event. Sandy Hook and San Bernadino because most of this activity happened over a month after the event, prompted by Obama's speeches. So far, the Stoneman Douglas High School resembles the Orlando Nightclub event in the decay curve of Google search volume. The Orlando Nightclub event showed sustained search activity for 11 days after the event. It is too soon to know if the Stoneman Douglas High School will match of exceed this.

``` r
p5 <- # all on one
  ggplot(search_results_over_time_with_cases_iot %>% 
           filter(Case %in% us_mass_shootings$Case[us_mass_shootings$Fatalities >= 10]),
         aes(days_relative_to_event,
             hits,
             colour = Case)) +
  geom_line(alpha = 0.9) +
  geom_vline(xintercept = 0, 
             colour = "red") +
  ylab("Google search volume for 'gun control'") +
  xlab("Days relative to event (red vertical line is the day of the event)") +
  guides(colour=FALSE)
ggplotly(p5)
```

<img src="figures/unnamed-chunk-12-1.png" style="display: block; margin: auto;" />

### Page views of Wikipedia articles on gun control

Second, we will look at public interest via traffic to Wikipedia pages related to mass shootings and gun control. These data only go back as far as 2015, so we cannot compare with Sandy Hook and other events before 2015.

``` r
library(pageviews) # only goes back to 2015

pages <- c("Gun_control", 
           "Gun_violence_in_the_United_States",
           "Mass_shootings_in_the_United_States",
           "Second_Amendment_to_the_United_States_Constitution")

trend_data <- 
article_pageviews(project = "en.wikipedia",
                  article = pages, 
                  platform = "all",
                  user_type = "all",
                  start = as.Date(min(search_intervals$Query_start_date)), 
                  end =   as.Date(today()))

trend_data$date <-  ymd(trend_data$date)
```

We will focus on page views over time of this set of Wikipedia pages: <https://en.wikipedia.org/wiki/Gun_control>, <https://en.wikipedia.org/wiki/Gun_violence_in_the_United_States>, <https://en.wikipedia.org/wiki/Mass_shootings_in_the_United_States>, <https://en.wikipedia.org/wiki/Second_Amendment_to_the_United_States_Constitution>. In the plot below we see that the article about the Second Amendment is by far the most frequently visited site of this set. This plot shows all the mass shootings since 2015, and how many of them don't generate any substantial increase in Wikipedia page views. The Orlando vent is significant here because it appears to have prompted the creation of the article "Mass shootings in the United States. The Stoneman Douglas High School event is remarkable because it has generated the most traffic to these pages of all the events. The Las Vegas and Orlando events also generated a lot of traffic to Wikipedia.

``` r
# prepare vertical lines to show when the events occurred
n <- 30
vlines <- 
  data_frame(x = search_intervals$Event_date[1:n],
             y = seq(1e5, 1, length.out = n),
             label = search_intervals$Case[1:n])
p11 <- 
ggplot(trend_data,
       aes(date,
           views,
           colour = article)) +
  geom_vline(data = search_intervals,
             aes(xintercept = Event_date),
             colour = "red",
             alpha = 0.3) +
  geom_line(size = 1)  +
  xlim( min(trend_data$date), max(trend_data$date) ) +
  scale_colour_discrete(guide = guide_legend()) +
  ylab("Page views for Wikipedia articles\nrelated to gun control") +
  xlab("Date") +
  labs(colour = "Wikipedia article") +
  theme(legend.position = c(0.2, 0.8),
        legend.text = element_text(size = 7),
        legend.key.height = unit(0.5,"line")) +
  ggtitle("The Stoneman Douglas event resulted in the most Wikipedia\npage views for 'gun control' articles (2015-present)")

# get a halo on the text https://stackoverflow.com/a/10691826/1036500
theta <- seq(pi/8, 2*pi, length.out=16)
xo <- diff(range(vlines$x))/200
yo <- diff(range(vlines$y))/200
for(i in theta) {
  p11 <- p11 + geom_text( data = vlines,
    aes_q(x = bquote(x+.(cos(i)*xo)),
          y = bquote(y+.(sin(i)*yo)),
          label = ~label), 
    size=3, colour='white', hjust = 1)
}
p11 + geom_text(data = vlines,
          aes(x, 
              y, 
              label = label), 
          hjust = 1,
          size = 3,
          colour = "black")
```

<img src="figures/unnamed-chunk-14-1.png" style="display: block; margin: auto;" />

However, there are two events unrelated to mass shootings that have also generated high numbers of page views to the Second Amendment Wikipedia page. The first of these, on 10 Aug 2016 relates to [a speech at a rally by Donald Trump](https://www.nytimes.com/2016/08/10/us/politics/donald-trump-hillary-clinton.html) in which he claimed that Hillary Clinton wanted to abolish the right to bear arms. The second peak, on 20 Oct 2016, relates to discussion of the Second Amendment during the [third presidential debate](https://www.politico.com/story/2016/10/full-transcript-third-2016-presidential-debate-230063) before the 2016 election.

We can zoom in on these data to see how the page views reflect persistence of interest in the topic in the days after the event. The plot below shows total page views for all the articles in the set, and how the number of page views changes in the days after each event where there were more than ten fatalities. We see that the Stoneman Douglas High School event stands out above all the others. We can immediately see that the Stoneman Douglas High School event is different from other previous events because of the very high number of page views of Wikipeadia pages relating to gun control.

``` r
library(fuzzyjoin)

wikipedia_traffic_for_events <- 
fuzzy_left_join(trend_data,
                search_intervals, 
                by = c("date" = "Query_start_date", 
                       "date" = "Query_end_date"), 
                match_fun = list(`>=`, `<`)) %>% 
  mutate(days_relative_to_event = ymd(date) - ymd(Event_date))

wiki_traffic_total_views_over_days <- 
wikipedia_traffic_for_events %>% 
  group_by(Case, days_relative_to_event) %>% 
  summarise(total_views = sum(views)) %>% 
  filter(!is.na(Case))

p6 <- 
ggplot(wiki_traffic_total_views_over_days %>% 
           filter(Case %in% us_mass_shootings$Case[us_mass_shootings$Fatalities >= 10]),
       aes(days_relative_to_event,
           total_views, 
           colour = Case)) +
  geom_vline(xintercept = 0,
             colour = "red") +
  geom_line(alpha = 0.9) +
  ylab("Page views for Wikipedia articles\nrelated to gun control") +
  xlab("Days relative to event\n(red vertical line is the day of the event)") 
ggplotly(p6)
```

<img src="figures/unnamed-chunk-16-1.png" style="display: block; margin: auto;" />

We might predict that the number of fatalities is correlated with the amount of public interest in gun control following a mass shooting event. There does seem to be a slight effect, with more pageviews associated with higher fatalities. The Texas First Baptist Church is unusual because it has fewer page views than expected for a massacre of that size.

``` r
wiki_traffic_and_mass_shooting_data <- 
wikipedia_traffic_for_events %>% 
  group_by(Case) %>% 
  summarise(total_views = sum(views)) %>% 
  left_join(us_mass_shootings) 

p7 <- 
ggplot(wiki_traffic_and_mass_shooting_data %>% filter(!is.na(Case)),
       aes(total_views,
           Fatalities, 
           label = Case)) +
  geom_point(size = 3) +
  geom_smooth(method = "lm") +
  ylab("Fatalities") +
  xlab("Page views for Wikipedia articles related to gun control") 
ggplotly(p7)
```

<img src="figures/unnamed-chunk-17-1.png" style="display: block; margin: auto;" />

### Broadcast television news coverage of gun control

We can investigate broadcast television news coverage of gun control using data from the [Internet Archive's Television News Archive](https://archive.org/details/tv). I searched for 'gun control' and downloaded the CSV from the [results page](https://api.gdeltproject.org/api/v2/summary/summary?DATASET=IATV&TYPE=SUMMARY&KEYWORDS=gun+control&TIMESPAN=&STARTDATETIME=&ENDDATETIME=&FILTER_STATIONS=station%3ACNN&FILTER_STATIONS=station%3AFOXNEWS&FILTER_STATIONS=station%3AMSNBC&FILTER_COMBSEP=&FILTER_TIMESMOOTH=&SHOW_VOLTIMELINE=INCLUDE_ZOOMABLE&SHOW_STATIONCHART=INCLUDE&SHOW_WORDCLOUD=INCLUDE&SHOW_TOPCLIPS=INCLUDE&CREATE=1). We have daily data going back to 2009 for MSNBC, CNN and FOX. The measurement variable is % airtime (15 second blocks).

``` r
# devtools::install_github("hrbrmstr/newsflash")
# library(newsflash)
# gun_control_tv <- query_tv("gun control")

# doesn't work, so I went to  https://api.gdeltproject.org/api/v2/summary/summary?DATASET=IATV&TYPE=SUMMARY&KEYWORDS=gun+control&TIMESPAN=&STARTDATETIME=&ENDDATETIME=&FILTER_STATIONS=station%3ACNN&FILTER_STATIONS=station%3AFOXNEWS&FILTER_STATIONS=station%3AMSNBC&FILTER_COMBSEP=&FILTER_TIMESMOOTH=&SHOW_VOLTIMELINE=INCLUDE_ZOOMABLE&SHOW_STATIONCHART=INCLUDE&SHOW_WORDCLOUD=INCLUDE&SHOW_TOPCLIPS=INCLUDE&CREATE=1

tv_data <- read_csv("data/tv_results.csv")
tv_data$Date <- ymd(tv_data$Date)

tv_data_for_events <- 
fuzzy_left_join(tv_data,
                search_intervals, 
                by = c("Date" = "Query_start_date", 
                       "Date" = "Query_end_date"), 
                match_fun = list(`>=`, `<`)) %>% 
  mutate(days_relative_to_event = ymd(Date) - ymd(Event_date)) 

tv_data_for_events$days_relative_to_event <- 
as.numeric(tv_data_for_events$days_relative_to_event)
```

For most mass shooting events, gun control fades from the news within two weeks. Of the larger massacres, Orlando and Las Vegas faded from the news after about 10-15 days. Sandy Hook Elemantary and San Bernadino are exceptions due to Obama's speeches. The Stoneman Douglas High School event has generated a lot of news discussion about gun control, more than Las Vegas and San Bernadino, and perhaps a similar amount as Sandy Hook in the first week after the event.

``` r
p8 <- 
ggplot(tv_data_for_events %>% 
           filter(Case %in% us_mass_shootings$Case[us_mass_shootings$Fatalities >= 10]),
       aes(jitter(days_relative_to_event),
           Value,
           colour = Case)) +
  geom_line(alpha = 0.2) +
  geom_smooth(se = FALSE,
              span = 0.1) +
  geom_vline(xintercept = 0,
             colour = "red") +
  guides(colour = FALSE) +
  ylab("% airtime (15 second blocks)") +
  xlab("Days relative to event\n(red vertical line is the day of the event)")

# geom_vline doesn't seem to work with plotly here, not sure why,
# so we add it back in
ggplotly(p8, originalData = FALSE) %>%
  mutate(zero = min(y)) %>% 
  mutate(inf = max(y)) %>% 
  add_fun(function(p) {
    p %>% slice(which(x == 0))  %>% 
      add_segments(x = ~x, 
                   xend = ~x, 
                   y = ~zero, 
                   yend = ~inf,
                   colour = "red") 
  })
```

<img src="figures/unnamed-chunk-19-1.png" style="display: block; margin: auto;" />

Summary
-------

We have looked at how mass shootings generate public interest in gun control in three ways: by measuring Google search volume for the term 'gun control' in the days after the event, by measuring page views of Wikipedia articles relating to gun control in the days after the event, and by measuring the amount of time that broadcast television news coverage dedicate to gun control stories. With these three methods we have investigated decay rates in public interest in gun control after a mass shooting event.

Although we are less than a month out from the Stoneman Douglas High School event, it appears to resemble previous mass shooting events in generating and sustaining public interest in gun control as measured by Google search volume and TV broadcast news content. Sandy Hook is the stand-out event for Google search volume, with more than twice the number of hits of any other event. Sandy Hook is also the dominant event in the broadcast television news coverage, but Stoneman Douglas High School event is very close. The Orlando event appears to have had the most sustained Google search volume in the days following the event, but limitations in the data provided by Google make close comparisons difficult. We can rerun this analysis in a few weeks to see how the Stoneman Douglas High School compares to Orlando over a longer time.

The most striking difference between Stoneman Douglas High School event and previous mass shootings is in high number of page views to Wikipedia articles that is has generated, and the sustained interest in viewing these articles in the days after the event. We might speculate that this indicates that people are seeking facts and reliable context about mass shootings. This may be due to a heightened interest in understanding why mass shootings occur, and what can be done to stop them. We hope, of course, that this heightened interest will soon translate into effective measures to prevent future mass shootings. There are many state-level laws that have [well-documented and robust effects on reducing gun violence](http://lawcenter.giffords.org/facts/research/). These effective state-level laws hint at a way forward for federal legislation that will reduce the likelihood of mass shooting events in the future.

Colophon
--------

This document was written in R Markdown. The code and data for this document is online at <https://github.com/benmarwick/test2>.

This document was generated on 2018-02-25 23:21:54 using the following computational environment and dependencies:

``` r
sessioninfo::session_info()
```

    ## - Session info ----------------------------------------------------------
    ##  setting  value                       
    ##  version  R version 3.4.3 (2017-11-30)
    ##  os       Windows 7 x64 SP 1          
    ##  system   x86_64, mingw32             
    ##  ui       RStudio                     
    ##  language (EN)                        
    ##  collate  English_Australia.1252      
    ##  tz       America/Los_Angeles         
    ##  date     2018-02-25                  
    ## 
    ## - Packages --------------------------------------------------------------
    ##  package      * version    date      
    ##  anytime        0.3.0      2017-06-05
    ##  assertthat     0.2.0      2017-04-11
    ##  backports      1.1.2      2017-12-13
    ##  bindr          0.1        2016-11-13
    ##  bindrcpp     * 0.2        2017-06-17
    ##  broom          0.4.3      2017-11-20
    ##  cellranger     1.1.0.9000 2017-05-31
    ##  cli            1.0.0      2017-11-05
    ##  clisymbols     1.2.0      2017-06-14
    ##  codetools      0.2-15     2016-10-05
    ##  colorspace     1.3-2      2016-12-14
    ##  crayon         1.3.4      2018-02-10
    ##  crosstalk      1.0.0      2016-12-21
    ##  curl           3.1        2017-12-12
    ##  data.table     1.10.4-3   2017-10-27
    ##  devtools       1.13.4     2017-11-09
    ##  digest         0.6.15     2018-01-28
    ##  dplyr        * 0.7.4      2017-09-28
    ##  evaluate       0.10.1     2017-06-24
    ##  forcats      * 0.2.0      2017-01-23
    ##  foreign        0.8-69     2017-06-21
    ##  fuzzyjoin    * 0.1.3      2017-06-19
    ##  ggplot2      * 2.2.1.9000 2018-02-26
    ##  git2r          0.21.0     2018-01-04
    ##  glue         * 1.2.0      2017-10-29
    ##  gtable         0.2.0      2016-02-26
    ##  gtrendsR     * 1.4.1      2018-02-23
    ##  haven          1.1.1      2018-01-18
    ##  hms            0.4.1      2018-01-24
    ##  htmltools      0.3.6      2017-04-28
    ##  htmlwidgets    1.0        2018-01-20
    ##  httpuv         1.3.5      2017-07-04
    ##  httr           1.3.1      2017-08-20
    ##  jsonlite       1.5        2017-06-01
    ##  knitr          1.19       2018-01-29
    ##  labeling       0.3        2014-08-23
    ##  lattice        0.20-35    2017-03-25
    ##  lazyeval       0.2.1      2017-10-29
    ##  lubridate    * 1.7.1      2017-11-03
    ##  magrittr       1.5        2014-11-22
    ##  memoise        1.1.0      2017-04-21
    ##  mime           0.5        2016-07-07
    ##  mnormt         1.5-5      2016-10-15
    ##  modelr         0.1.1      2017-07-24
    ##  munsell        0.4.3      2016-02-13
    ##  nlme           3.1-131    2017-02-06
    ##  pageviews    * 0.3.0      2016-10-17
    ##  pillar         1.1.0.9000 2018-02-10
    ##  pkgconfig      2.0.1      2017-03-21
    ##  plotly       * 4.7.1      2017-07-29
    ##  plyr           1.8.4      2016-06-08
    ##  psych          1.7.8      2017-09-09
    ##  purrr        * 0.2.4      2017-10-18
    ##  purrrlyr     * 0.0.2      2017-05-13
    ##  R6             2.2.2      2017-06-17
    ##  RApiDatetime   0.0.3      2017-04-02
    ##  RColorBrewer   1.1-2      2014-12-07
    ##  Rcpp           0.12.15    2018-01-20
    ##  readr        * 1.1.1      2017-05-16
    ##  readxl         1.0.0      2017-04-18
    ##  reshape2       1.4.3      2017-12-11
    ##  rlang          0.2.0.9000 2018-02-26
    ##  rmarkdown      1.8        2017-11-17
    ##  rprojroot      1.3-2      2018-01-03
    ##  rstudioapi     0.7.0-9000 2017-12-23
    ##  rvest          0.3.2      2016-06-17
    ##  scales         0.5.0.9000 2018-02-26
    ##  sessioninfo    1.0.0      2017-07-28
    ##  shiny          1.0.5      2017-08-23
    ##  stringi        1.1.6      2017-11-17
    ##  stringr      * 1.2.0      2017-02-18
    ##  tibble       * 1.4.2      2018-01-22
    ##  tidyr        * 0.8.0      2018-01-29
    ##  tidyselect     0.2.3      2017-11-06
    ##  tidyverse    * 1.2.1      2018-02-05
    ##  utf8           1.1.3      2018-01-03
    ##  viridisLite    0.3.0      2018-02-01
    ##  webshot        0.5.0      2017-11-29
    ##  withr          2.1.1.9000 2018-02-26
    ##  xml2           1.2.0      2018-01-24
    ##  xtable         1.8-2      2016-02-05
    ##  yaml           2.1.16     2017-12-12
    ##  source                                 
    ##  CRAN (R 3.4.3)                         
    ##  CRAN (R 3.4.0)                         
    ##  CRAN (R 3.4.3)                         
    ##  CRAN (R 3.4.0)                         
    ##  CRAN (R 3.4.0)                         
    ##  CRAN (R 3.4.3)                         
    ##  Github (rsheets/cellranger@024d5ba)    
    ##  CRAN (R 3.4.2)                         
    ##  Github (gaborcsardi/clisymbols@e49b4f5)
    ##  CRAN (R 3.4.0)                         
    ##  CRAN (R 3.4.0)                         
    ##  Github (gaborcsardi/crayon@95b3eae)    
    ##  CRAN (R 3.4.0)                         
    ##  CRAN (R 3.4.3)                         
    ##  CRAN (R 3.4.2)                         
    ##  CRAN (R 3.4.2)                         
    ##  CRAN (R 3.4.3)                         
    ##  CRAN (R 3.4.2)                         
    ##  CRAN (R 3.4.0)                         
    ##  CRAN (R 3.4.0)                         
    ##  CRAN (R 3.4.0)                         
    ##  CRAN (R 3.4.0)                         
    ##  Github (tidyverse/ggplot2@39e4a3b)     
    ##  CRAN (R 3.4.3)                         
    ##  CRAN (R 3.4.2)                         
    ##  CRAN (R 3.4.0)                         
    ##  Github (PMassicotte/gtrendsR@7b3dc40)  
    ##  CRAN (R 3.4.3)                         
    ##  CRAN (R 3.4.3)                         
    ##  CRAN (R 3.4.0)                         
    ##  CRAN (R 3.4.3)                         
    ##  CRAN (R 3.4.1)                         
    ##  CRAN (R 3.4.1)                         
    ##  CRAN (R 3.4.0)                         
    ##  CRAN (R 3.4.3)                         
    ##  CRAN (R 3.4.0)                         
    ##  CRAN (R 3.4.3)                         
    ##  CRAN (R 3.4.2)                         
    ##  CRAN (R 3.4.2)                         
    ##  CRAN (R 3.4.0)                         
    ##  CRAN (R 3.4.0)                         
    ##  CRAN (R 3.4.0)                         
    ##  CRAN (R 3.4.0)                         
    ##  CRAN (R 3.4.1)                         
    ##  CRAN (R 3.4.0)                         
    ##  CRAN (R 3.4.3)                         
    ##  CRAN (R 3.4.3)                         
    ##  Github (r-lib/pillar@595d1ac)          
    ##  CRAN (R 3.4.0)                         
    ##  CRAN (R 3.4.0)                         
    ##  CRAN (R 3.4.0)                         
    ##  CRAN (R 3.4.2)                         
    ##  CRAN (R 3.4.2)                         
    ##  CRAN (R 3.4.0)                         
    ##  CRAN (R 3.4.0)                         
    ##  CRAN (R 3.4.1)                         
    ##  CRAN (R 3.4.0)                         
    ##  CRAN (R 3.4.3)                         
    ##  CRAN (R 3.4.0)                         
    ##  CRAN (R 3.4.3)                         
    ##  CRAN (R 3.4.3)                         
    ##  Github (tidyverse/rlang@3143f00)       
    ##  CRAN (R 3.4.2)                         
    ##  CRAN (R 3.4.3)                         
    ##  Github (rstudio/rstudioapi@109e593)    
    ##  CRAN (R 3.4.0)                         
    ##  Github (hadley/scales@d767915)         
    ##  Github (r-lib/sessioninfo@8de1163)     
    ##  CRAN (R 3.4.1)                         
    ##  CRAN (R 3.4.2)                         
    ##  CRAN (R 3.4.0)                         
    ##  CRAN (R 3.4.3)                         
    ##  CRAN (R 3.4.3)                         
    ##  CRAN (R 3.4.2)                         
    ##  Github (hadley/tidyverse@03ccf9c)      
    ##  CRAN (R 3.4.3)                         
    ##  CRAN (R 3.4.3)                         
    ##  CRAN (R 3.4.3)                         
    ##  Github (jimhester/withr@5d05571)       
    ##  CRAN (R 3.4.3)                         
    ##  CRAN (R 3.4.0)                         
    ##  CRAN (R 3.4.3)

The current Git commit details are:

``` r
git2r::repository(".")
```

    ## Local:    master C:/Users/bmarwick/Desktop/us_mass_shooting_events/
    ## Remote:   master @ origin (https://github.com/benmarwick/test2.git)
    ## Head:     [97119b4] 2018-02-25: initial
