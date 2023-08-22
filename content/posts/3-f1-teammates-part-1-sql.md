---
title: 'Comparing the head-to-head performance of F1 teammates, part 1: Retreiving Ergast data using SQL'
date: 2023-07-28T23:12:51-07:00
draft: false
tags: [SQL, Motorsports, Formula 1, Ergast]
---

My goal with this project was to develop a [Formula 1 visualizer](https://stevenrossi.shinyapps.io/f1-teammates/) that analysed relatively basic data over a broad time scale, without getting into the more detailed data (e.g., lap-specific times, telemetry) that has become more readily available in recent years.

In Formula 1 there are well known difficulties in comparing the relative performance of drivers given that drivers compete in machinery of varying quality, i.e., did driver A finish ahead of driver B because he is a better driver or because he was driving a faster car? As such, the only fair comparison that can be made in races without controlling for a number of complex factors is between teammates, who (typically) compete in identical cars. I therefore sought to develop an analysis that only compared the performance of teammates to one another.

In this post I will review how I pulled the necessary data for this project and in follow-up posts I will detail some data manipulation using `dplyr` in R and the contruction of a [Shiny app](https://stevenrossi.shinyapps.io/f1-teammates/). For now, I only focus on the performance of teammates against each other in Grand Prix races, though in the future I'd like to add comparisons from qualifying, and less importantly, from sprint races.

The [Ergast Developer API](http://ergast.com/mrd/) hosts a large Formula 1 dataset that can be used for a variety of purposes.  The dataset includes results from each race and information on drivers, constructors, and circuits from the beginning of the world championships in 1950 to the present day. Other, more detailed but less complete data is also contained in the dataset, such as lap times and pit stop durations. The data can be downloaded several ways, including via API and as a set of [.csv files](http://ergast.com/downloads/f1db_csv.zip). In this post, I'll review retrieving the data from a MySQL database image, which can be downloaded [here](http://ergast.com/downloads/f1db.sql.gz).

In order to compare the relative performance of two teammates on race day, we'll need their finishing position in each race. The Egast Database contains [14 tables](http://ergast.com/images/ergast_db.png), one of which, named `results`, appears to contain the finishing order (or, `positionOrder`) in each race:

<br>

[<img src="/images/post3/ergast_db.png">](/images/post3/ergast_db.png)

<br>

The `raceId`, `driverId`, and `constructorId` columns in `results` are linked to the `races`, `drivers`, and `constructors` tables, respectively. These tables should contain everything we'll need for the teammate comparison dataset. Let's give the `results`, `races`, and `drivers` tables the aliases `r`, `a`, `d`, and `c` respectively.

We construct our dataset with the following SQL code:

<br>

```sql
SELECT r.driverId,
       r.constructorId,
       positionText,
       positionOrder,
       r.raceId,
       year,
       date,
       round,
       a.name AS raceName,
       c.name AS constructorName,
       CONCAT(surname, ', ', forename) AS driverName,
       d.number,
       d.url,
       dob,
       d.nationality
FROM sql_inventory.results r
JOIN races a
    ON r.raceId = a.raceId
JOIN drivers d
    ON r.driverId = d.driverId
JOIN constructors c
    ON r.constructorId = c.constructorId;
```

This script grabs the `driverID`, `constructorID`, `positionText`, `positionOrder`, and `raceID` from the `results`table. `positionText` gives the numeric finishing position of each car that finished the race and otherwise gives a letter indicating the reason for not finishing the race. In contrast, `positionOrder` gives the numeric finishing position regardless of whether or not the race was completed. Either could be handy, so we grab both.

The selected columns of `results` are appended by the `year`, `date`, and `round` from the `races` table and the `number`, `url`, `dob`, and `nationality` columns from the `drivers` table. We also contruct a `driverName` column by combining the `surname` and `forename` columns from the `drivers` table.

The three lines beginning with `JOIN` and `ON` link the `results` tables with the other tables via common columns.


The resulting dataset is saved to `f1-teammate-data-raw.csv`. Let's read this data into R and take a quick look:

<br>

```R
rawData <- read.csv("f1-teammate-data-raw.csv")

rawData
```
```
  driverId constructorId positionText positionOrder raceID year       date round              raceName
1        1             1            1             1     18 2008 2008-03-16     1 Australian Grand Prix
2        2             2            2             2     18 2008 2008-03-16     1 Australian Grand Prix
3        3             3            3             3     18 2008 2008-03-16     1 Australian Grand Prix
4        4             4            4             4     18 2008 2008-03-16     1 Australian Grand Prix
5        5             1            5             5     18 2008 2008-03-16     1 Australian Grand Prix
6        6             3            6             6     18 2008 2008-03-16     1 Australian Grand Prix
  constructorName         driverName number                                            url        dob
1         McLaren    Hamilton, Lewis     44    http://en.wikipedia.org/wiki/Lewis_Hamilton 1985-01-07
2      BMW Sauber     Heidfeld, Nick   NULL     http://en.wikipedia.org/wiki/Nick_Heidfeld 1977-05-10
3        Williams      Rosberg, Nico      6      http://en.wikipedia.org/wiki/Nico_Rosberg 1985-06-27
4         Renault   Alonso, Fernando     14   http://en.wikipedia.org/wiki/Fernando_Alonso 1981-07-29
5         McLaren Kovalainen, Heikki   NULL http://en.wikipedia.org/wiki/Heikki_Kovalainen 1981-10-19
6        Williams   Nakajima, Kazuki   NULL   http://en.wikipedia.org/wiki/Kazuki_Nakajima 1985-01-11
  nationality points
1     British     10
2      German      8
3      German      6
4     Spanish      5
5     Finnish      4
6    Japanese      3
```

Looks as though everything is in order here. In the next post, I will show how I manupulated this dataset using `dplyr` to calculate the relative performance between teammates.







