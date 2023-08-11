---
title: Visualizing the age distribution of Women's World Cup squads using R
date: 2023-07-21T23:12:51-07:00
draft: true
tags: [R, Football, Data Scraping, Visualization]
output:
  html_document:
    code_folding: show
---

With the 2023 FIFA Women's World Cup kicking off today in Australia and New Zealand, I thought I would try to visualize some statistics for all of the participating players. Detailed data on the women's game is not as readily available as data on the men's game, so my focus is on some basic statistics for which data is likely to be available (e.g., player age, caps, goals, assists, etc.).

A total of 32 teams qualify for the World Cup, which proceeds in two stages. First the teams are divided into 8 groups of 4, with each team playing every other team in their group once. Three points are earned for a win, 1 point for a draw, and no points for a loss. At the end of the group stage, the two teams in each group with the most points advance to a knockout stage in which teams are paired off "tournament-style" in a series of winner-takes-all matches.

Each of the 32 teams is required to register 23 players, of which 3 must be goalkeepers. This gives a total of 736 players competing in the tournament: 640 outfield players and 96 goalkeers.

# Reading-in data

The first order of business is acquiring squad lists for each team. I found an official PDF from FIFA listing the full squads for each team [(link)](https://fdp.fifa.org/assetspublic/ce93/pdf/SquadLists-English.pdf) but the lists of players is spread across multiple tables and there is a lot of extraneous information - both of these things tend to cause problems for PDF-scraping tools. I instead found a Wikipedia article listing the players for each team [(link)](https://en.wikipedia.org/wiki/2023_FIFA_Women%27s_World_Cup_squads) and figured these tables would be easier to retreive.

The `GET()` function in the `httr` package can be used to scrape websites for information. We can then use the `readHTMLTable()` function in the `XML` package read all of the tables in the scraped data:

<br>

```R
library(httr)
library(XML)

wiki <- GET("https://en.wikipedia.org/wiki/2023_FIFA_Women%27s_World_Cup_squads")

wikiTables <- readHTMLTable( doc=content(wiki,"text"), header=TRUE )
```

<br>

As an aside, there is an specific order in which the 32 teams are listed in official FIFA communications. Specifically, teams are first ordered by group (Group A, Group B, etc.) and within those groups they are ordered alphabetically. The squads on the Wikipedia page we are scraping are listed in this official order.

Let's take a look at the `wikiTables` object:

<br>

```R
mode(wikiTables)
```
```
[1] "list"
```
```R
length(wikiTables)
```
```
[1] 38
```
```R
wikiTables[[1]]
```
```
   No.\n Pos.\n                    Player\n                   Date of birth (age)\n Caps\n Goals\n                   Club\n
1      1    1GK                 Erin Nayler     (1992-04-17)17 April 1992 (aged 31)     84       0           IFK Norrköping
2      2    3MF Ria Percival (vice-captain)   (1989-12-07)7 December 1989 (aged 33)    163      15        Tottenham Hotspur
3      3    2DF               Claudia Bunge (1999-09-21)21 September 1999 (aged 23)     22       0        Melbourne Victory
4      4    2DF                     CJ Bott     (1995-04-22)22 April 1995 (aged 28)     38       3           Leicester City
5      5    2DF             Michaela Foster    (1999-01-09)9 January 1999 (aged 24)      6       0       Wellington Phoenix
6      6    3MF             Malia Steinmetz   (1999-01-18)18 January 1999 (aged 24)     20       0 Western Sydney Wanderers
7      7    2DF         Ali Riley (captain)   (1987-10-30)30 October 1987 (aged 35)    154       2               Angel City
8      8    3MF             Daisy Cleverley     (1997-04-30)30 April 1997 (aged 26)     31       2                  HB Køge
9      9    4FW                 Gabi Rennie       (2001-07-07)7 July 2001 (aged 22)     26       2 Arizona State Sun Devils
10    10    3MF               Annalie Longo       (1991-07-01)1 July 1991 (aged 32)    129      15      Christchurch United
11    11    3MF               Olivia Chance    (1993-10-05)5 October 1993 (aged 29)     45       2                   Celtic
12    12    3MF               Betsy Hassett     (1990-08-04)4 August 1990 (aged 32)    145      13       Wellington Phoenix
13    13    2DF               Rebekah Stott      (1993-06-17)17 June 1993 (aged 30)     90       4   Brighton & Hove Albion
14    14    2DF                 Katie Bowen     (1994-04-15)15 April 1994 (aged 29)     94       3           Melbourne City
15    15    4FW              Paige Satchell     (1998-04-13)13 April 1998 (aged 25)     43       2       Wellington Phoenix
16    16    4FW                 Jacqui Hand  (1999-02-19)19 February 1999 (aged 24)     14       2             Åland United
17    17    4FW            Hannah Wilkinson       (1992-05-28)28 May 1992 (aged 31)    115      28           Melbourne City
18    18    4FW                  Grace Jale     (1999-04-10)10 April 1999 (aged 24)     17       2          Canberra United
19    19    2DF             Elizabeth Anton  (1998-12-12)12 December 1998 (aged 24)     19       0              Perth Glory
20    20    4FW          Indiah-Paige Riley  (2001-12-20)20 December 2001 (aged 21)      9       0            Brisbane Roar
21    21    1GK              Victoria Esson      (1991-03-06)6 March 1991 (aged 32)     16       0                  Rangers
22    22    4FW                 Milly Clegg   (2005-11-01)1 November 2005 (aged 17)      4       0       Wellington Phoenix
23    23    1GK                   Anna Leat      (2001-06-26)26 June 2001 (aged 22)     10       0              Aston Villa
```

<br>

`wikiTables` is a list of length 38. The first element of `wikiTables` is a table of the New Zealand players, which is the first table to appear on the Wikipedia page. Checking further, we can confirm that the first 32 elements of `wikiTables` are the 32 squad tables listed on the Wikipedia page in the official order described above. We note, however, that these tables are not labelled. We'll need to label them somehow.

Elements 33-38 of `wikiTables` are statistics tables included at the end of the Wikipedia page. These elements would not be of interest to us but for one reason: the 36th element of `wikiTables` is a table of the average player age by country and is arranged in FIFA's official order:

<br>

```R
wikiTables[[36]]
```
```
               Nation\n Avg. Age\n                                Oldest player\n                         Youngest player\n
1           New Zealand       26.9                 Ali Riley (35 years, 263 days)          Milly Clegg (17 years, 261 days)
2                Norway       26.4              Maren Mjelde (33 years, 256 days)    Mathilde Harviken (21 years, 203 days)
3           Philippines       24.9               Tahnai Annis (34 years, 30 days)           Kaiya Jota (17 years, 165 days)
4           Switzerland         27           Gaëlle Thalmann (37 years, 183 days)           Livia Peng (21 years, 128 days)
5             Australia       27.4                 Aivi Luik (38 years, 124 days)          Mary Fowler (20 years, 156 days)
6                Canada         27         Christine Sinclair (40 years, 38 days)         Olivia Smith (18 years, 349 days)
7               Nigeria       26.4                  Onome Ebi (40 years, 73 days)         Rofiat Imuran (19 years, 33 days)
8   Republic of Ireland       28.5               Niamh Fahey (35 years, 280 days)          Abbie Larkin (18 years, 84 days)
9            Costa Rica       25.6              Carol Sánchez (37 years, 95 days)         Sheika Scott (16 years, 271 days)
10                Japan       24.8              Saki Kumagai (32 years, 276 days)          Maika Hamano (19 years, 72 days)
11                Spain       25.3           Jennifer Hermoso (33 years, 72 days)     Salma Paralluelo (19 years, 249 days)
12               Zambia       23.8                Susan Banda (33 years, 14 days)         Esther Banda (18 years, 241 days)
13                China       26.9                 Zhang Rui (34 years, 184 days)          Pan Hongyan (18 years, 202 days)
14              Denmark       26.1 Sanne Troelsgaard Nielsen (34 years, 339 days) Kathrine Møller Kühl (20 years, 106 days)
15              England       25.8              Laura Coombs (32 years, 172 days)       Katie Robinson (20 years, 346 days)
16                Haiti       22.3         Roselord Borgella (30 years, 110 days)       Darlina Joseph (19 years, 217 days)
17          Netherlands       25.7             Sherida Spitse (33 years, 52 days)        Wieke Kaptein (17 years, 325 days)
18             Portugal         27           Carolina Mendes (35 years, 235 days)   Francisca Nazareth (20 years, 245 days)
19        United States       28.3              Megan Rapinoe (38 years, 15 days)      Alyssa Thompson (18 years, 255 days)
20              Vietnam         27       Trần Thị Thùy Trang (34 years, 346 days)           Vũ Thị Hoa (19 years, 256 days)
21               Brazil       27.5                     Marta (37 years, 151 days)               Lauren (20 years, 310 days)
22               France       26.8          Eugénie Le Sommer (34 years, 63 days)        Laurina Fazer (19 years, 280 days)
23              Jamaica       24.9           Rebecca Spencer (32 years, 148 days)     Solai Washington (17 years, 292 days)
24               Panama       24.6            Farissa Córdoba (34 years, 20 days)       Deysiré Salazar (19 years, 77 days)
25            Argentina       27.2             Vanina Correa (39 years, 340 days)         Lara Esponda (17 years, 254 days)
26                Italy       25.9          Cristiana Girelli (33 years, 88 days)       Giulia Dragoni (16 years, 255 days)
27         South Africa         27               Noko Matlou (37 years, 293 days)        Wendy Shongwe (20 years, 183 days)
28               Sweden       28.5            Caroline Seger (38 years, 123 days)         Anna Sandberg (20 years, 58 days)
29             Colombia       25.1          Sandra Sepúlveda (35 years, 139 days)      Ana María Guzmán (18 years, 39 days)
30              Germany       26.3            Marina Hegering (33 years, 94 days)           Jule Brand (20 years, 277 days)
31              Morocco       25.6                Najat Badri (35 years, 62 days)          Sarah Kassi (19 years, 314 days)
32          South Korea       28.9               Kim Jung-mi (38 years, 277 days)           Casey Phair (16 years, 21 days)
```

<br>

We can grab the team names in proper order and use them to label the tables of players later:

<br>

```R
teams <- wikiTables[[36]]$Nation
```

```
> teams
 [1] " New Zealand"         " Norway"              " Philippines"         " Switzerland"        
 [5] " Australia"           " Canada"              " Nigeria"             " Republic of Ireland"
 [9] " Costa Rica"          " Japan"               " Spain"               " Zambia"             
[13] " China"               " Denmark"             " England"             " Haiti"              
[17] " Netherlands"         " Portugal"            " United States"       " Vietnam"            
[21] " Brazil"              " France"              " Jamaica"             " Panama"             
[25] " Argentina"           " Italy"               " South Africa"        " Sweden"             
[29] " Colombia"            " Germany"             " Morocco"             " South Korea" 
```

<br>

There's a space preceeding each team name, so let's get rid of that. Also, the name "Republic of Ireland" might take up a bit too much room in plots, so we might want to abbreviate it.

<br>

```R
# Replace " " with "" in teams
teams <- sub(pattern='.',replacement='',x=teams)

# Replace "Republic" with "Rep." in teams
teams <- sub(pattern="Republic",replacement="Rep.",x=teams)
```

<br>

# Merging datasets

Now let's merge the 32 team datasets into a single table. We want to retain all of the current columns, plus add a team column.

<br>

```R
N <- length(teams) # Good idea to double-check that this is 32

# Array to store players from all teams
x <- NULL

# Loop over all the teams
for( i in 1:N )
{
  y <- wikiTables[[i]]

  # Add Team column to table
  teamVec <- rep(teams[i],nrow(y))
  y <- cbind( y, teamVec )

  # Append to array x
  x <- rbind( x, y )
}

```

<br>

`x` is now an array containing the Name, Age, etc. of all players across all teams. Let's take a look:

<br>


```R
dim(x)
```
```
[1] 736  10
```
```R
head(x)
```
```
  No.\n Pos.\n                    Player\n                   Date of birth (age)\n Caps\n Goals\n                   Club\n     teamVec
1     1    1GK                 Erin Nayler     (1992-04-17)17 April 1992 (aged 31)     84       0           IFK Norrköping New Zealand
2     2    3MF Ria Percival (vice-captain)   (1989-12-07)7 December 1989 (aged 33)    163      15        Tottenham Hotspur New Zealand
3     3    2DF               Claudia Bunge (1999-09-21)21 September 1999 (aged 23)     22       0        Melbourne Victory New Zealand
4     4    2DF                     CJ Bott     (1995-04-22)22 April 1995 (aged 28)     38       3           Leicester City New Zealand
5     5    2DF             Michaela Foster    (1999-01-09)9 January 1999 (aged 24)      6       0       Wellington Phoenix New Zealand
6     6    3MF             Malia Steinmetz   (1999-01-18)18 January 1999 (aged 24)     20       0 Western Sydney Wanderers New Zealand
Browse[1]> tail(x)
    No.\n Pos.\n             Player\n                  Date of birth (age)\n Caps\n Goals\n                      Club\n     teamVec
731    18    1GK          Kim Jung-mi  (1984-10-16)16 October 1984 (aged 38)    135       0       Incheon Hyundai Steel South Korea
732    19    4FW          Casey Phair     (2007-06-29)29 June 2007 (aged 16)      0       0 Players Development Academy South Korea
733    20    2DF Kim Hye-ri (captain)     (1990-06-25)25 June 1990 (aged 33)    111       1       Incheon Hyundai Steel South Korea
734    21    1GK           Ryu Ji-soo (1997-09-03)3 September 1997 (aged 25)      0       0                   Seoul WFC South Korea
735    22    3MF           Bae Ye-bin  (2004-12-07)7 December 2004 (aged 18)      2       0            Uiduk University South Korea
736    23    4FW        Kang Chae-rim    (1998-03-23)23 March 1998 (aged 25)     24       6       Incheon Hyundai Steel South Korea
```

`x` has 23 * 32 = 736 rows, which is reassuring. However, the column names all have extraneous characters - let's delete those. Let's also rename the date of birth column to avoid having spaces in column names.

<br>

```R
cnames <- colnames(wikiTables[[1]])
cnames <- substr( cnames, 1, nchar(cnames)-1 )  # remove last two characters
cnames[4] <- "DOB"

colnames(x) <- c(cnames,"Team")
```

<br>

We want the ages of each player, which could be as simple as grabbing the last second or third characters from each date of birth entry. However, we cannot be sure of when exactly these ages were calculated. Ideally they would be calculated on the starting date of the tourament (July 20, 2023), so let's recalculate the ages ourselves. To avoid having the deal with leap years and other annoying edge cases, we can use the `age_calc()` function in the `eeptools` package to calculate the length of time between each player's date of birth and the starting date of the tournament.

We can easily modify every row of this column using the `mutate()` function within `dplyr`. While we're mutating, let's also convert all columns that should be numeric to numeric - the data scraping process created these as character objects.

<br>

```R
# Kick off date of the tournament
kickOffDate <- as.Date("2023-07-20")

library(dplyr)
library(eeptools)
x <- mutate( x,
             DOB=gsub("\\).*","",DOB),          # delete everything after closing paranthesis
             DOB=substr( DOB, 2, nchar(DOB) ),  # delete opening paranthesis
             DOB=as.Date(DOB),
             Age=age_calc(dob=DOB,enddate=kickOffDate,unit="years"),
             AgeFloor=floor(Age),
             Caps=as.numeric(Caps),
             Goals=as.numeric(Goals) )  
```

<br>


`dplyr` has a many other useful functions for manipulating data, which can be demonstrated by calculating the average age by team. We use the `group_by()` function in `dplyr` to "group" the data according to teams, i.e., the subsequent `summarize()` function will by applied to teams individually. Finally we use the `arrange()` and `desc()` functions to arrange the teams as a descending function of average age. We use the pipe operator `%>%` to string consecutive functions together, i.e., the output of a function is passed as the first argument to the subsequent function.

<br>

```R
avgAgeTable <- x %>%
               group_by( Team ) %>%
               summarize( avgAge=mean(Age) ) %>%
               arrange( desc(avgAge) )

print.data.frame(avgAgeTable)
```

```
              Team   avgAge
1      South Korea 29.38551
2  Rep. of Ireland 28.97519
3           Sweden 28.94628
4    United States 28.61482
5           Brazil 27.96971
6        Australia 27.89739
7        Argentina 27.69456
8     South Africa 27.55257
9          Vietnam 27.51609
10     Switzerland 27.49386
11        Portugal 27.44853
12           China 27.44172
13          Canada 27.41395
14     New Zealand 27.28457
15          France 27.21048
16         Nigeria 26.82680
17          Norway 26.82454
18         Germany 26.78976
19         Denmark 26.59683
20        Colombia 26.59666
21         England 26.39725
22           Italy 26.33088
23     Netherlands 26.23169
24      Costa Rica 26.12517
25         Morocco 26.06825
26           Spain 25.72335
27         Jamaica 25.44887
28     Philippines 25.33018
29           Japan 25.30414
30          Panama 25.07119
31          Zambia 23.86139
32           Haiti 22.90572
```

<br>

Another essential `dplyr` function is `filter()`, which returns rows that fulfill one or more conditions. We demonstrate `filter()` by checking to see whether the Goal column is nonzero for any of the goalkeepers. It's not impossible for a goalkeeper to have scored but it's an easy way of checking the data - if many goalkeepers have goals or if one keeper has many goals, that may be a red flag that an error has occured when reading-in the data.

<br>

```R
# Return list of goalkeepers
xGK <- x %>%
       filter( Pos.=="1GK",      # First condition  - return only goalkeepers
               !is.na(Goals) )   # Second condition - ignore missing values

sum(xGK$Goals)
```
```
[1] 0
```

<br>

There are goalkeeper goals in our dataset - this is reassuring that we read-in the data correctly.

<br>

# Analyzing the data

Now that we've read-in the data we can see how complete the individual data attributes are. To do this, we check for NAs in the Age, Caps and Goals column:

<br>

```R
nMissing <- x %>%
            group_by(Team) %>%
            summarize(Age   = sum(is.na(Age)),
                      Caps  = sum(is.na(Caps)),
                      Goals = sum(is.na(Goals)))
```

```
> print.data.frame(nMissing)
              Team Age Caps Goals
1        Argentina   0   23    23
2        Australia   0    0     0
3           Brazil   0    0     0
4           Canada   0    0     0
5            China   0    0     0
6         Colombia   0    0     0
7       Costa Rica   0   23    23
8          Denmark   0    0     0
9          England   0    0     0
10          France   0    0     0
11         Germany   0    0     0
12           Haiti   0    0     0
13           Italy   0    0     0
14         Jamaica   0    0     0
15           Japan   0    0     0
16         Morocco   0   23    23
17     Netherlands   0    0     0
18     New Zealand   0    0     0
19         Nigeria   0    1     1
20          Norway   0    0     0
21          Panama   0    0     0
22     Philippines   0    0     0
23        Portugal   0    0     0
24 Rep. of Ireland   0    0     0
25    South Africa   0   23    23
26     South Korea   0    0     0
27           Spain   0    0     0
28          Sweden   0    0     0
29     Switzerland   0    0     0
30   United States   0    0     0
31         Vietnam   0    0     0
32          Zambia   0   23    23
```

<br>

Cap and Goal tallies are missing for five teams: Argentina, Costa Rica, Morocco, and South Africa. They are also missing for one Nigeria player.
Missing caps and goals for five teams is not ideal, so let's focus on analyzing the ages for now.

<br>



# Visualizing the age data

Let's visualize the distribution of ages within each team on a single plot. We'll need a grid of plots with 1 plot for each of the 32 teams, so a 4x8 plot is probably the way to go.

<br>

```R
# Create a file to save the plot to
png( file="WwcSquadsByAge.png", res=300, height=8.5, width=9.5, units="in" )

# Make a 4x8 grid of plots
par( mfrow=c(4,8), mar=c(1,0,1,0), oma=c(0,5,5,1), xpd=TRUE )
```

<br>

We loop over the teams, ordered from higher mean age to lowest, and plot the distribution of ages for each team. The y-axis will correspond to age. All players of a given age will then be plotted symmetrically about the y-axis of each team plot.

<br>

```R
bgs <- c("black","blue","yellow","red")

ageRange <- range(x$Age)

# Loop over all teams
for( i in 1:N )
{
  # Get team - these are arranged from highest mean age to lowest
  team <- avgAgeTable$Team[i]

  # Get players from this team
  x_i <- filter( x, Team==team )

  # Calculate number of players at each age
  y <- group_by(x_i,AgeFloor) %>%
       summarize(n=length(AgeFloor))

  # We should also note which players are captains
  # There may be a captain or two co-captains, so maybe vector
  # of length 2 to store ages of possible captains
  capAge <- rep(-1,2)
  
  # Get the captains
  cap <- grepl("(captain)",x_i$Player,fixed=TRUE) | 
         grepl("(co-captain)",x_i$Player,fixed=TRUE)

  # If there are captains, get their ages
  if( any(cap) )
    capAge[1:sum(cap)] <- x_i[cap,"AgeFloor"]

  # Blank plot
  plot( x=c(-6,6), y=ageRange+c(-1,1), type="n", axes=FALSE )
  
  # Green plot background
  rect( xleft = par("usr")[1],
        ybottom = par("usr")[3],
        xright = par("usr")[2],
        ytop = par("usr")[4],
        col = "chartreuse3" )

  # Title the plot with the team name
  title( main=team, cex.main=1.3, line=0.2 )

  # Thick horizontal at average age
  abline( h=avgAge[i,"avgAge"], col="white", lwd=2 )
  
  # Thin grid lines
  abline( h=seq(from=0,to=ageRange[2],by=5), col="lightgreen", lty=1 )

  # Draw box around plot
  box()
  
  # Plot y-axis if plotting on the left edge of the grid
  if( i %% 8 == 1 )
    axis( side=2, las=1 )

  # Loop over ages
  for( j in 1:nrow(y) )
  {
    # Get number of players at this age
    yn <- unlist(y[j,"n"])
    
    # Lay out the players horizontally
    if( yn == 1 )
      xpos <- 0
    else if( yn %% 2 == 0 )
      xpos <- seq(from=-yn/2,to=yn/2,length.out=yn)
    else
      xpos <- seq(from=floor(-yn/2),to=ceiling(yn/2),length.out=yn)

    # All players are black except captains who are red
    pchs <- rep(21,yn)
    if( y[j,"AgeFloor"] %in% capAge )
      if( capAge[1]==capAge[2] )
        pchs[1:2] <- 24
      else
        pchs[1] <- 24

    # Get position code for each player at this age
    pos <- filter(x_i,AgeFloor==unlist(y[j,"AgeFloor"]))$Pos. %>%
           substr(start=1,stop=1) %>%
           as.numeric()

    # Plot players
    points( x=xpos, y=rep(y[j,"AgeFloor"],yn), bg=bgs[pos],
            pch=pchs, lwd=1.5, cex=1.1 )

  } # next age

  # Plot position legend above plot (1,7)
  if( i == 7 )
  {
    par(xpd=NA)    # Allow plotting in margins
    legend( x="topleft", inset=c(0,-0.32), legend=allPos,
            pch=21, pt.bg=bgs, horiz=TRUE, pt.lwd=1.5 )
    par(xpd=FALSE) # Turn off plotting in margins
  }

  # Plot captain legend above plot (1,8)
  if( i == 8 )     # Allow plotting in margins
  {
    par(xpd=NA)
    legend( x="topright", inset=c(0,-0.32), legend="Captain",
            pch=24, bg="white", pt.lwd=1.5 )
    par(xpd=FALSE) # Turn off plotting in margins
  }

} # next team

# Axis label and plot title
mtext( side=2, text="Age (yr)", outer=TRUE, line=3 )
par(font=2)
mtext( side=3, text="2023 FIFA Women's World Cup Squads", outer=TRUE, line=1.2, cex=1.3 )
par(font=1)

# Turn off plotting device
dev.off()
```

![AgesByTeam](/images/post1/ageDistributionByTeam.png)


## Analysis

Some interesting observations become apparent upon viewing the chart.

South Korea and Haiti offer some important points of contrast. Not only are South Korea and Haiti the oldest and youngest squads on average, but South Korea's squad is quite evenly distributed - there are no more than two players at any age and there are just three players younger than 25 (though this does include the youngest player in the tournament, aged 16). In contrast, all of Haiti's players except for 1 are between 19 and 26 years old, including 6 players aged 19 and 4 players aged 20.

The Republic of Ireland has an older, tightly-clustered squad - 61% players between 26 and 30. The Irish squad has only three players younger than 26.

Expectedly, captains tend to be among the most senior squad players, though the captains of both Republic of Ireland and Zambia are younger than the average squad age.

In the men's game a premium is often placed on experience for goalkeepers and defenders, while younger players may be more likely to feature in midfield or attacking roles due to the increased mobility needed for these roles. There are some minor team-specific patterns to note from the plot. For exmaples, several teams feature two veteran goalies, (e.g., South Korea, Nigeria, Morocco, Panama).
It is difficult to discern any clear age patterns from this plot. However, we note that these age distributions may be skewed by presence of players who will play few if any minutes at the tournament. Re-plotting the age distributions after the tournament with such marginal players excluded may reveal more discernable patterns.

Some other anomalies:

  -The United States' youngest defender and midfield are 23 and 24 years old respectively. Only three players on the US squad are younger than 23 and all of those players are forwards.

  -China and Spain have relatively young keepers. China's oldest GK is 25, while all of Spain's keepers are between 21 and 23.


## Next steps

It would be nice to make this plot interactive so that hovering over the individual points triggers a display of player-specific information. I'll attempt to construct such an interactive plot in Shiny or Tableau for one or more of the men's European club leagues when they kick off next month.


