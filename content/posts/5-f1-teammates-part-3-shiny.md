---
title: 'Comparing the head-to-head performance of F1 teammates, part 3: Shiny app'
date: 2023-08-20T01:01:01-07:00
draft: true
tags: [Motorsports, Formula 1, Ergast, R, Shiny, Visualization]
---


The goal of this project is to develop a [Shiny app](https://stevenrossi.shinyapps.io/f1-teammates/) to compare the head-to-head performance of Formula 1 teammates.

In the [previous post](/posts/3-f1-teammates-part-1-sql/) I used SQL to pull historical race results from the [Ergast](http://ergast.com/mrd/) database, which I saved to a file called [f1-teammate-data-raw.csv](https://github.com/stevenrossi/f1-teammates/blob/main/f1-teammate-data-raw.csv). In this post, I'll detail how I transformed the data using the handy `dplyr` package in `R` to obtain head-to-head comparisons.

The main metric I am aiming to compare teammates by is their relative difference in race finishing position. I'm ignoring qualifying performance at the moment, but I'd like to circle back to it in the future.

Each row of [f1-teammate-data-raw.csv](https://github.com/stevenrossi/f1-teammates/blob/main/f1-teammate-data-raw.csv) contains the result for a particular driver and race. Among the information is the driver's name, finishing position, and number of points scored. Let's add columns to the dataset (via the `mutate()` function in `dplyr`) so that the name, finishing position and number of points scored by each driver's teammate is also contained on each row. I also add a column for the relative difference in finishing position between a driver and their teammate (the main quantity of interest) and a column to store whether the race winner was the driver of interest, their teammate, or neither driver.

Additionally, it would be nice to collapse the date of each race into a single numeric value (`yearx`) to make time-series plotting easier. To do this, we convert the date column to objects of class "`Date`", extract the Julian day of year using the `as.POSIXlt()` function, then combine the year and the proportion of the year elapsed:

```R
library(dplyr)

dat <- read.csv(file="f1-teammate-data-raw.csv") %>%
       mutate( Teammate         = NA,
               teammatePosition = NA,
               teammatePoints   = NA,
               relPosition      = NA,
               raceWinner       = "Neither",
               date             = as.Date(date),
               julian           = as.POSIXlt(date)$yday,
               yearx            = year + julian/365 ) %>%
       arrange( year, round )
```

In the modern era of Formula 1, teams cannot run more than two cars in a race. This makes teammate comparisons relatively straightforward, since each driver generally always has exactly one teammate. However, in earlier days of F1, teams of three or larger sometimes raced, making teammate comparisons more complicated. For instance, should a driver be compared to all their teammates (perhaps mean team performance) or only their fastest teammate? For now we leave these questions aside and only consider 1v1 teammate comparisons.

To fill-in teammate information we loop over every race and return records for all teammates in that race, which is done using the `filter()` function in `dplyr`. Each race is uniquely determined by a `year` and `round` value. Additionally, teammates will have the same `constructorId`. We can therefore pull the results of all teammates in a given race by pulling all records matching these three values. We then filter out the driver of interest to obtain only the records of teammates. If there is more than one teammate, we don't record the teammate data.

```R
for( i in 1:nrow(dat) )
{
  # Get teammates data
  teamDat <- dat %>%
             filter( year          == dat[i,"year"],
                     round         == dat[i,"round"]
                     constructorId == dat[i,"constructorId"],
                     driverName    != dat[i,"driverName"],
                     
                )

  # If only 1 teammate, record the data
  if( nrow(teamDat) == 1  )
  {  
    dat[i,"relPosition"] <- teamDat[1,"positionOrder"]-dat[i,"positionOrder"]
    dat[i,"Teammate"] <- teamDat[1,"driverName"]
    dat[i,"teammatePosition"] <- teamDat[1,"positionOrder"]
    dat[i,"teammatePoints"] <- teamDat[1,"points"]
  }
}
```

Now let's fill in the `raceWinner` column and save the dataset:

```R
driverWins   <- dat[ ,"positionOrder"]==1
teammateWins <- dat[ ,"teammatePosition"]==1

dat[driverWins,"raceWinner"]   <- dat[driverWins,"driverName"]
dat[teammateWins,"raceWinner"] <- "Teammate"

write.csv(dat,file="f1-teammate-data.csv")
```
To demonstrate how the dataset can be used, let's plot a time-series of current F1 champion Max Verstappen's performance against his teammates. Since debuting as 17 year old for Red Bull junior team Toro Rosso, Verstappen has acquired something of a reputation as a teammate-killer. Let's see what the data says. We'll use colours to indicate teammates and shapes to indicate the race winner. `ggplot` is well-suited for quickly generating this sort of plot.

```R
library(ggplot2)

mvDat <- filter(dat,driverName=="Verstappen, Max")

# Refactor temmate column so teammates show up in chorological order in the legend
mvDat$Teammate <- factor( x     = mvDat$Teammate,
                         levels = unique(mvDat$Teammate) )

# Graphics device for storing plot
png(file="verstappenTeammateComparison.png",height=4.5,width=9,res=300,units="in")

# Create ggplot
a <- ggplot( mvDat,
             aes( x=yearx,
                  y=relPosition,
                  colour=factor(Teammate),
                  shape = factor(raceWinner)) )

# Plot
a +
  geom_point() +
  ylim(-20,20) +
  xlab("Year") +
  ylab("Fishing position relative to teammate") + 
  scale_x_continuous( breaks=seq(2015,2023,by=2) ) + 
  geom_hline(yintercept=0) +
  labs(color="Teammate",shape="Race Winner") + 
  ggtitle("Max Verstappen vs. Formula 1 Teammates") + 
  theme(plot.title = element_text(hjust = 0.5)) # Center title

# Turn off plotting device
dev.off()


```

<img src="/images/post4/verstappenTeammateComparison.png">

We can see that Verstappen had some struggles early in 2015 season against Carlos Sainz, Jr., but recovered to comprehensively beat his teammate in the latter half of the season. Verstappen was bumped up to the senior Red Bull team early in the 2016 season to partner Danny Ricciardo and famously [won his first race](https://www.youtube.com/watch?v=hohuswdeznA) for the team, which we see on the plot represented by a square. There was a pretty wide disparity in race-to-race performance between Verstappen and Ricciardo in 2017, with one Red Bull often finishing high up in the order and the other Red Bull not finishing. Max firmly had the upper hand on Danny in 2018, particularly in the latter half of the season, where we observe no points below the x-axis. Ricciardo would leave Red Bull for the 2019 season, to be replaced by Pierre Gasly, who greatly struggled to match Max's pace. Verstappen beat Gasly in all but one race (note the solitary green data point below the x-axis), leading Red Bull to replace him with Alex Albon. Albon fared better against Verstappen, but ultimately failed to beat him on a consistent basis. Red Bull replaced Albon with Sergio "Checo" Pérez for the 2021 season onward. While Checo clearly joined the long list of drivers who could not keep pace with Max in equal machinery, the dominance of the Red Bull car really stands out on the plot - note how many data points from 2022 onward, including all of the points in 2023, are represented by a triangle or square, indicating that Max or Checo won the race.

So is Verstappen a teammate killer? He outperformed all of his teammates - exceptionally so from 2018 onward. Max's overperformance against Ricciardo in the latter half of 2018 is striking and is the best example of Verstappen sending a teammate into a tailspin - Ricciardo would subsequently leave Red Bull to drive a less competitive Renault and his career began a downward progression from which it has not recovered. Max's overperformance against Gasly also stongly stands out and Gasly's reputation took a huge hit from his time partnering Max. However, Gasly would later rebuild his career, winning the 2020 Italian Grand Prix and emerging as a strong driver. Albon, too, joined a lesser team after being beaten by Max, rebuilt his career, and is now being mooted to fill vacancies at top teams. Despite Checo being comprehensively beaten by Max and struggling through much of the 2023 season, it would be difficult to say Max is killing Checo's career since Checo's career already appeared over before being thrown a lifeline by Red Bull. Indeed, virtually all of Checo's career highlights have come at Red Bull. Though it seems that Checo's inability to keep up with Max's pace on the track has created tension in the team in unexpected ways. For instance, at the 2022 Monaco Grand Prix, Perez is alleged to have [intentionally crashed](https://www.youtube.com/watch?v=fFImqBddhfs) in qualifying in order to put himself ahead of Max on the starting grid. Checo was presumably willing to (allegedly) resort to such tactics because he knew he stood little chance of beating Max otherwise. This (alleged) incident has led to some well publicized disputes and many would not be surprised if Red Bull showed Checo the door at the end of the 2023 season.

In my next post, I'll detail how I constructed a [Shiny app](https://stevenrossi.shinyapps.io/f1-teammates/) to compare the performance of any driver against their teammate.










