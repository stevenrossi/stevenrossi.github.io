---
title: 'Fitting finite mixture models to salmon arrival distributions using expectation-maximization'
date: 2023-09-04T01:01:01-07:00
draft: true
tags: [Salmon, R, mclust, Clustering, Machine Learning, Expectation-Maximization, Unsupervised Algorithm]
---


# Mixture models and expectation-maximization


# Application: Fraser River Sockeye, Summer run-timing group


## Data

```R
library(dplyr)
library(mclust)

dat <- read.csv("sockeye-mission-passage.csv")

str(dat)
```

```
'data.frame': 646 obs. of  7 variables:
 $ year        : int  2003 2003 2003 2003 2003 2003 2003 2003 2003 2003 ...
 $ date_ordinal: int  185 186 187 188 189 190 191 192 193 194 ...
 $ Ch          : num  34.3 17.2 125.8 174.9 331.7 ...
 $ Qu          : num  8.01 4 29.36 40.82 77.41 ...
 $ Ls          : num  NA NA NA NA NA NA NA NA NA NA ...
 $ St          : num  NA NA NA NA NA NA NA NA NA NA ...
 $ LsSt        : num  0 0 0 0 0 0 0 0 0 0 ...
```

```R
unique(dat$year)
```

```
[1] 2003 2004 2005 2006 2007 2008 2009 2010 2011
```

```R
# Years with aggregate Ls/St estimate
dat %>%
filter( stock=="LsSt",
        !is.na(count) ) %>%
.$year %>%
unique()
```

```
[1] 2003 2005
```


