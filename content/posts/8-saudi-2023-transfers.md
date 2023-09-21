---
title: 'The 2023 Saudi Football Raid, Visualized'
date: 2023-09-07T01:01:01-07:00
draft: false
tags: [Football, R, ggplot, dplyr]
header-includes:
    - \usepackage{setspace}
---

One of the biggest stories in football this year has been the rise of the Saudi Professional League (SPL). The SPL was widely expected to pursue more of the game's major stars following Cristiano Ronaldo's signing for the Riyadh-based Al-Nassr FC at the end of last year, however, most observers have been caught off guard by the aggressiveness of Saudi clubs in this summer's transfer window. A flurry of transfers was topped off by a €90 million transfer of Neymar from PSG to the Riyadh-based Al-Hilal, which was itself nearly surpassed by in the closing days of the window when the Jeddah-based Al-Ittihad made an unsuccessful late move for Liverpool's Mohamed Salah.

In some cases, the free-spending nature of Saudi clubs has helped the European clubs shed high-earning players who are not currently at their peak and would otherwise have few potential destinations (e.g., Neymar, or any of the players prised from Chelsea). However, Jeddah-based Al-Ahli SFC beat top clubs in England and Italy to the signature of 21-year old Spanish midfielder Gabri Veiga, demonstrating the ability of Saudi clubs to also poach young talent, albeit at astronomical costs (Al-Ahli were willing to pay a higher transfer fee than the European clubs, while Veiga's salary at Al-Ahli will be nearly 6 times higher than the salary offered to him by Napoli).

Comparisons have been made between the Saudi football transfer raid of 2023 and the short-lived emergence of the Chinese Super League in 2015-2016, in which Chinese clubs spent big on aging stars while also signing some younger players still desired by Europe's top clubs, such as Jackson Martinez. Of course, the Chinese Super League fell from prominance even faster than it emerged following the imposition of steep taxes on foreign transfers to stem the tide of capital flowing out of China. There is little worry of similar events unfolding in Saudi Arabia, so many are anticipating that the Saudi league is here to stay in a way the Chinese league was not.

In this analysis I examine Saudi transfers since 2000.
Note that this analysis only includes expenditures and income related to transfers and does not consider salaries. This is important, since many of the biggest "transfers" to the SPL were free signings (e.g., Ronaldo, Karim Benzema, N'Golo Kante) and since star players earn significantly higher salaries in the Saudi League than they would elsewhere.

I perform this analysis in `R` using the typical packages for manipulating and plotting data (`dplyr`,`tidyr`,`ggplot2`).

```R
library(dplyr)
library(tidyr)
library(ggplot2)
```

# Data

The data used for this analysis was scraped from [Transfermarkt](transfermarkt.com) and is available on my [Kaggle](https://www.kaggle.com/datasets/rossi14/saudi-pro-league-transfers).

To scrape the data, I used the scripts available [here](https://github.com/ewenme). Unfortunately, I kept getting `403 Forbidden` errors whenever the `GET()` function in `functions.R` was being called. To resolve this, I replaced the `GET()` call with the following line, using the `bow()` and `scrape()` functions available in the `polite` package:


```R
page <- bow(transfers_url) %>%
        scrape()
```

# Saudi Pro League Transfers Prior to 2023

To begin, we remove the transfers with an unknown fee, select only the columns of interest, then sort by year and fee.

```R
dat <- read.csv("saudi-pro-league-transfers.csv") %>%
       filter( transfer_movement=="in",
               !is.na(fee_cleaned) )  %>% 
       select( club_name,
               player_name,
               age,
               position,
               club_involved_name,
               club_involved_country,
               fee_cleaned,
               year ) %>%
       group_by(year) %>%
       arrange( desc(fee_cleaned) )
```

Let's take a look at the top three incoming transfers by year prior to 2010:

```R
top_3_by_year <- dat %>%
                 slice_head( n=3 )

top_3_by_year %>%
filter( fee_cleaned>0,
        year<2010 ) %>%
print.data.frame()
```


```
       club_name           player_name age           position club_involved_name club_involved_country fee_cleaned year
1   Ittihad Club     Marzouq Al-Otaibi  24     Centre-Forward          Al-Shabab          Saudi Arabia        2.00 2000
2   Ittihad Club       Guilherme Alves  28     Centre-Forward        Corinthians                Brazil        1.10 2002
3   Al-Hilal SFC Abdulaziz Al-Khathran  29 Defensive Midfield          Al-Shabab          Saudi Arabia        0.17 2002
4    Al-Nassr FC    Marcelinho Carioca  32 Attacking Midfield      Vasco da Gama                Brazil        2.00 2003
5    Al-Ahli SFC                   Kim  23     Centre-Forward        Atlético-MG                  Peru        0.50 2003
6   Ittihad Club         Mario Carevic  22      Left Midfield       Hajduk Split               Croatia        0.40 2004
7    Al-Ahli SFC          Lúcio Flávio  25 Attacking Midfield        São Caetano            San Marino        0.15 2004
8   Al-Hilal SFC     Yasser Al-Qahtani  22     Centre-Forward      Al-Qadsiah FC          Saudi Arabia        6.00 2005
9   Ittihad Club                  Lima  23     Centre-Forward        Atlético-PR                Brazil        4.50 2005
10  Ittihad Club          Prince Tagoe  19     Centre-Forward      Hearts of Oak                 Ghana        1.00 2005
11  Ittihad Club                Wagner  21 Attacking Midfield           Cruzeiro                Brazil        9.00 2006
12  Ittihad Club       Alhassane Keita  23     Centre-Forward          FC Zürich           Switzerland        2.50 2006
13  Al-Shabab FC     Amir Azmi Megahed  23        Centre-Back      PAOK Salonika                Greece        0.60 2006
14  Al-Shabab FC    Nasser Al-Shamrani  23     Centre-Forward           Al-Wehda          Saudi Arabia        2.57 2007
15 Al-Ettifaq FC          Prince Tagoe  20     Centre-Forward           Al-Shaab                 Qatar        1.00 2007
16  Al-Hilal SFC            Lelo Mbele  20     Centre-Forward         CS Sfaxien               Tunisia        0.90 2007
17  Al-Hilal SFC           Mirel Radoi  27 Defensive Midfield   Steaua Bucharest               Romania        6.00 2008
18  Al-Hilal SFC Christian Wilhelmsson  28       Right Winger          FC Nantes                France        3.50 2008
19  Ittihad Club           Renato Cajá  24 Attacking Midfield        Ponte Preta                Brazil        2.60 2008
20  Al-Hilal SFC          Thiago Neves  24 Attacking Midfield       Hamburger SV               Germany        7.00 2009
21  Al-Shabab FC                Flávio  29     Centre-Forward            El Ahly                 Egypt        2.75 2009
22   Al-Nassr FC       Víctor Figueroa  25       Right Winger     Chacarita Jrs.             Argentina        1.75 2009
```

<br>

First, we can see that early data is pretty sketchy - between 2000 and 2004 there were just 7 transfers on transfermarkt with a known fee. Many of the biggest incoming transfers between 2000 and 2009 were of Brazilian forwards - not surprising given the propensity of journeyman Brazilian footballers to ply their trade abroad in obscure leagues and given the attacking prowess usually associated with Brazilians. Otherwise, several of the biggest transfers in the early 2000s were between Saudi clubs, e.g., the transfer of Saudi international Marzouq Al-Otaibi from Al-Shabab to Al-Ittihad for €2 million, or the transfer of fellow Saudi international Yasser Al-Qahtani from Al-Qadsiah to Al-Hilal for €6 million. The biggest transfers from outside the SPL during this era were Brazilian Wágner from Cruzerio to Al-Ittihad for €9 million, Brazil international Thiago Neves from Hamburger to Al-Hilal for €7 million, and Romania international Mirel Rădoi from Steaua Bucharest to Al-Hilal for €6 million. Notably, Wágner made just 11 appearences for the Jeddah-based club before returning to Brazil due to Al-Ittihad's failure to finish paying his fee.

Now let's check out the next seven years:

```R
top_3_by_year %>%
filter( fee_cleaned>0,
        (2010 < year) & (year < 2018) ) %>%
print.data.frame()
```

```
      club_name            player_name age           position club_involved_name club_involved_country fee_cleaned year
1  Al-Hilal SFC       Youssef El Arabi  24     Centre-Forward            SM Caen                France        7.50 2011
2  Al-Shabab FC               Fernando  30 Defensive Midfield        G. Bordeaux                France        6.00 2011
3  Al-Hilal SFC          Achille Emaná  29 Attacking Midfield         Real Betis                 Spain        4.50 2011
4  Ittihad Club            Diego Souza  27     Centre-Forward      Vasco da Gama                Brazil        8.20 2012
5   Al-Ahli SFC            Bruno César  24   Central Midfield            Benfica              Portugal        5.00 2012
6   Al-Nassr FC          Rafael Bastos  28 Attacking Midfield           CFR Cluj               Romania        3.50 2012
7   Al-Nassr FC        Yahya Al-Shehri  23 Attacking Midfield         Al-Ettifaq          Saudi Arabia        9.80 2013
8  Al-Hilal SFC           Thiago Neves  28 Attacking Midfield         Fluminense                Brazil        6.00 2013
9  Al-Shabab FC        Macnelly Torres  28 Attacking Midfield      Atl. Nacional              Paraguay        4.75 2013
10  Al-Nassr FC                Hernane  28     Centre-Forward           Flamengo                Brazil        5.30 2014
11  Al-Nassr FC    Adrian Mierzejewski  27 Attacking Midfield        Trabzonspor                Turkey        3.20 2014
12 Al-Hilal SFC         Mihai Pintilii  29 Defensive Midfield   Steaua Bucharest               Romania        2.40 2014
13  Al-Nassr FC Naif Ahmed Taib Hazazi  26     Centre-Forward          Al-Shabab          Saudi Arabia        6.80 2015
14  Al-Nassr FC           Modibo Maïga  27     Centre-Forward           West Ham               England        4.00 2015
15 Ittihad Club           Gelmin Rivas  26     Centre-Forward            Táchira             Venezuela        3.00 2015
16 Al-Hilal SFC           Léo Bonatini  22     Centre-Forward      Estoril Praia              Portugal        5.50 2016
17  Al-Ahli SFC              Ali Awagi  26     Second Striker           Al-Wehda          Saudi Arabia        3.50 2016
18 Ittihad Club                Kahraba  22     Centre-Forward            Zamalek                 Egypt        3.35 2016
19 Al-Hilal SFC           Omar Khribin  23     Centre-Forward       Al-Dhafra FC          Saudi Arabia        5.00 2017
20  Al-Ahli SFC               Leonardo  25        Left Winger           Partizan                Serbia        4.00 2017
21 Al-Hilal SFC       Ezequiel Cerutti  26       Right Winger        San Lorenzo             Argentina        4.00 2017
```


We certainly see a step-up in transfer fees in the early 2010s - multi-million Euro transfers start to become the norm each year among the biggest clubs. Surprisingly, two of the biggest transfers were still intraleague sales, with Saudi internationals Yahya Al-Shehri and Naif Ahmed Taib Hazazi moving to Al-Nassr for €9.8 million and €6.8 million, respectively. However, outside of these two transfers, most of the big-money moves were for attacking players in their late 20s from European and South American clubs.

Finally, let's review 2018-2022. Transfers start to ramp up here so let's take a larger slice of the data.

```R
dat %>%
slice_head( n=7 ) %>%
filter( fee_cleaned>0,
        (2017 < year) & (year < 2023) ) %>%
print.data.frame()
```

```
       club_name         player_name age           position club_involved_name club_involved_country fee_cleaned year
1    Al-Nassr FC          Ahmed Musa  25        Left Winger          Leicester               England       16.50 2018
2    Al-Ahli SFC               Souza  29 Defensive Midfield         Fenerbahce                Turkey       12.00 2018
3    Al-Nassr FC            Giuliano  28   Central Midfield         Fenerbahce                Turkey       10.50 2018
4    Al-Ahli SFC             Djaniny  27     Centre-Forward      Santos Laguna                Mexico       10.24 2018
5    Al-Ahli SFC     Nicolae Stanciu  25 Attacking Midfield      Sparta Prague        Czech Republic       10.00 2018
6   Ittihad Club Aleksandar Prijovic  28     Centre-Forward      PAOK Salonika                Greece       10.00 2018
7   Ittihad Club     Garry Rodrigues  28        Left Winger        Galatasaray                Turkey        9.00 2018
8   Al-Hilal SFC      André Carrillo  28       Right Winger            Benfica              Portugal        9.50 2019
9   Al-Hilal SFC     Gustavo Cuéllar  26 Defensive Midfield           Flamengo                Brazil        7.40 2019
10   Al-Ahli SFC          Lucas Lima  27          Left-Back          FC Nantes                France        6.00 2019
11  Al-Shabab FC      Alfred N'Diaye  29 Defensive Midfield         Villarreal                 Spain        6.00 2019
12   Al-Nassr FC    Abdulfattah Adam  24     Centre-Forward         Al-Taawoun          Saudi Arabia        5.20 2019
13 Al-Ettifaq FC          Naïm Sliti  27        Left Winger              Dijon                France        5.00 2019
14   Al-Ahli SFC     Danijel Aleksic  28 Attacking Midfield     Y. Malatyaspor                Turkey        3.80 2019
15   Al-Nassr FC       Pity Martínez  27 Attacking Midfield            Atlanta         United States       16.00 2020
16  Al-Hilal SFC      Luciano Vietto  26     Centre-Forward        Sporting CP              Portugal        7.00 2020
17   Al-Nassr FC          Ali Lajami  24        Centre-Back           Al-Fateh          Saudi Arabia        5.56 2020
18   Al-Nassr FC       Ali Al-Hassan  23 Defensive Midfield           Al-Fateh          Saudi Arabia        5.56 2020
19  Ittihad Club      Bruno Henrique  30   Central Midfield          Palmeiras                Brazil        4.00 2020
20  Al-Shabab FC     Igor Lichnovsky  26        Centre-Back       CD Cruz Azul                Mexico        2.50 2020
21   Al-Batin FC         Fabio Abreu  27     Centre-Forward         Moreirense              Portugal        2.50 2020
22  Al-Hilal SFC     Matheus Pereira  25 Attacking Midfield          West Brom               England       18.00 2021
23  Ittihad Club       Igor Coronado  28 Attacking Midfield         Sharjah FC  United Arab Emirates       10.15 2021
24   Al-Nassr FC             Talisca  27 Attacking Midfield            GuaZ FC                Brazil        8.00 2021
25  Al-Hilal SFC             Michael  25        Left Winger           Flamengo                Brazil        7.60 2021
26   Al-Nassr FC  Jonathan Rodríguez  28     Centre-Forward       CD Cruz Azul                Mexico        5.29 2021
27 Al-Faisaly FC        Martin Boyle  28       Right Winger       Hibernian FC              Scotland        3.45 2021
28   Al-Ahli SFC       Alassane Ndao  24       Right Winger         Karagümrük                Turkey        3.00 2021
29  Al-Shabab FC     Aaron Boupendza  26     Centre-Forward        Al-Arabi SC                 Qatar        7.00 2022
30   Al-Nassr FC Abdulrahman Ghareeb  25        Left Winger        Al-Ahli SFC          Saudi Arabia        5.25 2022
31  Ittihad Club        Hélder Costa  28       Right Winger              Leeds               England        4.80 2022
32   Al-Batin FC         Renzo López  28     Centre-Forward    Central Córdoba             Argentina        4.70 2022
33 Al-Ettifaq FC    Marcel Tisserand  29        Centre-Back         Fenerbahce                Turkey        3.00 2022
34 Al-Ettifaq FC       Berat Özdemir  24 Defensive Midfield        Trabzonspor                Turkey        2.44 2022
35      Damac FC          Adam Maher  28 Defensive Midfield         FC Utrecht           Netherlands        1.80 2022
```

Not only was the transfer record held by Hazazi broken in 2018, but it was surpassed *six* times as Al-Nassr, Al-Ahli, and Al-Ittihad splurged on midfielders and attackers in their mid-to-late 20s, primarily from European clubs, though the €12.24 million transfer of Cape Verde international Djaniny from Mexican club Santos Laguna stands out as an exception. Al-Hilal joined the spending spree the following year, splashing out €9.5 million for Benfica's André Carrillo and €7.4 million for Flamengo's Gustavo Cuéllar. The transfer record was broken in 2020 when Al-Nassr paid €16 million for Atlanta United's Pity Martínez and again in 2021 when Matheus Pereira moved from West Bromwich Albion to Al-Hilal for €18 million.


# The Floodgates Open - 2023

Since the record transfer for the SPL was €16 million going into 2023, let's take a look at all 2023 transfers surpassing this fee:

```R
dat %>%
filter(year==2023,
       fee_cleaned>=18) %>%
print.data.frame()
```

```
      club_name             player_name age           position club_involved_name club_involved_country fee_cleaned year
1  Al-Hilal SFC                  Neymar  31        Left Winger           Paris SG                France        90.0 2023
2  Al-Hilal SFC                  Malcom  26       Right Winger         Zenit S-Pb                Russia        60.0 2023
3   Al-Nassr FC                  Otávio  28       Right Winger           FC Porto              Portugal        60.0 2023
4  Al-Hilal SFC             Rúben Neves  26 Defensive Midfield             Wolves               England        55.0 2023
5  Al-Hilal SFC     Aleksandar Mitrovic  28     Centre-Forward             Fulham               England        52.6 2023
6  Ittihad Club                 Fabinho  29 Defensive Midfield          Liverpool               England        46.7 2023
7  Al-Hilal SFC Sergej Milinković-Savić  28   Central Midfield              Lazio                 Italy        40.0 2023
8   Al-Ahli SFC             Gabri Veiga  21   Central Midfield      Celta de Vigo                 Spain        40.0 2023
9   Al-Ahli SFC            Riyad Mahrez  32       Right Winger           Man City               England        35.0 2023
10  Al-Ahli SFC            Roger Ibañez  24        Centre-Back            AS Roma                 Italy        30.0 2023
11  Al-Nassr FC              Sadio Mané  31        Left Winger      Bayern Munich               Germany        30.0 2023
12 Ittihad Club                    Jota  24        Left Winger             Celtic              Scotland        29.1 2023
13  Al-Nassr FC         Aymeric Laporte  29        Centre-Back           Man City               England        27.5 2023
14  Al-Ahli SFC     Allan Saint-Maximin  26        Left Winger          Newcastle               England        27.2 2023
15  Al-Nassr FC             Seko Fofana  28   Central Midfield               Lens                France        25.0 2023
16 Al-Hilal SFC       Kalidou Koulibaly  32        Centre-Back            Chelsea               England        23.0 2023
17 Ittihad Club             Luiz Felipe  26        Centre-Back         Real Betis                 Spain        22.0 2023
18 Al-Hilal SFC                    Bono  32         Goalkeeper         Sevilla FC                 Spain        21.0 2023
19  Al-Ahli SFC           Merih Demiral  25        Centre-Back        Atalanta BC                 Italy        20.0 2023
20  Al-Ahli SFC           Edouard Mendy  31         Goalkeeper            Chelsea               England        18.5 2023
21  Al-Nassr FC        Marcelo Brozovic  30 Defensive Midfield              Inter                 Italy        18.0 2023
22 Al-Shabab FC            Habib Diallo  28     Centre-Forward      R. Strasbourg                France        18.0 2023
```

A remarkable 20 transfers in 2023 surpassed the previous record of €18 million, and two more transfers equalling it, with all of these players arriving from European clubs. Prehaps the most eye-catching column is `club_involved_name` - Saudi clubs are no longer primarily targeting players in Arab, South American, or mid-tier European leagues, but are instead pursuing players in the world's biggest leagues. Despite this change of focus, the preference of Saudi clubs for Brazilian forwards appears undiminished - the three most expensive transfers were all for Brazilian-born wingers. Just five of these 22 transfers worth at least €18 million were for defenders (all centre-backs), while two were goalkeepers (Bono from Sevilla and Edouard Mendy from Chelsea).


A simple plot of annual transfer fees puts the incredible scale of 2023 transfers in perspective.

```R

annual_dat <- dat %>%
              filter(year<2024) %>%
              group_by(year,transfer_movement) %>%
              summarize( total_fee=sum(fee_cleaned,na.rm=TRUE) )

in_dat  <- filter( annual_dat, transfer_movement=="in" )
out_dat <- filter( annual_dat, transfer_movement=="out" )

x <- in_dat$year

# Initialize plot

par(mar=c(5,5,1,1))
plot( x=range(x),
      y=c(-2,max(annual_dat$total_fee)*1.05),
      axes=FALSE,
      yaxs="i",
      type="n",
      xlab="",
      ylab="" )
grid()
box()

# Plot bars

rect( xleft=x-0.4, xright=x, ybottom=0, ytop=in_dat$total_fee,
      col="red", border=NA )
rect( xleft=x, xright=x+0.4, ybottom=0, ytop=out_dat$total_fee,
      col="black", border=NA )

# Axis labels

xseq <- seq(from=2000,to=2020,by=5)
xlabels <- paste(xseq,substr(xseq+1,3,4),sep="-")

axis( side=1, cex=1.3, at=xseq, labels=xlabels )
axis( side=2, las=1, cex=1.3 )

legend( x="topleft",
        bty="n",
        pch=22,
        cex=1.3,
        legend=c("Incoming","Outgoing"),
        pt.bg=c("red","black"),
        col=c("red","black") )

mtext( side=1, text="Season", line=3, cex=1.3 )
mtext( side=2, text="Total transfer fees (million €)", line=3, cex=1.3 )

# Annotate

text( x=2017.8, y=420, labels="\"Big Four\"\ngo big", cex=0.9 )
arrows( x0=2017.8, y0=350, y1=200 )

text( x=2003.5, y=260, cex=0.8,
      labels="Al-Qahtani (CF) to Al-Hilal\nLima (CF) to Ittihad")
arrows( x0=2003.5, x1=2004.8, y0=180, y1=20 )

text( x=2008, y=510, cex=0.8,
      labels="Rădoi (DM) and\nWilhelmsson (RW)\n to Al-Hilal")
arrows( x0=2008, x1=2007.8, y0=380, y1=30 )

text( x=2013.5, y=250, cex=0.8,
      labels="Naif Hazazi (CF)\nfrom Al-Shabab\n to Al-Nassr")
arrows( x0=2013.5, x1=2015, y0=145, y1=40 )
```

![income-vs-expenditure-by-year](/images/post8/income-vs-expenditure-by-year.png)


We can see from this plot that transfers were negligible until 2005 and were still somewhat marginal (<€21 million) through 2010. Spending kicked up a notch between 2011 and 2017, ranging between €22-41 million annually. The 2018-2019 season marked the first major splash of cash, with teams spending nearly €200 million combined. Spending remained relatively high (€43-93 million) between 2019 and 2022 and nearly reached €1 billion in 2023. If Al-Ittihad's move for Mo Salah had been successful, total spending would indeed have eclipsed the €1 billion mark.

Player sales from Saudi clubs were marginal compared to purchases. In fact, most transfers of players from Saudi clubs for nonzero fees were to other Saudi clubs.


## 2023 transfers by team

Let's visualize which clubs were most active in the 2023 transfer market. To aid this visualization, I've created a table of Saudi Pro League clubs [here](https://github.com/stevenrossi/saudi-transfers/blob/main/saudi-clubs.csv), with a column indicating the region of Saudi Arabia that the club is from. I've also scraped `.png` logos for each team from Wikipedia and uploaded them [here](https://github.com/stevenrossi/saudi-transfers/tree/main/logos). We can use these to make the plots a little more interesting. We can add these logos to our plot by first reading them into raster arrays using the `png` package then plotting the rasters using the `sinkr` package.

```R
library(png)
library(sinkr)

# Annual fees by club and movement type (in or out)
dat_2023 <- read.csv("saudi-pro-league-transfers.csv") %>%
            filter( year==2023 ) %>%
            group_by(club_name,transfer_movement) %>%
            summarize( totalFee=sum(fee_cleaned,na.rm=TRUE))

# Get transfer fees for incoming players and sort from highest to lowest
in_2023  <- filter( dat_2023, transfer_movement=="in" ) %>%
              arrange( desc(totalFee) )

# Get transfer fees for outgoing players and match incoming sorting
out_2023 <- filter( dat_2023, transfer_movement=="out" )
out_2023 <- out_2023[match(in_2023$club_name,out_2023$club_name),]

# Number of teams
nTeam <- nrow(in_2023)

# Create plotting region
par(mar=c(5,5,1,1))
plot( x=c(0,nTeam+1),
      y=c(-2,max(dat_2023$totalFee)*1.15),
      axes=FALSE,
      yaxs="i",
      type="n",
      xlab="",
      ylab="" )
grid()
box()

# Plot red bars for incomings and black bars for outgoings
x <- 1:nTeam
rect( xleft=x-0.4,
      xright=x,
      ybottom=0,
      ytop=in_2023$totalFee,
      col="red",
      border="red" )
rect( xleft=x,
      xright=x+0.4,
      ybottom=0,
      ytop=out_2023$totalFee,
      col="black" )

# Labels
axis( side=1, at=x, labels=NA )
axis( side=2, las=1, cex=1.3 )
par( font=2 )
text( x=x,
      y=par("usr")[3]-20,
      adj=0.95,
      xpd=NA,
      cex=0.8,
      labels=in_2023$club_name,
      srt=35,
      col=cols )
par( font=1)

mtext( side=2, text="Total transfer fees in 2023 (million €)", line=3, cex=1.3 )

legend( x=8,
        y=max(dat_2023$totalFee)*1.1,
        bty="n",
        pch=22,
        cex=1.3,
        legend=c("Incoming","Outgoing"),
        pt.bg=c("red","black"),
        col=c("red","black") )

# Load region names and give each a unique colour
club_table <- read.csv("saudi-clubs.csv")
textCol <- brewer.pal(max(club_table$code)+1,"Set1")[-6]
cols <- textCol[club_table$code[match(in_2023$club_name,club_table$club_name)]]

regions <- club_table %>%
           group_by(region) %>%
           summarize(code=mean(code)) %>%
           arrange(code) %>%
           .$region %>%
           unique()

par(font = 2)
# Legend title
legend( x=13.75,
        y=max(dat_2023$totalFee)*1.1,
        legend="Region",
        bty="n",
        cex=1.3
        )
# Legend content
legend( x=14,
        y=max(dat_2023$totalFee),
        legend=regions,
        text.col=textCol,
        bty="n",
        cex=1.1 )
par(font = 1)

# Plot logos
for( i in 1:nTeam )
{
  shortName <- gsub(" .*","",in_2023$club_name[i])
  pngName <- paste0("logos/",shortName,".png")
  pngObj  <- readPNG(pngName)
  y <- max(in_2023$totalFee[i],out_2023$totalFee[i])
  addImg( obj=pngObj, x=x[i], y=y+30, width=0.9 )
}
```

![income-vs-expenditure-by-team](/images/post8/income-vs-expenditure-by-team.png)


Al-Hilal sit comfortably atop the heap, having spent over €350 million on incoming players, while Al-Ahli are second with a transfer bill of nearly €200 million. Al-Nassr and Al-Ittihad come in third and fourth, having spent €165 million and €119 million, respectively, though it should be noted that Al-Ittihad's spending would have comfortably eclipsed Al-Alhi's and Al-Nassr's had they managed to buy Mo Salah from Liverpool. Notably, only two clubs have a transfer surplus: Al-Taawoun and Al-Fateh.


## Age distribution of 2023 incomings

It's clear that the age-distribution of players purchased for big transfer fees skews older - of the 22 transfers above €18 million listed above, just three are 25 years old or younger. Let's aggregate the data into approximately 5-year bins and compare the age distribution of incoming transfers.

```R
age_dat <- dat %>%
           mutate( year = as.character(year),
                          # Can still numerically compare characters
                   year = ifelse( year<2006, "2000-2005",
                           ifelse( year<2011, "2006-2010",
                            ifelse( year<2016, "2011-2015",
                             ifelse( year<2023, "2016-2022", "2023" ) ) ) )
                    )
plot_age_dist <- function( dat )
  ggplot(dat, aes(x=age, fill=year)) +
  geom_density(alpha=0.4) + 
  xlim( 16, 42 ) + 
  labs( x="Age (yr)",
        y="Density" )

plot_age_dist( dat=age_dat)
```

![age-distributions-all](/images/post8/age-distributions-all.png)

The age of players joining the SPL has increased over the last two decades - the most common age of new recruits increased from 23-24 years old in the early 2000s to 27-30 years old in 2023.

# Where are incoming players coming from?

We can output the ten foreign clubs that have sold the most players to Saudi clubs as follows:

```R
dat %>%
filter( club_involved_country != "Saudi Arabia",
        fee_cleaned > 0 ) %>%
group_by( club_involved_name ) %>%
summarize( n=n() ) %>%
arrange( desc(n) )
```

```
# A tibble: 242 × 2
   club_involved_name     n
   <chr>              <int>
 1 El Ahly               11
 2 ES Sahel               7
 3 ES Tunis               7
 4 Ismaily                7
 5 Red Star               7
 6 Zamalek                7
 7 Steaua Bucharest       6
 8 Galatasaray            5
 9 Partizan               5
10 Trabzonspor            5
```

Egyptian and Tunisian clubs account for 5 of the top 6 clubs that have transferred the most players to Saudi Arabia. The other five clubs are from Turkey and eastern Europe, including both of Belgrade's fierce rivals. It is  not surprising that Egypt would be an important source of talent for the Saudi league, given the relative strength of Egyptian rivals Al Ahly and Zamalek in the Arab football world. I was, however, surprised to see two Tunisian clubs in the top 10.

Let's visualize where incoming players are being bought from. To simplify things, we'll look more broadly at which countries players are being bought from. First let's grab a list of the 10 countries that have transferred the most players to Saudi Arabia.

```R
top_10_countries <- dat %>%
                    group_by( club_involved_country ) %>%
                    summarize( n=n() ) %>%
                    arrange( desc(n) ) %>%
                    .[1:10,"club_involved_country"] %>%
                    unlist() %>%
                    unname()

top_10_countries
```

```
 [1] "Saudi Arabia" "France"       "Egypt"        "Portugal"     "Brazil"       "England"      "Turkey"       "Tunisia"     
 [9] "Romania"      "Spain"
```

Nothing looks too out of place here - Europe's most prominant footballing countries other than Italy are all present, as well as Brazil. I originally calculated the top 10 countries by total fees received rather than number of players sold - the list was virtually identical, except Russia made the top 10 solely on the back of Malcom's €60 million move from Zenit to Al-Hilal. According to Transfermarkt, Malcom was the only 
player to ever transfer from a Russian club to a Saudi club for a fee, though 7 players did join Saudi clubs for free after leaving Russian clubs. As such, I figured having Russia in the top 10 would not be too useful and so I recalculated the top 10 based on number of players sold.

Given the astronomical increase in transfer fees paid by (Saudi) clubs over the last 20 years, it makes more sense to compare the proportion of fees spent by country over time rather than the fees themselves. The following code aggregates fees by the country of the selling club, re-labels all countries outside of the "top 10" as `Other`, and fills on 0s for all missing entries using the `complete()` function in the `tidyr` package:

```R
in_top_10 <- dat %>%
             group_by( year, club_involved_country ) %>%
             summarize( fee_total = sum(fee_cleaned) ) %>%
             mutate( p = fee_total/sum(fee_total),
                     club_involved_country=ifelse(club_involved_country%in%top_10_countries,
                                                  club_involved_country,
                                                  "Other") ) %>%
             ungroup() %>%
             complete( year,
                       club_involved_country,
                       fill=list(p=0,p_weighted=0))
```

We can then use `ggplot` to create a stacked barplot:

```R
# Define a colour palette and
pal <- c( "#000000", "#7BEDC7", "#999999", "#E69F00", "#56B4E9", "#0072B2", "#F0E442", "#009E73", "#D55E00", "#CC79A7","#8A451A")

p <- ggplot( in_top_10, 
             aes( x=year,
                  y=p,
                  fill=factor(x=club_involved_country,
                              levels=c( "Other", rev(top_10_countries) ) ) ) )
p + 
  geom_bar( position="stack", stat="identity" )  +
  ylim( 0, 1 ) + 
  scale_fill_manual(values=pal) +
  scale_x_continuous("Season", breaks=c(2000,2010,2020), labels=c("2000/2001","2010/2011","2020/2021") ) + 
  guides(colour=guide_legend(ncol=4)) + 
  labs( y="Proportion",
        title="Saudi incoming transfer fees by country of selling club",
        fill=NULL ) +
  theme(axis.title.x = element_text(margin = margin(t = 15), size=12),
        axis.title.y = element_text(margin = margin(r = 15), size=12),
        axis.text.x = element_text(size = 11),
        axis.text.y = element_text(size = 11) )
```

![proportion-by-country](/images/post8/proportion-by-country.png)

In the first decade of the 2000s, most Saudi transfer payments went to clubs from mid-tier leagues or lower (e.g., Brazil, Saudi Arabia, Tunisia, Romania, Egypt). In the 2010s, larger chunks of Saudi transfer budgets start getting spent in more prominant leagues (e.g, France, Portugal, Spain, Turkey, England). In 2023, virtually all Saudi transfer fees were paid to clubs in Europe's top leagues.


# Are all the fees just being spent on attacking players?

Well, sort of, but this aspect of the Saudi transfer policy has actually been more restrained than in some previous years. To see this, we can first re-label the positions in the dataset more broadly:

```R
unique(dat$position)
```

```
 [1] "Left Winger"        "Right Winger"       "Defensive Midfield" "Centre-Forward"    
 [5] "Central Midfield"   "Centre-Back"        "Goalkeeper"         "Attacking Midfield"
 [9] "Left-Back"          "Second Striker"     "Midfield"           "Left Midfield"     
[13] "Right-Back"         "Right Midfield"     "Defence"            "Attack"  
```

Goalkeepers are unambiguously labelled. Any label with "Defence" or "-Back" represents a defender, any label with "Midfield" represents a midfielder, and anything else is an attacker:

```R
dat_pos <- dat %>%
           mutate( position_gen = ifelse( grepl("Goalkeeper", position ), "Goalkeeper", 
                                   ifelse( grepl("Back|Defence", position ), "Defence",
                                    ifelse( grepl("Midfield", position), "Midfield","Attack" ) ) ) ) %>%
           group_by( year, position_gen ) %>%
           summarize( fee_total = sum(fee_cleaned) ) %>%
           ungroup() %>%
           # fill-in 0 for missing cells
           complete( year,
                     position_gen,
                     fill=list(fee_total=0))

g <- ggplot( dat_pos,
             aes( x=year,
                  y=fee_total,
                  fill=factor( x=position_gen,
                               levels=c("Goalkeeper",
                                        "Defence",
                                        "Midfield",
                                        "Attack" ) ) ) )
g + 
  geom_bar( position="fill", stat="identity" ) +
  scale_fill_brewer(type="qual",palette="Set1",name = "Position") + 
  scale_x_continuous("Season", breaks=c(2000,2010,2020), labels=c("2000/2001","2010/2011","2020/2021") ) + 
  labs( y="Proportion of transfer fees spent" )
```

![position-props](/images/post8/position-props.png)

Fifty percent of transfer fees in 2023 were paid for attacking players, which represents a 12% decline from the previous year.




```R
dat %>%
filter( year==2023, 
        fee_cleaned==0,
        club_involved_country%in%c("England",
                                   "Italy",
                                   "France",
                                   "Spain") )
```

# Next steps

Saudi clubs spent nearly €1 billion in the 2023 summer transfer window - a total surpassed only by clubs in the English Premier League. The €1 billion mark for Saudi clubs the 2023-2024 season will certainly be surpassed once the winter window opens and clubs can spend again. It will be interesting to see whether Saudi clubs keep up this level of activity in future seasons and I'll be doing follow up posts to track this. I'll also write some posts comparing transfer activity across various leagues.




