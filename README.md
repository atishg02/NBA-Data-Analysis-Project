# Three-point Shooting Analysis

## Table of Contents

- [Project Overview](#project-overview)
- [Data Sources](data-sources)
- [Tools](Tools)
- [Data Cleaning and Preparation](Data Cleaning-and-preparation)

### Project Overview

This data analysis project explores the rise of three-point shooting taking over the NBA. We will investigate whether or not this revolution is as dramatic as it seems, and whether or not players are getting better at shooting three-pointers. Our motivation for this project stems from league-wide calls to institute new rules for three-pointer as it's seen as "over-powered" and not properly balanced. 

### Data Sources

The primary datasets used for this analysis are the "NBA_3pt_2025.csv" file and the "NBA_3pt_2005.csv" file. These files contain the top 5 players from each of the 30 NBA teams by three-pointers attempted. 

### Tools

- Selector Gadget - Data Collection
- R- Data Cleaning and Data Analysis
- Overleaf - Creating reports

### Data Cleaning and Preparation

For the initial data preparation phase, we did the following things: 
1. Data loading and inspection.
2. Data cleaning and formatting. 

### Exploratory Data Analysis

EDA involved exploring the NBA data to answer key questions, such as:

- Has there been a dramatic increase or decrease in three-pointers _attempted_ between 2005 and 2025?
- Has there been an increase or decrease in three-pointers _made_ between 2005 and 2025?
- Are the top NBA three-point shooters shooting better in 2005 or 2025?

### Data Analysis

*SCRAPING OUR DATA FOR 2025*
```R
library(rvest)
library(dplyr)

url <- "https://www.basketball-reference.com/leagues/NBA_2025_totals.html"
page <- read_html(url)

player_stats <- page %>%
  html_element("table#totals_stats") %>%
  html_table()

top_3pt_players_df <- player_stats %>%
  filter(!is.na(Team), !grepl("TM", Team)) %>%
  mutate(
  `3PA` = as.numeric(`3PA`),
  `3P` = as.numeric(`3P`)
) %>%
group_by(Team) %>%
arrange(desc(`3PA`), .by_group = TRUE) %>%
slice_head(n = 5) %>%
ungroup() %>%
dplyr::select(Player, Team, `3P`, `3PA`) %>%
as.data.frame()

top_3pt_players_df <- top_3pt_players_df[-1, ]

print(top_3pt_players_df)
```

*SCRAPING OUR DATA FOR 2005*
```R
library(rvest)
library(dplyr)

url <- "https://www.basketball-reference.com/leagues/NBA_2005_totals.html"
page <- read_html(url)

player_stats <- page %>%
html_element("table#totals_stats") %>%
html_table()

names(player_stats) <- trimws(names(player_stats))

top_3pt_players_df_2005 <- player_stats %>%
  filter(!is.na(Team), !grepl("TM", Team)) %>%
  mutate(
  `3PA` = as.numeric(`3PA`),
  `3P` = as.numeric(`3P`)
) %>%
group_by(Team) %>%
arrange(desc(`3PA`), .by_group = TRUE) %>%
slice_head(n = 5) %>%
ungroup() %>%
dplyr::select(Player, Team, `3P`, `3PA`) %>%
as.data.frame()

top_3pt_players_df_2005 <- top_3pt_players_df_2005[-1, ]

print(top_3pt_players_df_2005)
```

### Results/Findings

The analysis results are summarized as follows:
1. The NBA has seen a rise in three-point shooting within the last 20 years, however, three-point percentage has not significantly risen.
2. There is a trend toward position-less basketball, where every player is expected to be proficient in three-point shooting. This is evident in the overall density of three-point percentage between 2005 and 2025.

### Limitations

Two seasons alone cannot guarantee persistence of trends; future analyses should model full year-by-year trends. These two seasons were selected in hopes this would make the data more digestible and easier to understand, but it comes with limitations. Just as one team per season can easily be an outlier, there is nothing to say that one or both of these seasons aren’t themselves outliers. It therefore has the ability to skew our conclusions on overall trends in the league. Also, this paper only examines the three-point shot. There are other methods of scoring in a basketball game, such as the two-point shot and a free throw. Variables like win-percentage or standing in the overall league could provide valuable insight into the actual effectiveness of the three-point shot.

### References

1. Basketball Reference. ”NBA Player Three-Point Shooting Stats,” https://www.basketball-reference. com/.
2. Stan Development Team. ”Stan Modeling Language Users Guide and Reference Manual,” https: //mc-stan.org.
3. Syracuse University News. ”Deflation: Study Shows NBA 3-Point Shot Has Lost Its Value,” https:// news.syr.edu/blog/2024/02/09/deflation-study-shows-nba-3-point-shot-has-lost-its-value/.
4. FOX Sports. ”NBA commissioner Adam Silver vows to fix 3-point shooting issue” https://www. foxsports.com/stories/nba/nba-commissioner-adam-silver-vows-fix-3-point-shooting-issue-we-it.
5. Overleaf, online latex editor. Overleaf, Online LaTeX Editor. (n.d.). https://www.overleaf.com/ 
