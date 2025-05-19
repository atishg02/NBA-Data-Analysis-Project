# Three-point Shooting Analysis

### Project Overview

This data analysis project explores the 

### Data Sources

The primary datasets used for this analysis are the "NBA_3pt_2025.csv" file and the "NBA_3pt_2005.csv" file. These files contain the top 5 players from each of the 30 NBA teams by three-pointer attempted. 

### Tools

- Selector Gadget - Data Collection
- R/R Studio - Data Cleaning and Data Analysis
- Overleaf - Creating reports

### Data Cleaning/Preparation

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


