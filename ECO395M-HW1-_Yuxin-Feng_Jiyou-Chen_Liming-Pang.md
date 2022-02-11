<br> <br> <br>

# Question1 - Flights at ABIA

<br>

#### <font color = darkslateblue>Library several packages we will need.</font>

<br>

    library(tidyverse)

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.1 ──

    ## ✓ ggplot2 3.3.5     ✓ purrr   0.3.4
    ## ✓ tibble  3.1.6     ✓ dplyr   1.0.7
    ## ✓ tidyr   1.1.4     ✓ stringr 1.4.0
    ## ✓ readr   2.1.1     ✓ forcats 0.5.1

    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

    library(mosaic)

    ## Registered S3 method overwritten by 'mosaic':
    ##   method                           from   
    ##   fortify.SpatialPolygonsDataFrame ggplot2

    ## 
    ## The 'mosaic' package masks several functions from core packages in order to add 
    ## additional features.  The original behavior of these functions should not be affected by this.

    ## 
    ## Attaching package: 'mosaic'

    ## The following object is masked from 'package:Matrix':
    ## 
    ##     mean

    ## The following objects are masked from 'package:dplyr':
    ## 
    ##     count, do, tally

    ## The following object is masked from 'package:purrr':
    ## 
    ##     cross

    ## The following object is masked from 'package:ggplot2':
    ## 
    ##     stat

    ## The following objects are masked from 'package:stats':
    ## 
    ##     binom.test, cor, cor.test, cov, fivenum, IQR, median, prop.test,
    ##     quantile, sd, t.test, var

    ## The following objects are masked from 'package:base':
    ## 
    ##     max, mean, min, prod, range, sample, sum

    library(ggplot2)
    library(data.table)

    ## 
    ## Attaching package: 'data.table'

    ## The following objects are masked from 'package:dplyr':
    ## 
    ##     between, first, last

    ## The following object is masked from 'package:purrr':
    ## 
    ##     transpose

    library(rsample)
    library(caret)

    ## 
    ## Attaching package: 'caret'

    ## The following object is masked from 'package:mosaic':
    ## 
    ##     dotPlot

    ## The following object is masked from 'package:purrr':
    ## 
    ##     lift

    library(modelr)

    ## 
    ## Attaching package: 'modelr'

    ## The following object is masked from 'package:mosaic':
    ## 
    ##     resample

    ## The following object is masked from 'package:ggformula':
    ## 
    ##     na.warn

    library(parallel)
    library(foreach)

    ## 
    ## Attaching package: 'foreach'

    ## The following objects are masked from 'package:purrr':
    ## 
    ##     accumulate, when

<br>

### Question a: What’s the best time of year to fly to minimize delays?

<br>

    ABIA = read.csv("https://raw.githubusercontent.com/jgscott/ECO395M/master/data/ABIA.csv")
    # What is the best time of day to fly to minimize delays? 
    ABIA %>%
      summarize(mean_DepDelay = mean(DepDelay))

    ##   mean_DepDelay
    ## 1            NA

    ABIA %>% 
      summarize(favstats(DepDelay))

    ##   min Q1 median Q3 max     mean       sd     n missing
    ## 1 -42 -4      0  8 875 9.171135 31.15531 97847    1413

    ABIA %>%
      summarize(mean_DepDelay = mean(DepDelay, na.rm=TRUE))

    ##   mean_DepDelay
    ## 1      9.171135

    by_monthly_uniquecarrier = ABIA %>% 
      group_by(Month, UniqueCarrier) %>% 
      summarize(count = n(),
                mean_DepDelay = mean(DepDelay, na.rm=TRUE))

    ## `summarise()` has grouped output by 'Month'. You can override using the `.groups` argument.

    by_monthly_uniquecarrier

    ## # A tibble: 175 × 4
    ## # Groups:   Month [12]
    ##    Month UniqueCarrier count mean_DepDelay
    ##    <int> <chr>         <int>         <dbl>
    ##  1     1 9E              175        17.4  
    ##  2     1 AA             1728         9.43 
    ##  3     1 B6              242         6.56 
    ##  4     1 CO              764         6.26 
    ##  5     1 DL               92         8.67 
    ##  6     1 EV              266        13.9  
    ##  7     1 F9              178         2.49 
    ##  8     1 MQ              460         0.991
    ##  9     1 NW               23        -1.57 
    ## 10     1 OH              281        11.2  
    ## # … with 165 more rows

    ggplot(by_monthly_uniquecarrier) + geom_col(aes(x=factor(Month), y=mean_DepDelay))

![](ECO395M-HW1-_Yuxin-Feng_Jiyou-Chen_Liming-Pang_files/figure-markdown_strict/unnamed-chunk-3-1.png)

<br>

#### <font color = darkslateblue>Answer:</font>

#### <font color = darkslateblue>The chart below shows that December is the worst time to fly since the highest mean-delay.</font>

<br>

<br>

### Question b: Does the best time to fly to min delays change by destination?

<br>

    ggplot(by_monthly_uniquecarrier) + 
      geom_col(aes(x=factor(Month), y=mean_DepDelay)) + 
      facet_wrap(~UniqueCarrier)

![](ECO395M-HW1-_Yuxin-Feng_Jiyou-Chen_Liming-Pang_files/figure-markdown_strict/unnamed-chunk-4-1.png)

<br>

#### <font color = darkslateblue>Answer:</font>

#### <font color = darkslateblue>Yes. Different airlines have different situation (shown in the last table). Take AA as example, the worst month is June apparently.</font>

<br>

    by_monthly_uniquecarrier %>% 
      group_by(UniqueCarrier) %>%
      slice_max(order_by = mean_DepDelay, n=1)

    ## # A tibble: 16 × 4
    ## # Groups:   UniqueCarrier [16]
    ##    Month UniqueCarrier count mean_DepDelay
    ##    <int> <chr>         <int>         <dbl>
    ##  1     1 9E              175         17.4 
    ##  2     6 AA             1719         14.2 
    ##  3    12 B6              489         22.4 
    ##  4    12 CO              743         16.0 
    ##  5     3 DL              121         25.3 
    ##  6    11 EV                2         74   
    ##  7    12 F9              168         14.8 
    ##  8     3 MQ              446         12.1 
    ##  9     4 NW               93         10.9 
    ## 10    12 OH               82         32.7 
    ## 11    12 OO              129         27.1 
    ## 12     6 UA              174         19.1 
    ## 13     8 US              120          5.91
    ## 14    12 WN             3022         17.8 
    ## 15     3 XE              762         11.1 
    ## 16     6 YV              512         21.5

<br> <br> <br> <br>

# Question2 - Wrangling the Billboard Top100

<br>

## Part A

<br>

### Top 10 songs since 1958

<br>

    billboard <- read.csv("https://raw.githubusercontent.com/jgscott/ECO395M/master/data/billboard.csv", row.names=1)
    billboard_select = select(billboard, performer, song)
    bb = count(billboard_select, performer, song) %>% arrange(desc(n))
    colnames(bb) = c("performer", "song", "count")
    billboard_top10 = bb[1:10,]

    setDT(billboard_top10)
    print (billboard_top10)

    ##                                     performer
    ##  1:                           Imagine Dragons
    ##  2:                                AWOLNATION
    ##  3:                                Jason Mraz
    ##  4:                                The Weeknd
    ##  5:                               LeAnn Rimes
    ##  6: LMFAO Featuring Lauren Bennett & GoonRock
    ##  7:                               OneRepublic
    ##  8:                                     Adele
    ##  9:                                     Jewel
    ## 10:                          Carrie Underwood
    ##                                    song count
    ##  1:                         Radioactive    87
    ##  2:                                Sail    79
    ##  3:                           I'm Yours    76
    ##  4:                     Blinding Lights    76
    ##  5:                       How Do I Live    69
    ##  6:                   Party Rock Anthem    68
    ##  7:                      Counting Stars    68
    ##  8:                 Rolling In The Deep    65
    ##  9: Foolish Games/You Were Meant For Me    65
    ## 10:                    Before He Cheats    64

    knitr::kable(billboard_top10,
                 caption = "This is the top 10 songs since 1958 on Billboard!")

<table>
<caption>This is the top 10 songs since 1958 on Billboard!</caption>
<thead>
<tr class="header">
<th style="text-align: left;">performer</th>
<th style="text-align: left;">song</th>
<th style="text-align: right;">count</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">Imagine Dragons</td>
<td style="text-align: left;">Radioactive</td>
<td style="text-align: right;">87</td>
</tr>
<tr class="even">
<td style="text-align: left;">AWOLNATION</td>
<td style="text-align: left;">Sail</td>
<td style="text-align: right;">79</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Jason Mraz</td>
<td style="text-align: left;">I’m Yours</td>
<td style="text-align: right;">76</td>
</tr>
<tr class="even">
<td style="text-align: left;">The Weeknd</td>
<td style="text-align: left;">Blinding Lights</td>
<td style="text-align: right;">76</td>
</tr>
<tr class="odd">
<td style="text-align: left;">LeAnn Rimes</td>
<td style="text-align: left;">How Do I Live</td>
<td style="text-align: right;">69</td>
</tr>
<tr class="even">
<td style="text-align: left;">LMFAO Featuring Lauren Bennett &amp; GoonRock</td>
<td style="text-align: left;">Party Rock Anthem</td>
<td style="text-align: right;">68</td>
</tr>
<tr class="odd">
<td style="text-align: left;">OneRepublic</td>
<td style="text-align: left;">Counting Stars</td>
<td style="text-align: right;">68</td>
</tr>
<tr class="even">
<td style="text-align: left;">Adele</td>
<td style="text-align: left;">Rolling In The Deep</td>
<td style="text-align: right;">65</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Jewel</td>
<td style="text-align: left;">Foolish Games/You Were Meant For Me</td>
<td style="text-align: right;">65</td>
</tr>
<tr class="even">
<td style="text-align: left;">Carrie Underwood</td>
<td style="text-align: left;">Before He Cheats</td>
<td style="text-align: right;">64</td>
</tr>
</tbody>
</table>

This is the top 10 songs since 1958 on Billboard!

<br>

## Part B

<br>

### Musical Diversity Trend

<br>

    billboard_select2 = select(billboard, song_id, year)
    bb_diversity_unorganized1 = distinct(billboard_select2, song_id, year) %>% 
      arrange(year)
    bb_diversity_unorganized2 = count(bb_diversity_unorganized1, year)
    colnames(bb_diversity_unorganized2) = c("year","numbers")
    n = nrow(bb_diversity_unorganized2)
    billboard_diversity_by_year = bb_diversity_unorganized2[2:n-1,]

    bbpicture = ggplot(data = billboard_diversity_by_year) + 
      geom_point(aes(x = year, y = numbers)) +
      geom_line(aes(x = year, y = numbers), size = 0.5, color = "Orange")
    bbpicture + xlab("year(1589-2020)") + 
      ylab("Musical Diversity") +
      ggtitle("Billboard Musical Diversity Changes by year")

![](ECO395M-HW1-_Yuxin-Feng_Jiyou-Chen_Liming-Pang_files/figure-markdown_strict/unnamed-chunk-7-1.png)

<br>

#### <font color = darkslateblue> Conclusion:</font>

#### <font color = darkslateblue> Once Billboard was introduced to the public, the diversity of music goes up very quickly until reached the peak in the 1960s, then it starts to go down for a long time and reached the lowest point around the start of the 2020s. After that, it appeared to go up again.</font>

<br>

## Part C

<br>

### Nineteen Artists with over 30 “ten-week-hit” songs

<br>

    ten_week_hit = bb <- filter(bb, bb$count >= 10) %>% arrange(performer)
    new_list = count(ten_week_hit, performer) %>% arrange(desc(n))
    excellent_artists  = new_list[1:19,]

    singer_pic = ggplot(data = excellent_artists, 
                        mapping = aes(x = performer, y = n, fill = performer))
    singer_pic + geom_col() +
      xlab("Singers with 30+ 'Ten-Week-Hit' Songs") + 
      ylab("Number of Songs on Billboard for over 10 weeks") +
      ggtitle("Legendary Artists in Billboard History") +
      theme(axis.text.x = element_text(angle = 45, vjust = 1, hjust = 1))

![](ECO395M-HW1-_Yuxin-Feng_Jiyou-Chen_Liming-Pang_files/figure-markdown_strict/unnamed-chunk-8-1.png)

<br> <br> <br> <br>

# Question3 - Wrangling the Olympics

<br>

## Part A

<br>

### 95 Percentile of Heignts for Female Competitors

<br>

    olympics_top20 = read.csv("https://raw.githubusercontent.com/jgscott/ECO395M/master/data/olympics_top20.csv")
    olympics_top20_Female = olympics_top20 %>% 
      filter(sex == "F")
    head(olympics_top20_Female)

    ##    id                                    name sex age height weight
    ## 1  37                      Ann Kristin Aarnes   F  23    182     64
    ## 2  67 Mariya Vasilyevna Abakumova (-Tarabina)   F  22    179     80
    ## 3  90               Tamila Rashidovna Abasova   F  21    163     60
    ## 4 259                              Reema Abdo   F  21    173     59
    ## 5 394                              Irene Abel   F  19    160     48
    ## 6 428                       Elvan Abeylegesse   F  25    159     40
    ##           team noc       games year season        city      sport
    ## 1       Norway NOR 1996 Summer 1996 Summer     Atlanta   Football
    ## 2       Russia RUS 2008 Summer 2008 Summer     Beijing  Athletics
    ## 3       Russia RUS 2004 Summer 2004 Summer      Athina    Cycling
    ## 4       Canada CAN 1984 Summer 1984 Summer Los Angeles   Swimming
    ## 5 East Germany GDR 1972 Summer 1972 Summer      Munich Gymnastics
    ## 6       Turkey TUR 2008 Summer 2008 Summer     Beijing  Athletics
    ##                                          event  medal
    ## 1                    Football Women's Football Bronze
    ## 2              Athletics Women's Javelin Throw Silver
    ## 3                       Cycling Women's Sprint Silver
    ## 4 Swimming Women's 4 x 100 metres Medley Relay Bronze
    ## 5           Gymnastics Women's Team All-Around Silver
    ## 6               Athletics Women's 5,000 metres Silver

    olympics_top20_Female %>% summarize(q95_height = quantile(height, 0.95)) %>% round(3)

    ##   q95_height
    ## 1        186

<br>

#### <font color = darkslateblue> Female Competitor’s 95 percentile height is 186.</font>

<br>

## Part B

<br>

### Women’s Event with the Greatest Variability in Competitor’s Height

<br>

    olympics_top20_Female %>%
      group_by(event) %>%
      summarize(sd_height = sd(height)) %>% summarize(event=event[which.max(sd_height)],max_sd=max(sd_height, na.rm = TRUE))

    ## # A tibble: 1 × 2
    ##   event                      max_sd
    ##   <chr>                       <dbl>
    ## 1 Rowing Women's Coxed Fours   10.9

<br>

#### <font color = darkslateblue> “Rowing Women’s Coxed Fours” has the greatest women height variability.</font>

<br>

## Part C

<br>

### Average Age of Olympic Swimmers Trend

<br>

    olympics_top20_swimmers = olympics_top20 %>% 
      filter(sport == "Swimming")


    df_m_c <- data.frame(olympics_top20 %>% 
                           filter(sport == "Swimming", sex == "M" ) %>%
                           group_by(year) %>%
                           summarize(avg_age = mean(age, na.rm = TRUE)) %>%
                           round(3))


    df_f_c <- data.frame(olympics_top20 %>%
                           filter(sport == "Swimming", sex == "F" ) %>%
                           group_by(year) %>%
                           summarize(avg_age = mean(age, na.rm = TRUE)) %>%
                           round(3))


    df_all <- olympics_top20 %>% select(sex, age, year, sport) %>%
                          filter(sport == "Swimming") %>% 
                          group_by(year, sex) %>%
                          summarize(avg_age = mean(age, na.rm = TRUE))

    ## `summarise()` has grouped output by 'year'. You can override using the `.groups` argument.

    ggplot(NULL,aes(year, avg_age)) +
      geom_line(data = df_m_c, col = "red") +
      geom_line(data = df_f_c, col = "blue") + labs(x = "Years", y = "Age", title = "Trend of Swimmers")

![](ECO395M-HW1-_Yuxin-Feng_Jiyou-Chen_Liming-Pang_files/figure-markdown_strict/unnamed-chunk-11-1.png)

<br>

#### <font color = darkslateblue> Concllusion: The trend bewteen male and female competitors is different. Female competitor’s ages are usually bigger than male’s. Both average ages began to increase gradually since 1950.</font>

<br> <br> <br> <br>

# Question4 - k-Nearest Neighbors

<br>

### Predictive Model

<br>

    sclass = read.csv("https://raw.githubusercontent.com/jgscott/ECO395M/master/data/sclass.csv")
    sclass_350 = sclass %>% 
      filter(trim == "350") %>% select("mileage", "price")

    sclass_65 = sclass %>% 
      filter(trim == "65 AMG") %>% select("mileage", "price")

    sclass_350_split =  initial_split(sclass_350, prop=0.8)
    sclass_350_train = training(sclass_350_split)
    sclass_350_test  = testing(sclass_350_split)

    sclass_65_split =  initial_split(sclass_65, prop=0.8)
    sclass_65_train = training(sclass_65_split)
    sclass_65_test  = testing(sclass_65_split)

    lm_350 = lm(price ~ mileage, data=sclass_350_train)
    lm_65 = lm(price ~ mileage, data=sclass_65_train)


    K_folds = 10

    sclass_350_folds = crossv_kfold(sclass_350, k=K_folds)
    sclass_65_folds = crossv_kfold(sclass_350, k=K_folds)


    sclass_350_models = map(sclass_350_folds$train, ~ knnreg(price ~ mileage, k=100, data = ., use.all=FALSE))
    sclass_65_models = map(sclass_65_folds$train, ~ knnreg(price ~ mileage, k=100, data = ., use.all=FALSE))


    sclass_350_rmse = map2_dbl(sclass_350_models, sclass_350_folds$test, modelr::rmse)
    mean(sclass_350_rmse)

    ## [1] 10877.58

    sclass_65_rmse = map2_dbl(sclass_350_models, sclass_65_folds$test, modelr::rmse)
    mean(sclass_65_rmse)

    ## [1] 10733.02

    k_grid = c(2, 4, 6, 8, 10, 15, 20, 25, 30, 35, 40, 45,
               50, 60, 70, 80, 90, 100, 125, 150, 175, 200, 250, 300)


    sclass_350_grid = foreach(k = k_grid, .combine='rbind') %dopar% {
      models = map(sclass_350_folds$train, ~ knnreg(price ~ mileage, k=k, data = ., use.all=FALSE))
      sclass_350_rmse = map2_dbl(models, sclass_350_folds$test, modelr::rmse)
      c(k=k, err = mean(sclass_350_rmse))
    } %>% as.data.frame

    ## Warning: executing %dopar% sequentially: no parallel backend registered

    sclass_65_grid = foreach(k = k_grid, .combine='rbind') %dopar% {
      models = map(sclass_65_folds$train, ~ knnreg(price ~ mileage, k=k, data = ., use.all=FALSE))
      sclass_65_rmse = map2_dbl(models, sclass_65_folds$test, modelr::rmse)
      c(k=k, err = mean(sclass_65_rmse))
    } %>% as.data.frame

<br>

### 350’s AMG

<br>

    ggplot(sclass_350_grid) + 
      geom_line(aes(x=k, y=err)) + labs(x = "k", y = "RMSE_350", title = "350") +
      scale_x_log10()

![](ECO395M-HW1-_Yuxin-Feng_Jiyou-Chen_Liming-Pang_files/figure-markdown_strict/unnamed-chunk-13-1.png)

<br>

#### <font color = darkslateblue> From the picture we guess that RMSE\_350 reaches its bottom when k is in \[10,20\].</font>

<br>

### 65’s AMG

<br>

    ggplot(sclass_65_grid) + 
      geom_line(aes(x=k, y=err)) + labs(x = "k", y = "RMSE_65", title = "65") +
      scale_x_log10()

![](ECO395M-HW1-_Yuxin-Feng_Jiyou-Chen_Liming-Pang_files/figure-markdown_strict/unnamed-chunk-14-1.png)

<br>

#### <font color = darkslateblue> From the picture we guess that RMSE\_65 reaches its bottom when k is in \[10,20\].</font>

<br>

### Calculating 350’s and 65’s AMG optimal k value

<br>

    min_k_350 <- sclass_350_grid %>% summarize(k=k[which.min(err)])
    print(min_k_350)

    ##    k
    ## 1 15

    min_k_65 <- sclass_65_grid %>% 
      summarize(k=k[which.min(err)])
    print(min_k_65)

    ##    k
    ## 1 15

<br>

#### <font color = darkslateblue> Optimal k’s for 350’s and 65’s are both 15.</font>

<br>

### sclass\_350\_test

<br>

    knn350 = knnreg(price ~ mileage, data=sclass_350_train, k=min_k_350[1,1])
    rmse(knn350, sclass_350_test)

    ## [1] 9950.014

    sclass_350_test = sclass_350_test %>%
      mutate(price_pred = predict(knn350, sclass_350_test))

    test_350 = ggplot(data = sclass_350_test) + 
      geom_point(mapping = aes(x = mileage, y = price), alpha=0.2)
    test_350

![](ECO395M-HW1-_Yuxin-Feng_Jiyou-Chen_Liming-Pang_files/figure-markdown_strict/unnamed-chunk-16-1.png)

    test_350 + geom_line(aes(x = mileage, y = price_pred), color='red', size=1.5)

![](ECO395M-HW1-_Yuxin-Feng_Jiyou-Chen_Liming-Pang_files/figure-markdown_strict/unnamed-chunk-16-2.png)

<br>

### sclass\_65\_test

<br>

    knn65 = knnreg(price ~ mileage, data=sclass_65_train, k=min_k_65[1,1])
    rmse(knn65, sclass_65_test)

    ## [1] 15279.89

    sclass_65_test = sclass_65_test %>%
      mutate(price_pred = predict(knn65, sclass_65_test))

    test_65 = ggplot(data = sclass_65_test) + 
      geom_point(mapping = aes(x = mileage, y = price), alpha=0.2)
    test_65

![](ECO395M-HW1-_Yuxin-Feng_Jiyou-Chen_Liming-Pang_files/figure-markdown_strict/unnamed-chunk-17-1.png)

    test_65 + geom_line(aes(x = mileage, y = price_pred), color='blue', size=1.5)

![](ECO395M-HW1-_Yuxin-Feng_Jiyou-Chen_Liming-Pang_files/figure-markdown_strict/unnamed-chunk-17-2.png)

<br>

#### <font color = darkslateblue> Two trims yields the same optimal k value. This mignt be because in this kind of situations, not mather the car’s trim level changes, k = 15 is always the optimal value for doing KNN. Car’s trim level might not be an influential factor .</font>

<br> <br>
