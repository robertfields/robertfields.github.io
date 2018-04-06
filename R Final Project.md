---
layout: page
title: Robert Fields
permalink: R Final Project/
output:
  md_document:
    code_folding: hide
    highlight: tango
    theme: flatly
---

```{r setup, echo = TRUE, message = FALSE, include=FALSE, warning = FALSE}
knitr::opts_chunk$set(echo = TRUE)
list.of.packages <- c("tidyverse", "plotly","DT","knitr")
new.packages <- list.of.packages[!(list.of.packages %in% installed.packages()[,"Package"])]
if(length(new.packages)) install.packages(new.packages)

library(tidyverse)
library(jsonlite)
library(plotly)
library(DT)
library(knitr)
library(stringr)
```
<img src = "https://raw.githubusercontent.com/robert-fields/nba-stats/master/nba-stats-logo.JPG" height = 100 width = 320 hspace = 25 vspace = 25 align = right>

# Do the Warriors *really* play the right way?
#### Robert Fields
#### August 23, 2017

# {.tabset}
## Introduction

### What's the question?

#### Which NBA players make their teammates better?

The NBA has realized a talent boom in the last 5-10 years. Popularity, attention, and league-wide revenue has been growing, as well as punditry and data analysis. Fans of the league are riveted by such important, profound questions such as ["is Kevin Durant a cupcake?"](https://www.sbnation.com/lookit/2017/2/11/14589144/thunder-fans-are-chanting-cupcake-at-kevin-durant-heres-why)

More importantly, the NBA is on the cutting-edge of data collection in sports. Thanks to the NBA's partnership with [STATS SportVU](https://www.stats.com/sportvu-basketball-media/), they are tracking data using a system of 6 cameras hung from the ceiling that track ball and player movement 25 times per second. They can use this unstructured data to define metrics such as secondary assists, miles traveled, made field goals defended at rim, etc.

The Golden State Warriors received high praise last season as the best team ever. They are frequently said to play the *right* type of basketball, that they're unselfish, and that they make their teammates better, but can we prove this? Or disprove it?

Basketball is one of the most popular sports in America and worldwide. The strategy on and off the court is fascinating. It's my personal favorite and I'm excited that we finally have the means to answer the really complex questions that keep fans coming back.


### What data do we need?
Lucky for us, the NBA has made the data they collect readily available on their [website](stats.nba.com). Unfortunately, they do not make it readily available for download. This required some data wrangling using the `jsonlite` package.

Variables of interest include:

* Real Plus/Minus
* Points Scored
* Points Scored Against
* Player Efficiency Rating (PER)
* Usage Rates
* Effective Field Goal %
* True Shooting %
* Net Rating
* Wins

Other variables will be considered as the model is constructed. I will use data from the 2016-2017 NBA regular season and postseason.

### What methodology can we use?

I'll first try to understand how a player adds value to his team in a quantifiable way.

Then, I can establish a baseline 'sum of its parts' for overall team performance and see which teams underperform/overperform.

I can then try to predict which teams will outperform the sum of their parts, or which players make their teams better indirectly.


Ultimately, there are a few assumptions I'll have to more fully address over the course of this project:

* How am I defining making your teammates 'better'?
* How will I measure a player's impact on this definition of 'better'?
* And how do I determine the difference between one player making teammates better and synergy between teammates?
    - Comparing similar lineups with and without a particular player may shed light on this, but ultimately this question may not matter. The information is still useful in picking lineups or players that maximize a group of 5 players' value while they're on the court.




### Who cares?

Besides the aforementioned pundits and fans, basketball organizations should care, players should care, agents should care, everyone who gets paid from basketball related revenue should care. Team strategy off the court is proving critical to success. It's hard to gain an edge in a league with a salary cap and paying the best players/getting the best coach doesn't cut it anymore. Being able to answer questions like these allows teams to find value where others miss it and optimize the resources they have.

Further iterations on this question would allow teams to understand not only which players impact their teammates, but which types of teammates best suit their star players. This could help organizations tailor their teams around a foundation (e.g. how the Cavs build around Lebron James after Kyrie Irving's request for a trade, or how to build around the new pairing of Chris Paul and James Harden for the Rockets).

## Packages & Data Prep {.tabset}
### Packages Used
```
  # Packages Required
  library(tidyverse) # used for several functions in r, including dplyr, ggplot2, etc.
  library(jsonlite) # used to convert JSON files to dataframes
  library(plotly) # used for making graphs interactive and easier to interpret (especially when overplotted)
  library(DT) # used for providing a data dictionary in HTML
  library(stringr) # needed to alter some text strings to match observations
```

### Data Set {.tabset}

#### Compilation

```{r nba_data_import, echo = TRUE, message = FALSE, warning = FALSE, results = 'hide'}


# Import json files from web

# List of json URLs

json_urls <- c("http://stats.nba.com/stats/leaguedashplayerstats?College=&Conference=&Country=&DateFrom=&DateTo=&Division=&DraftPick=&DraftYear=&GameScope=&GameSegment=&Height=&LastNGames=0&LeagueID=00&Location=&MeasureType=Advanced&Month=0&OpponentTeamID=0&Outcome=&PORound=0&PaceAdjust=N&PerMode=PerGame&Period=0&PlayerExperience=&PlayerPosition=&PlusMinus=N&Rank=N&Season=2016-17&SeasonSegment=&SeasonType=Regular+Season&ShotClockRange=&StarterBench=&TeamID=0&VsConference=&VsDivision=&Weight=
",
               "http://stats.nba.com/stats/leaguedashplayerstats?College=&Conference=&Country=&DateFrom=&DateTo=&Division=&DraftPick=&DraftYear=&GameScope=&GameSegment=&Height=&LastNGames=0&LeagueID=00&Location=&MeasureType=Scoring&Month=0&OpponentTeamID=0&Outcome=&PORound=0&PaceAdjust=N&PerMode=PerGame&Period=0&PlayerExperience=&PlayerPosition=&PlusMinus=N&Rank=N&Season=2016-17&SeasonSegment=&SeasonType=Regular+Season&ShotClockRange=&StarterBench=&TeamID=0&VsConference=&VsDivision=&Weight=
",
               "http://stats.nba.com/stats/leaguedashptdefend?College=&Conference=&Country=&DateFrom=&DateTo=&DefenseCategory=2+Pointers&Division=&DraftPick=&DraftYear=&GameSegment=&Height=&LastNGames=0&LeagueID=00&Location=&Month=0&OpponentTeamID=0&Outcome=&PORound=0&PerMode=PerGame&Period=0&PlayerExperience=&PlayerPosition=&Season=2016-17&SeasonSegment=&SeasonType=Regular+Season&StarterBench=&TeamID=0&VsConference=&VsDivision=&Weight=
",
               "http://stats.nba.com/stats/leaguedashptdefend?College=&Conference=&Country=&DateFrom=&DateTo=&DefenseCategory=3+Pointers&Division=&DraftPick=&DraftYear=&GameSegment=&Height=&LastNGames=0&LeagueID=00&Location=&Month=0&OpponentTeamID=0&Outcome=&PORound=0&PerMode=PerGame&Period=0&PlayerExperience=&PlayerPosition=&Season=2016-17&SeasonSegment=&SeasonType=Regular+Season&StarterBench=&TeamID=0&VsConference=&VsDivision=&Weight=
",
               "http://stats.nba.com/stats/leaguedashptdefend?College=&Conference=&Country=&DateFrom=&DateTo=&DefenseCategory=Overall&Division=&DraftPick=&DraftYear=&GameSegment=&Height=&LastNGames=0&LeagueID=00&Location=&Month=0&OpponentTeamID=0&Outcome=&PORound=0&PerMode=PerGame&Period=0&PlayerExperience=&PlayerPosition=&Season=2016-17&SeasonSegment=&SeasonType=Regular+Season&StarterBench=&TeamID=0&VsConference=&VsDivision=&Weight=
",
               "http://stats.nba.com/stats/leaguedashplayerstats?College=&Conference=&Country=&DateFrom=&DateTo=&Division=&DraftPick=&DraftYear=&GameScope=&GameSegment=&Height=&LastNGames=0&LeagueID=00&Location=&MeasureType=Base&Month=0&OpponentTeamID=0&Outcome=&PORound=0&PaceAdjust=N&PerMode=PerGame&Period=0&PlayerExperience=&PlayerPosition=&PlusMinus=N&Rank=N&Season=2016-17&SeasonSegment=&SeasonType=Regular+Season&ShotClockRange=&StarterBench=&TeamID=0&VsConference=&VsDivision=&Weight=
","http://stats.nba.com/stats/leaguedashteamstats?Conference=&DateFrom=&DateTo=&Division=&GameScope=&GameSegment=&LastNGames=0&LeagueID=00&Location=&MeasureType=Base&Month=0&OpponentTeamID=0&Outcome=&PORound=0&PaceAdjust=N&PerMode=PerGame&Period=0&PlayerExperience=&PlayerPosition=&PlusMinus=N&Rank=N&Season=2016-17&SeasonSegment=&SeasonType=Regular+Season&ShotClockRange=&StarterBench=&TeamID=0&VsConference=&VsDivision=
",
                    "http://stats.nba.com/stats/leaguedashteamstats?Conference=&DateFrom=&DateTo=&Division=&GameScope=&GameSegment=&LastNGames=0&LeagueID=00&Location=&MeasureType=Advanced&Month=0&OpponentTeamID=0&Outcome=&PORound=0&PaceAdjust=N&PerMode=PerGame&Period=0&PlayerExperience=&PlayerPosition=&PlusMinus=N&Rank=N&Season=2016-17&SeasonSegment=&SeasonType=Regular+Season&ShotClockRange=&StarterBench=&TeamID=0&VsConference=&VsDivision=
",
                    "http://stats.nba.com/stats/leaguedashteamstats?Conference=&DateFrom=&DateTo=&Division=&GameScope=&GameSegment=&LastNGames=0&LeagueID=00&Location=&MeasureType=Four+Factors&Month=0&OpponentTeamID=0&Outcome=&PORound=0&PaceAdjust=N&PerMode=PerGame&Period=0&PlayerExperience=&PlayerPosition=&PlusMinus=N&Rank=N&Season=2016-17&SeasonSegment=&SeasonType=Regular+Season&ShotClockRange=&StarterBench=&TeamID=0&VsConference=&VsDivision=
")
json_names <- list("nba_advanced", "nba_assistedFGs", "nba_d2fgm", "nba_d3fgm", "nba_dfg", "nba_plusminus", "nba_teambasic", "nba_teamadvanced", "nba_fourfactors")

# Create Data Import Loop

data_list <- list(NULL)
for(i in seq_along(json_urls)){
  foo <- fromJSON(paste(readLines(json_urls[i])),
                  simplifyDataFrame = TRUE,
                  flatten = TRUE)
  flat_file <- flatten(foo$resultSets)
  foo <- data.frame(flat_file$rowSet)
  colnames(foo) <- t(data.frame(flat_file$headers))
  data_list[i] <- list(foo)
  print(i)
  }
data_list <- setNames(data_list, nm = json_names)

# import csv, edit column = name to match json files

nba_rpm <- read_csv(
  "https://raw.githubusercontent.com/robert-fields/nba-stats/master/nba_rpm_csv.csv",
  col_types = "_?___????")
nba_rpm$Name <- substr(nba_rpm$Name,
                       1,
                       nchar(nba_rpm$Name)-
                         (nchar(nba_rpm$Name)-str_locate(nba_rpm$Name,','))-1)

# match different spellings of names
nba_rpm$Name[nba_rpm$Name == "Nene Hilario"] <- "Nene"
nba_rpm$Name[nba_rpm$Name == "Sheldon Mac"] <- "Sheldon McClellan"

# Merge relevant variables from json files and csv and filter out NA from csv

player_data <-
  data_list[[1]] %>%
  left_join(data_list[[2]][,c("PLAYER_ID","PCT_AST_FGM","PCT_UAST_FGM")],
          by = "PLAYER_ID") %>%
  left_join(data_list[[3]][,c("PLAYER_NAME","FG2M")],
            by = "PLAYER_NAME") %>%
  left_join(data_list[[4]][,c("PLAYER_NAME","FG3M")],
            by = "PLAYER_NAME") %>%
  left_join(data_list[[5]][,c("PLAYER_NAME","D_FG_PCT")],
            by = "PLAYER_NAME") %>%
  left_join(data_list[[6]][,c("PLAYER_ID","PLUS_MINUS")],
            by = "PLAYER_ID") %>%
  left_join(nba_rpm, by = c("PLAYER_NAME" = "Name")) %>%
  filter(!is.na(RPM))

# Merge team data

team_data <-
  data_list[[9]] %>%
  left_join(data_list[[7]][,c("TEAM_ID","PLUS_MINUS")],
            by = "TEAM_ID") %>%
  left_join(data_list[[8]][,c("TEAM_ID","TS_PCT","PIE",
                              "NET_RATING","OFF_RATING","DEF_RATING",
                              "AST_PCT","AST_TO","AST_RATIO")],
            by = "TEAM_ID")

# clean up workspace
rm(list = setdiff(ls(), c("player_data","team_data")))

# change vectors to numerics
player_data[5:67] <- sapply(player_data[5:67],
  function(x) as.numeric(as.character(x)))

team_data[3:39] <- sapply(team_data[3:39],
                            function(x) as.numeric(as.character(x)))

# filter out unneeded variables, match team and player data, add variables
player_data <-
  player_data %>%
  select(-(GP_RANK:CFPARAMS)) %>%
  left_join(mutate(team_data,
                   TEAM_PIE = PIE,
                   TEAM_PLUS_MINUS = PLUS_MINUS,
                   TEAM_WINS = W,
                   TEAM_MIN = MIN,
                   TEAM_NET_RATING = NET_RATING,
                   TEAM_OFF_RATING = OFF_RATING,
                   TEAM_DEF_RATING = DEF_RATING)
            [,c("TEAM_ID","TEAM_PIE","TEAM_PLUS_MINUS","TEAM_WINS","TEAM_MIN",
                "TEAM_NET_RATING","TEAM_OFF_RATING","TEAM_DEF_RATING")],
       by = "TEAM_ID") %>%
       mutate(TEAM_RPM_SHARE = (RPM * (GP * MIN) / (TEAM_MIN)))

 team_data <-
   team_data %>%
   select(-(GP_RANK:CFPARAMS)) %>%
   left_join(
     aggregate(player_data$TEAM_RPM_SHARE ~ player_data$TEAM_ID,
       player_data, sum),
       by = c("TEAM_ID" = "player_data$TEAM_ID")) %>%
   rename("TEAM_RPM" = "player_data$TEAM_RPM_SHARE") %>%
   left_join(
             aggregate(player_data$WINS ~ player_data$TEAM_ID,player_data, sum),
             by = c("TEAM_ID" = "player_data$TEAM_ID")) %>%
   rename("TEAM_RPM_WINS" = "player_data$WINS")

 player_data <- player_data %>%
  left_join(mutate(team_data,
            TEAM_RPM_WINS = TEAM_RPM_WINS,
            TEAM_RPM = TEAM_RPM)[,c("TEAM_ID","TEAM_RPM_WINS",
                                    "TEAM_RPM")],
            by = "TEAM_ID")

```

I have included a data dictionary with details on each variable below each data set on their respective tabs. A more in depth look at different statistics used for NBA data can be found in [the NBA stats glossary](stats.nba.com/help/glossary).

I had to compile two data sets for **Team** data and **Player** data. I did this by linking to the JSON URL from the [stats.nba.com](stats.nba.com) website HTML and using the "jsonlite" package to scrape this data and flatten it into data frames. I used a loop to tidy up this repetitive process.

```
data_list <- list(NULL)
for(i in seq_along(json_urls)){
  foo <- fromJSON(paste(readLines(json_urls[i])),
                  simplifyDataFrame = TRUE,
                  flatten = TRUE)
  flat_file <- flatten(foo$resultSets)
  foo <- data.frame(flat_file$rowSet)
  colnames(foo) <- t(data.frame(flat_file$headers))
  data_list[i] <- list(foo)
  print(i)
  }
data_list <- setNames(data_list, nm = json_names)
```

I combined all of these data frames into two single data sets. I also pulled Real Plus/Minus data from ESPN, converted it into a .csv and imported that. I was able to merge all the JSON dataframes and the csv based on either a Player ID from nba.com or the Player Name. I also joined data from the `team_data` set to the `player_data` set to make some comparisons between teams and players.

I created a variable `TEAM_RPM_SHARE` in the `player_data` set that weighted a player's `RPM` by the minutes he played during the season. I then aggregated this by team to create a `TEAM_RPM` variable in the `team_data` set. This is the variable I use as a baseline for team performance based on the players they have on their team. I did the same thing with RPM `WINS`, creating the `TEAM_RPM_WINS` variable in the `team_data` set.

```
mutate(player_data, TEAM_RPM_SHARE = (RPM * (GP * MIN) / (TEAM_MIN)))

left_join(team_data,
     aggregate(player_data$TEAM_RPM_SHARE ~ player_data$TEAM_ID,
       player_data, sum),
       by = c("TEAM_ID" = "player_data$TEAM_ID")) %>%
   rename("TEAM_RPM" = "player_data$TEAM_RPM_SHARE")
```

I dropped any unneccessary variables, paring it down to 39 variables for **Team** data and 67 variables for **Player** data that might be of interest later on.

``` {r echo = FALSE, warnings = FALSE, message = FALSE}
glimpse(team_data)
glimpse(player_data)
```


#### Player Data

``` {r player_datatable, echo = TRUE, message = FALSE, warning = FALSE}
datatable(player_data,
          extensions = c("FixedColumns","Buttons","KeyTable","Scroller"),
          options = list(orderClasses = TRUE,
                         fixedColumns = list(leftColumns = 5),
                         dom = "Bfrtip",
                         buttons = c("copy","csv","excel","colvis"),
                         keys = TRUE,
                         deferRender = FALSE,
                         scrollY = 750,
                         scrollX = 1000,
                         scroller = TRUE,
                         filter = "top",
                         orderMulti = TRUE,
                         columnDefs =
                           list(list(visible = FALSE, targets = c(1,3,5:50)))))

```



```{r player_data_dictionary, echo = TRUE, message=FALSE, warning = FALSE}
#Creating Data Dictionary

VarType<- c("PLAYER_NAME",
            "TEAM_ABBREVIATION",
            "GP",
            "W",
            "L",
            "MIN",
            "OFF_RATING",
            "DEF_RATING",
            "NET_RATING",
            "AST_PCT",
            "AST_TO",
            "AST_RATIO",
            "OREB_PCT",
            "DREB_PCT",
            "REB_PCT",
            "TM_TOV_PCT",
            "EFG_PCT",
            "TS_PCT",
            "USG_PCT",
            "PACE",
            "PIE",
            "FG_PCT",
            "PCT_UAST_FGM",
            "DFG2M",
            "DFG3M",
            "D_FG_PCT",
            "PLUS_MINUS",
            "ORPM",
            "DRPM",
            "RPM",
            "WINS",
            "TEAM_PIE",
            "TEAM_PLUS_MINUS",
            "TEAM_WINS",
            "TEAM_RPM_WINS",
            "TEAM_RPM")

VarDesc<-c("Player Name",
           "3-letter abbreviation of team name",
           "Games Played",
           "Wins",
           "Losses",
           "Minutes Played",
           "Offensive Rating (points per 100 possessions while player is on court)",
           "Defensive Rating (points allowed per 100 possesions while player is on court)",
           "Net Rating (point differential per 100 possessions while player is on court)",
           "Assist Percentage (percentage of shots a player assists (AST / TmFGM - FGM))",
           "Assist to Turnover Ratio",
           "Assist Ratio (number of assists per 100 possessions used)",
           "Offensive Rebound Percentage (percentage of available offensive rebounds a player grabbed while on floor)",
           "Defensive Rebound Percentage (percentage of available defensive rebounds a player grabbed while on floor)",
           "Overall Rebound Percentage (percentage of total rebounds a player grabbed while on floor)",
           "Team Turnover Percentage (turnovers per 100 possessions used by a player)",
           "Effective Field Goal Percentage (adjusts field goal percentage based on higher expected value of 3-pointers)",
           "True Shooting Percentage (metric of shooting efficiency (points / points possible on possessions with FG or FT attempt))",
           "Usage Rate (possessions used by a player when on the floor ((FGA + (.44*FTA) + TO) / Possessions)))",
           "Pace of play (number of possessions per 48 minutes)",
           "Player Impact Estimate (comparable to other advanced statistic ratings e.g. PER)",
           "Field Goal Percentage",
           "Percentage of FGs made that were unassisted",
           "Made 2-point FGs a player defended",
           "Made 3-point FGs a player defended",
           "Defensive Field Goal Percentage",
           "Plus Minus (point differential while player is on the floor)",
           "Offensive Real Plus Minus",
           "Defensive Real Plus Minus",
           "Real Plus Minus",
           "RPM Wins (similar to win share, or number of wins a player contributes to his team's win total)",
           "Team Player Impact Estimate (overall PIE of the team a player is on)",
           "Team Plus Minus (Overall Plus Minus of team a player is on)",
           "Team Wins (overall wins of the team a player is on, including games in which a player did not play)",
           "Aggregate of RPM Wins (predicted wins a player contributes to a team)",
           "Aggregate of RPM for all players on a team, weighted by minutes played)")
Data_Dictionary<- as.data.frame(cbind(VarType,VarDesc))
colnames(Data_Dictionary)<-c("Variable Name","Variable Description")
kable(Data_Dictionary, caption = "Data Dictionary of relevant variables")
```

#### Team Data

``` {r team_datatable, echo = TRUE, message = FALSE, warning = FALSE}
datatable(team_data,
          extensions = c("FixedColumns","Buttons","KeyTable","Scroller"),
          options = list(orderClasses = TRUE,
                         fixedColumns = list(leftColumns = 3),
                         dom = "Bfrtip",
                         buttons = c("copy","csv","excel","colvis"),
                         keys = TRUE,
                         deferRender = FALSE,
                         scrollY = 750,
                         scrollX = 1000,
                         scroller = TRUE,
                         filter = "top",
                         orderMulti = TRUE,
                         columnDefs =
                           list(list(visible = FALSE, targets = c(1,3:26)))))

```



```{r team_data_dictionary, echo = TRUE, message = FALSE, warning = FALSE}
VarType<- c("TEAM_NAME",
            "W",
            "L",
            "W_PCT",
            "MIN",
            "EFG_PCT",
            "TM_TOV_PCT",
            "OREB_PCT",
            "OPP_EFG_PCT",
            "OPP_TOV_PCT",
            "OPP_OREB_PCT",
            "PLUS_MINUS",
            "TS_PCT",
            "PIE",
            "NET_RATING",
            "AST_PCT",
            "AST_TO",
            "AST_RATIO",
            "TEAM_RPM")

VarDesc<-c("Team Name",
           "Wins",
           "Losses",
           "Win Percentage",
           "Minutes",
           "Effective Field Goal Percentage (adjusts field goal percentage based on higher expected value of 3-pointers)",
           "Team Turnover Percentage (turnovers per 100 possessions)",
           "Offensive Rebound Percentage (percentage of available offensive rebounds a team grabs)",
           "Opponent's Effective Field Goal Percentage",
           "Opponent's Turnover Percentage",
           "Opponent's Offensive Rebound Percentage",
           "Plus Minus (overall point differential)",
           "True Shooting Percentage (metric of shooting efficiency (points / points possible on possessions with FG or FT attempt))",
           "Player Impact Estimate",
           "Net Rating",
           "Assist Percentage",
           "Assist to Turnovers",
           "Assist to Turnover Ratio",
           "Team Real Plus Minus")
Data_Dictionary<- as.data.frame(cbind(VarType,VarDesc))
colnames(Data_Dictionary)<-c("Variable Name","Description")
kable(Data_Dictionary, caption = "Data Dictionary of relevant variables")

```

## Analysis {.tabset}
### Data Exploration {.tabset}
#### Talent Distribution

Some preliminary analysis shows that there are noticeable differences in the distributions of player talent across teams, which should be expected.

This follows conventional wisdom (the Warriors are good, the Sixers not so much), but there are a few interesting points that deserve further analysis. Such as whether the Warriors are better or worse than the sum of their parts (comparing the sum of player `RPM`, a predicted value, vs. `NET_RATING`, an observed value), and how other teams compare in the same context.

``` {r exploratory_analysis, echo = TRUE, warning = FALSE, message = FALSE}
# boxplot of player RPM on each team
ggplotly(
  player_data %>%
  arrange(desc(TEAM_PLUS_MINUS)) %>%
  ggplot(aes(TEAM_ABBREVIATION, RPM)) +
    geom_boxplot())

# density plot of player RPM on each team
ggplotly(
  player_data %>%
  arrange(desc(TEAM_PLUS_MINUS)) %>%
  ggplot(aes(RPM, fill = TEAM_ABBREVIATION)) +
  geom_density(alpha = .2))
```

#### Team Performance vs. Baseline

In thinking through approaching this analysis, I settled on two different questions:

1. How do teams perform against a "sum of their parts" baseline?
2. Which players are responsible for this performance?

For the first question, I compared `TEAM_RPM` vs. `NET_RATING` for each team. I also compared `TEAM_RPM_WINS` vs. `W`. The RPM statistics are produced by ESPN and adjusted to be a representation of a player's contribution to his team, independent of teammates or opponents. Therefore, when aggregated these statistics should represent the sum of each player's individual contribution.

I plotted both of these with a line with slope = 1 (a null hypothesis that teams equal the sum of their parts). I also plotted these with a `geom_smooth` line of best fit to show how teams might compare to other teams in the league.

```{r analysis_q1, echo = TRUE, warning = FALSE, message = FALSE}
ggplotly(
  ggplot(team_data,aes(TEAM_RPM,NET_RATING)) +
    geom_point(aes(color = TEAM_NAME)) +
    geom_smooth(method = "glm") +
    geom_abline()
)
ggplotly(
  ggplot(team_data,aes(TEAM_RPM_WINS,W)) +
    geom_point(aes(color = TEAM_NAME)) +
    geom_smooth(method = "glm") +
    geom_abline())
```

These graphs indicate that teams like the Utah Jazz and San Antonio Spurs may be some of the bigger overachievers.

The `NET_RATING` chart suggests that while the Warriors might not be maximizing the sum of the talent on that team, they are still doing better than league average.

It is also worth observing where the slope = 1 line and the best fit line intersect, suggesting that it's easier for bad teams to outperform their talent level. Additionally, the gap is much closer when observing `NET_RATING` (a points measure) instead of wins, suggesting that point differentials are better representations of talent levels (but ultimately, wins are still what teams are after).

This might suggest there are diminishing marginal returns to talent and how efficiently it translates to points/wins.


### Next Steps

I'd like to build a model that looks into what players do on the court (i.e. shooting, passing, defensive statistics, etc.) and how this contributes to the synergy of a team. Furthermore, I'd like to see which player lineups are most effective at pushing a team past its "sum of its parts" baseline.

Potential ideas to explore:

* On/Off numbers for each player when they're on or off the court
    - Points, Net Rating, Rebounds, Assists, etc.
* Hierarchical models that explore the overall impacts of a player and sets of players
* Factor Analysis on game statistics and their impact on winning
* Clustering on Players/Lineups/Teams to determine if there are NBA 'Archetypes' of successful teams
* Differences in the Regular season vs. Postseason

This is an on-going project that I will continue to work on in my spare time.
