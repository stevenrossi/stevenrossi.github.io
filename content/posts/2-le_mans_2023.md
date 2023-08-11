---
title: Visualizing the 2023 24 Hours of Le Mans results in R using website scraping and regular expressions
date: 2023-07-24T23:12:51-07:00
draft: true
tags: [R, Motorsports, Data Scraping, Visualization]
---

The 2023 24 hours of Le Mans was an historic edition of the legendary endurance race, not only because it marked the 100 year anniversary from the first running, but also because it featured the return of a number of hallowed car manufacturers to the top flight of sports car racing. In this project I aim to make a basic visualization of the distance travelled by each car in the 2023 edition of Le Mans. In doing so, my real goal was not to make any specific insight into the race result but was to see if I could scrape images of all of the cars from an online source and use these in the visualization.

# Background on the 24 Hours of Le Mans

## History of Le Mans

The [24 Hours of Le Mans](https://en.wikipedia.org/wiki/24_Hours_of_Le_Mans) (or, 24 Heures du Mans) is one of the most prestigious races in motorsport and the oldest active endurance race in the world, first being run in 1923. The race, which is won by the car the covers the greatest distance in 24 hours, is run at the Circuit de la Sarthe at Le Mans, Sarthe, France, which is an amalgamation of public roads and private, race-specific segments that measures 13.626 km in length in its present configuration. The race is currently the centerpiece of the World Endurance Championship (WEC), which is a worldwide series of endurance races, each lasting at least six hours.

In motorosports, "sport cars" are closed-wheel, two-seater cars that are either prototypes (i.e., purpose-built for racing and not modified from a production car) or based on high-performance production vehicles. This is contrast to "formula" racing, which employs strictly purpose-buit, open-wheel, single-seaters, and to either "stock car" or "touring car" racing, which are both generally based on common sedans or hatchbacks. Sports car races usually feature multiple classes of car racing simultaneously (i.e., both prototypes and production-based cars), with prototype classes being the "top" or fastest classes.

Bugatti, Bentley, and Alfa Romeo were each multiple-time winners of the initial Le Mans races held in the interwar period, with Ferrari and Jaguar emerging as the top manufacters of the postwar era. Ferraris took 6 consecutive races between 1960 and 1965, only to be knocked off their perch by the unlimited bankroll of Ford Motor Company and the ingenuity of Carroll Shelby. (The story of the legendary 1966 race, in which Bruce Mclaren and Chris Amon drove the Ford GT40 Mk. II to victory in a 1-2-3 finish for Ford, was recently told in the film [*Ford v Ferrari*](https://www.imdb.com/title/tt1950186/)). After failing to win back their Le Man crown in the following years, Ferrari withdrew from sportscar racing to focus soley on Formula 1. Privateers would continue to run their own Ferraris at Le Mans, but Ferrari would not officially back another top-level prototype entry at Le Mans for the next five decades.

Porsche were regular Le Mans winners from 1970 through the late 1980s, and a number of privateer teams (i.e., teams not backed by a manufacturer) also excelled. The late 80s and 90s marked an intensely competitive period, with Sauber/Mercedes, Jaguar, Mazda, Peugeot, Porsche, McLaren, and BMW each taking overall victories.

Many manufacturers withdraw from sports car racing in the new millenium due to high costs, ushering in a new era of largely uncompetitive prototype racing. Audi won 13 races between 2000 and 2014, while Porsche won each Le Mans between 2015 and 2017. Toyota were consecutive winners from 2018 to 2022, facing little competition in each race. Indeed, from 2019 to 2022, Toyota's only challengers in the top prototype class were privateer teams running on comparatively miniscule budgets or by cars that had been grandfathered into the prototype class under previous rulesets and posed no serious threat to Toyota.

## The 2023 World Endurance Championship season

New regulations were recently introduced that were aimed at reducing the cost of fielding a top-class prototype (now termed "hypercars"), prompting many manufacturers to establish new sports car programs. Peugout, Ferrari, Porsche, and Cadillac announced entries for the 2023 WEC season while Lamborghini, BMW,  and Alpine are slated to join in 2024. Would any of these new teams be able to take down the formidable Toyota Gazoo Racing?

Le Mans was the fourth race on the 2024 WEC calendar, following the 1000 Miles of Sebring in Florida, the 6 hours of Portimão in the Algarve region of Portugal, and the 6 Hours of Spa-Francorchamps at the historic Circuit de Spa-Francorchamps in Belgium. Ferrari's Antonio Fuoco pulled off a shock in qualification for the opening race at Sebring, putting the #50 Ferrari on pole, though both of Toyota's cars (#7 and #8) passed the Ferrari early in the actual race and Toyota cruised to a comfortable victory.

I was lucky enough to attend the second race of the season at Portimão. This time, technical issues hampered the #7 Toyota, but the #8 Toyota was able to easily cruise to victory. The Ferraris looked comfortably quicker than the rest of the field and, as a Ferrari fan, I was happy to see the #50 take second place, but closing the gap to Toyota still felt a very long way off. (As an aside, I will mention that Cadillac's V8 engine sounds so impressive in person - you always know when the Cadillac is driving past you even if you're watching some other part of the race track. Unfortunately most of the other hypercars run hybrid V6 configurations, which don't sound nearly as imposing.)

<table style="width: 100%">
<colgroup>
 <col span="1" style="width: 85%;">
 <col span="1" style="width: 15%;">
</colgroup>
<tbody>
<tr>
<td><img src="/images/post2/pitlane-prema.jpeg" alt=""/></td>
<td><i>The unbiased author enjoys a stroll down pitlane at the 6 Hours of Portimão.</i></td>
</tr>
</tbody>
</table>


<table style="width: 100%">
<colgroup>
 <col span="1" style="width: 50%;">
 <col span="1" style="width: 50%;">
</colgroup>
<tbody>
<tr>
<td><img src="/images/post2/toyota_8_portugal_paddock.jpg"  alt=""/></td>
<td><i>The author at the 2023 6 Hours of Portimão paddock with Ryō Hirakawa and Sébastien Buemi of the #8 Toyota. Buemi won Le Mans the 2014, 2018, 2019, and 2022, was Formula E champion in 2015-16 and competed in 55 Formula 1 races for Red Bull junior team Toro Rosso. Hirakawa won the 2022 Le Mans race alongside Buemi.</i> </td>
</tr>
</tbody>
</table>

<table style="width: 100%">
<colgroup>
 <col span="1" style="width: 50%;">
 <col span="1" style="width: 50%;">
</colgroup>
<tbody>
<tr>
<td><img src="/images/post2/giovinazzi.jpg" alt=""/></td>
<td><i>The author at the 2023 6 Hours of Portimão paddock with Antonio Giovinazzi of the #51 Ferrari and eventual winner of the 2023 edition of Le Mans. Giovinazzi completed in 61 Formula 1 races between 2017 and 2021 for Sauber/Alfa Romeo.</i></td>
</tr>
</tbody>
</table>

<table style="width: 100%">
<colgroup>
 <col span="1" style="width: 50%;">
 <col span="1" style="width: 50%;">
</colgroup>
<tbody>
<tr>
<td><img src="/images/post2/ferrari_51_in_garage.jpg" alt=""/></td>
<td><i>The #51 receives some final attention in the pits before going out for the 6 Hours of Portimão.</i></td>
</tr>
</tbody>
</table>


<table style="width: 100%">
<colgroup>
 <col span="1" style="width: 80%;">
 <col span="1" style="width: 20%;">
</colgroup>
<tbody>
<tr>
<td><img src="/images/post2/turn-1-portimao.jpeg" alt="" height="400"/></td>
<td><i>The #51 Ferrari passes the United Autosports LMP2 car on the inside going into turn 1 at the Autódromo Internacional do Algarve during Free Practice 3 of the 6 Hours of Portimão.</i></td>
</tr>
</tbody>
</table>



<br>

Toyota took pole in the third race of the season at Spa and went on to finish the race 1-2. The #51 Ferrari took third place, a minute and nine seconds behind the winning Toyota. Toyota's GR010 Hybrid looked unbeatable going into the climax of the season.

Ferrari started the Le Mans weekend in ideal fashion, taking a 1-2 in qualifying while Toyota could only manage positions 3 and 5. However, the #8 Toyota took the lead from the #50 Ferrari on the first lap of the race and stayed comfortably ahead for the first stint until rain and crashes mixed up the field. An [overnight crash](https://www.youtube.com/watch?v=SzHBUuXrU5M) put the #7 Toyota out of the race, while the #8 Toyota battled with the #50 Ferrari for the lead, though the latter car was forced to pit with a leak and lost six laps. The #51 Ferrari and #8 Toyota would go on to fight for the lead, with the #8 eventually [spinning out](https://www.youtube.com/watch?v=PDy82c2jeew) and taking damage with less than two hours to go, handing Ferrari a historic and unexpected victory on its return to Le Mans.

![PostRace](/images/post2/ferrari_post_race.png)

It was a great race. I recommend watching it if you have [24 hours to spare](https://www.youtube.com/watch?v=ciSNHrKrkeQ).



# Visualizing the results of the 2023 race


## Scraping website for car images

The official WEC website hosts a [page](https://www.fiawec.com/en/race/result/4792) with results from the 2023 race at Le Mans with small pictures of each of the 62 cars - these would be time consuming to save one-by-one, so let's see if we can scrape them. I copied the source code for this page and dumped it in a text file called `le-mans-website.txt`.

On the official website, if we check the URL of a car image, we see something like this:

<br>

```
https://storage.googleapis.com/ecm-prod/media/cache/entry_car/assets/1/engage/72555/2023-lm-51-ferrari-droite_46c6eb.png
```

<br>

We note that the car images are PNGs, and that the image is stored in a directory called "entry_car" - we can probably use this information to distinguish the car images from other images on the page. For instance, there is also a tire manufacturer logo for each car, with URL as follows:

<br>

```
https://storage.googleapis.com/ecm-prod/media/cache/tyres/assets/Tyre/Logo_Michelin-1819_5acca3_d0741b.png
```

<br>

We can see that this image is stored in a "tyres" folder, not the "entry_car" folder. So, let's grab every line of source code that displays a PNG image from the "entry_car" directory:

<br>

```R
sourceCode <- readLines("le-mans-website.txt")

rows <- grepl( "png", sourceCode ) &
        grepl( "<img src", sourceCode ) &
        grepl( "entry_car", sourceCode )

sourceCode <- sourceCode[rows, ]
```

<br>

Now let's extract the URL from each from. A typical row looks like this:

<br>

```R
sourceCode[1]
```
```
[1] "                    <img src=\"https://storage.googleapis.com/ecm-prod/media/cache/entry_car/assets/1/engage/72555/2023-lm-51-ferrari-droite_46c6eb.png\" alt=\"Ferrari 499P\""
```

<br>

We want to extract everything between `https://` and `.png`, which we can do using the `gsub()`:

<br>

```R
pngURL <- gsub(".*<img src=\"|\".*","",sourceCode)
```
<br>

The first argument of `gsub()` is the [regular expression](https://en.wikipedia.org/wiki/Regular_expression) to match, the second argument is a string to replace the matched text with, and the third argument is the string `x`where matches are sought. We pass the argument `".*<img src=\"\\s*|\".*"` as the first argument. The initial portion of this argument, `.*<img src="` selects all of the characters up to and including `<img src=`, while the final portion, `\".*`, selects all of the charcters from `\"` onward. The second argument is a blank string, which effectively deletes all parts of `x` that match the first argument.

If we inspect pngURL, we see that it contains 124 elements - two identical entries for each car. We can delete duplicate entries using the `unique()` function:

<br>

```R
pngURL <- unique(pngURL)

# Number of cars - should be 62
N <- length(pngURL)
```
<br>

Now we've got a list of URLs that we can download the images from, but we'll need to give each image a name. The simplest thing to do is to name each image using each car's unique number. Let's check out a bunch of URLs and see how we can do this.

<br>

```R
pngURL[1:20]
```
```
 [1] "https://storage.googleapis.com/ecm-prod/media/cache/entry_car/assets/1/engage/72555/2023-lm-51-ferrari-droite_46c6eb.png"     
 [2] "https://storage.googleapis.com/ecm-prod/media/cache/entry_car/assets/1/engage/72936/2023-lm-toyota-gr010-8-droite_b985c4.png" 
 [3] "https://storage.googleapis.com/ecm-prod/media/cache/entry_car/assets/1/engage/72546/2023-lm-2-cadillac-droite_db12c3.png"     
 [4] "https://storage.googleapis.com/ecm-prod/media/cache/entry_car/assets/1/engage/72547/2023-lm-3-cadillac-droite_6c6dc0.png"     
 [5] "https://storage.googleapis.com/ecm-prod/media/cache/entry_car/assets/1/engage/72554/2023-lm-50-ferrari-droite_256646.png"     
 [6] "https://storage.googleapis.com/ecm-prod/media/cache/entry_car/assets/1/engage/72560/2023-lm-glickenhaus-708-droite_6646c6.png"
 [7] "https://storage.googleapis.com/ecm-prod/media/cache/entry_car/assets/1/engage/72561/2023-lm-glickenhaus-709-droite_f3a4c3.png"
 [8] "https://storage.googleapis.com/ecm-prod/media/cache/entry_car/assets/1/engage/72557/2023-lm-93-peugeot-9x8-droite_46c6ee.png" 
 [9] "https://storage.googleapis.com/ecm-prod/media/cache/entry_car/assets/1/engage/72572/2023-lm-34-oreca-07-droite_31bc23.png"    
[10] "https://storage.googleapis.com/ecm-prod/media/cache/entry_car/assets/1/engage/72938/2023-lm-41-oreca-07-droite_519647.png"    
[11] "https://storage.googleapis.com/ecm-prod/media/cache/entry_car/assets/1/engage/72569/2023-lm-30-oreca-07-droite_0093c9.png"    
[12] "https://storage.googleapis.com/ecm-prod/media/cache/entry_car/assets/1/engage/72574/2023-lm-36-oreca-07-droite_646c70.png"    
[13] "https://storage.googleapis.com/ecm-prod/media/cache/entry_car/assets/1/engage/72937/2023-lm-31-oreca-07-droite_b98809.png"    
[14] "https://storage.googleapis.com/ecm-prod/media/cache/entry_car/assets/1/engage/73532/2023-lm-48-oreca-07-droite_216481.png"    
[15] "https://storage.googleapis.com/ecm-prod/media/cache/entry_car/assets/1/engage/72563/2023-lm-10-oreca-07-droite_f80700.png"    
```

<br>

It looks like most cars have a filename starting with `2023-lm-`, followed by the car number, followed by `-` and the manufacturer's name. The Toyota and Glickenhaus cars follow different naming conventions, but we can fix those seperately.

Using similar logic to before, we delete everything up to and including `lm-` and everything from the next `-` onward. 

<br>

```R
  carNumber <- gsub(".*lm-|-.*","",pngURL)
```
```
> carNumber
 [1] "51"          "toyota"      "2"           "3"           "50"          "glickenhaus" "glickenhaus"
 [8] "93"          "34"          "41"          "30"          "36"          "31"          "48"         
[15] "10"          "5"           "311"         "23"          "35"          "45"          "22"         
[22] "6"           "37"          "28"          "65"          "33"          "94"          "25"         
[29] "86"          "85"          "54"          "43"          "98"          "9"           "56"         
[36] "100"         "39"          "74"          "24"          "38"          "57"          "911"        
[43] "80"          "88"          "4"           "777"         "47"          "77"          "32"         
[50] "63"          "toyota"      "66"          "923"         "75"          "72"          "83"         
[57] "60"          "16"          "55"          "21"          "13"          "14"    
```
<br>

It worked for every car except Toyota and Glickenhaus. Let's clean those entries up now:

<br>

```R
  t <- carNumber=="toyota"
  carNumber[t] <- gsub(".*gr010-|-.*","",pngURL[t])
  
  g <- carNumber=="glickenhaus"
  carNumber[g] <- gsub(".*glickenhaus-|-.*","",pngURL[g])
```

<br>

Now let's loop over the list URLs and download each file:

<br>


```R
  # Create directory to store images
  dir.create("images")

  # Download images
  for( i in 1:N )
  {
    # File to save image to
    destfile <- paste0("images/",carNumber[i],".png")

    # Download image to destfile
    download.file( url=pngURL[i],
                   destfile=destfile) )
  }
```

<br>

We can check the contents of the images directory to see if the files downloaded:

<br>

```R
list.files("images")
```
```
 [1] "10.png"  "100.png" "13.png"  "14.png"  "16.png"  "2.png"   "21.png"  "22.png"  "23.png"  "24.png" 
[11] "25.png"  "28.png"  "3.png"   "30.png"  "31.png"  "311.png" "32.png"  "33.png"  "34.png"  "35.png" 
[21] "36.png"  "37.png"  "38.png"  "39.png"  "4.png"   "41.png"  "43.png"  "45.png"  "47.png"  "48.png" 
[31] "5.png"   "50.png"  "51.png"  "54.png"  "55.png"  "56.png"  "57.png"  "6.png"   "60.png"  "63.png" 
[41] "65.png"  "66.png"  "7.png"   "708.png" "709.png" "72.png"  "74.png"  "75.png"  "77.png"  "777.png"
[51] "8.png"   "80.png"  "83.png"  "85.png"  "86.png"  "88.png"  "9.png"   "911.png" "923.png" "93.png" 
[61] "94.png"  "98.png"
```


## Retrieving results

Scraping the results from the WEC website source could would be a pain, so let's read the results in from Wikipedia, which is easier to deal with. We can first use the `GET()` function in the `httr` package can be used to scrape websites for information, then use the `readHTMLTable()` function in the `XML` package to read-in the tables from the scraped website:

<br>

```R
wiki <- GET("https://en.wikipedia.org/wiki/2023_24_Hours_of_Le_Mans")

wikiTables <- readHTMLTable( doc=content(wiki,"text"),
                             header=TRUE )
```

<br>

The 2023 edition of Le Mans featured four classes of cars:

  - Hypercar - purpose-built prototypes developed by each team
  - LMP2 - purpose-built prototypes constructed by each team from a list of approved parts from specific manufacturers
  - GTE-AM - modified, production-based *grand tourers*
  - Innovative - a special class for experimental concepts. This year's Innovative entry was a NASCAR that was specially modified to handle the Le Mans circuit.


<br>

```R
resultTable <- wikiTables[[5]]
colnames(resultTable) <- resultTable[1, ]


# Length of Circuit de la Sarthe (km)
circuitLengthKM <- 13.626

resultTable <- filter( resultTable,
                       Pos!="Pos", # Remove first row
                       !is.na(No.) ) %>%
               mutate( Distance=as.numeric(Laps)*circuitLengthKM,
                       Class=if_else(Class=="LMP2 (Pro-Am)","LMP2",Class) )

```

<br>

Only one car finished on the lead lap with the winning #51 Ferrari was the #8 Toyota. At the time the #51 was crossing the finish line, the #8 Toyota was 1 minute and XXX behind. Let's subtract this portion of the lap that was unfished when the Ferrari took the chequered flag from the Toyota's overall distance travelled.

<br>

```R
# Time differential between #8 Toyota and winning #51 Ferrari
timeDiff <- resultTable[2,"Time/Reason"]

# Extract the number of minutes from the timeDiff string
mins <- str_match( timeDiff, "\\+\\s*(.*?)\\:" )[2] %>%
        as.numeric()

# Extract the number of seconds from the timeDiff string
# and add the number of minutes
secDiff <- sub(".*:","",timeDiff) %>%
           as.numeric() + 60*mins

# Source: The B Pillar (https://mcusercontent.com/58ec14b624aac0d4f32a73311/files/1a937598-5795-63eb-6c7b-4a57844916e4/TBP_Report___2023_WEC_R04_Le_Mans.pdf)
# Brandon Hartley - 3:30.475
# Ryo Hirawkawa - 3:31.753
# Sebastian Buemi - 3:31.015
avgLapTimeSecs <- mean( c(30.475+3*60,
                          31.753+3*60,
                          31.015+3*60) )

# Proportion of lap remaining when #51 Ferrari took chequered flag
pLapRemaining <- secs/avgLapTimeSecs

# Subtract lap distance remaining from distance travelled
resultTable[2,"Distance"] <- resultTable[2,"Distance"] - pLapRemaining*circuitLengthKM

```

<br>

## Plotting results

We'll want to differentiate the cars by classes, so let's assign each class a unique colour. The `brewer.pal()` function in the `RColorBrewer` package is useful for generating aesthetically pleasing colour palettes:

<br>

```R
class <- unique(resultTable$Class)

library(RColorBrewer)
cols <- brewer.pal(length(class),"Set1")
```

<br>

Let's set up a plot so that the x-axis is distance travelled and the y-axis represents finishing position, with first place at the top and last place at the bottom.

<br>

```R
png( file="le-mans-2023-finishing-positions.png",
     res=300,
     height=12,
     width=7,
     units="in" )

# We'll need a maximum distance travelled to bound the
# x-axis, so round furthest distance travelled to next
# 1000th using round_any() in plyr package
library(plyr)
maxX <- round_any(max(resultTable$Distance),1000,f=ceiling)

plot( x    = c(0,maxX),
      y    = c(0,N+1),
      type = "n",       # Empty plot
      axes = FALSE,
      xaxs = "i",
      yaxs = "i",
      xlab = "Distance travelled (km)",
      ylab = "Finishing position",
      main = "91st 24 Hours of Le Mans (2023)" )
```

<br>

Add vertical grid lines to improve legibility and add a horizontal line to seperate the cars that finished the race from the cars that did not. The "Pos" or position column of the `resultTable` is labelled DNF for any cars that did not finish.

<br>

```R
abline( v=seq(500,maxX-500,by=1000), col="grey80", lty=3 )
abline( v=seq(1000,maxX,by=1000), col="grey60", lty=3 )

# Number of cars that finished the race
nFinishers <- sum(resultTable$Pos!="DNF")

# DNF line
abline( h=N-nFinishers+0.5, lty=2, lwd=2, col="orange" )
```

<br>

Now let's loop over all the cars and plot the car image and a horizontal line indicating the distance travelled. We want the nose of the car to indicate the distance travelled, with a horizontal line trailing behind, coloured according to class.

<br>

```R
for( i in 1:N )
{
  # Read-in the car image
  carFilename <- paste0("images/",resultTable[i,"No."],".png")
  carObj      <- readPNG(carFilename)

  # Distance travelled by car i
  dist <- resultTable[i,"Distance"]

  # Class of car i
  classi <- which( class %in% resultTable[i,"Class"] )

  # Horizontal line trailing car i
  segments( x0=0,
            x1=dist-300,
            y0=N-i+1,
            lwd=11,
            col=cols[classi] )

  # Write car info in the trailing horizontal line
  # If statement prevents this from happening for cars that
  # DNF'd early since there is no space to write
  if( dist>500 )
  {
    label <- paste(resultTable[i,"No."],resultTable[i,"Team"],resultTable[i,"Chassis"],sep="-")
    text( x      = 0,
          y      = N-i+0.95,
          labels = label,
          pos    = 4,
          col    = "white",
          cex    = 0.6 )
  }

  # Plot the car image
  addImg( obj=carObj,
          x=dist-250,
          y=N-i+1.1,
          width=500 )
}
```

<br>

Finally, we can add the axes, legends, and labels:

<br>

```R
# X-axis
axis( side=1, at=seq(0,maxX,by=1000) )

# Y-axis - label in descending order
axis( side   = 2,
      at     = c(N,seq(N-4,1,by=-5),1),
      labels = c(1,seq(5,N,by=5),N),
      las    = 1 )

# Legend for car calsses
legend( x      = "bottomright",
        lwd    = 11,
        col    = cols,
        legend = class, 
        bty    = "n" )

# Label the DNF line
par(font=2)
text( x      = 0.9*maxX,
      y      = N-nFinishers-1.5,
      cex    = 1.2,
      labels = "DNF",
      col    = "orange" )
par(font=1)

# Turn off plotting device
dev.off()
```

<br>

![PostRace](/images/post2/le-mans-2023-finishing-positions.png)


Not bad! Again, there is not a ton of insight to gain from this plot, but it does look pretty cool. The anomaly that jumps out are the finishes around the DNF line - the 41st and 42nd placed finishers appear to have covered a greater distance than the 40th placed car. This occured because the #57 Kessel Racing Ferrari and the #911 Proton Competition Porsche (41st and 42nd place finishers, respectively), retired from the race due to damage, while the #38 Hertz Team Jota Porsche (39th place) was still running at the conclusion of the race despite spending an appreciable amount of time in the garage.



# Next steps

I'd like to apply the apply the table-scraping script to Wikipedia pages for previous Le Mans races and build up a dataset of finishes over the years. I'd also like to do more detailed analyses of endurance car races, though the data is not as readily available as for, say, Formula 1.






