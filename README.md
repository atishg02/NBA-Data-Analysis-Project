# Three-point Shooting Analysis

## Table of Contents

- [Project Overview](#project-overview)
- [Data Sources](#data-sources)
- [Tools](#tools)
- [Data Cleaning and Preparation](#data-cleaning-and-preparation)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Data Analysis](#data-analysis)
- [Results and Findings](#results-and-findings)
- [Limitations](#limitations)
- [References](#references)

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

*Generating our Posterior Means for 2025*
```R
library(rstan)
library(dplyr)
library(ggplot2)

# Stan model code as a string
stan_model_code <- "
data {
  int<lower=1> J;
  int<lower=1> N;
  int<lower=1, upper=J> team[N];
  int<lower=0> y[N];
  int<lower=0> n[N];
}
parameters {
  real<lower=0> alpha;
  real<lower=0> beta;
  vector<lower=0, upper=1>[J] theta;
}
model {
  alpha ~ exponential(1);
  beta ~ exponential(1);
  for (j in 1:J)
    theta[j] ~ beta(alpha, beta);
  for (i in 1:N)
    y[i] ~ binomial(n[i], theta[team[i]]);
}
"

# Prepare 2025 data
data_2025 <- combined_df %>% filter(Season == "2025")
team_factor <- factor(data_2025$Team)
team_ids <- as.numeric(team_factor)
team_name_map <- levels(team_factor)

stan_data <- list(
  J = length(team_name_map),
  N = nrow(data_2025),
  team = team_ids,
  y = data_2025$`3P`,
  n = data_2025$`3PA`
)

# Fit Stan model
rstan_options(auto_write = TRUE)
options(mc.cores = parallel::detectCores())

fit <- stan(
  model_code = stan_model_code,
  data = stan_data,
  iter = 2000,
  chains = 4,
  seed = 123
)


team_map <- data.frame(
  Team = unique(data_2025$Team),
  PosteriorMean3P = theta_means
)

print(team_map)

# Extract posterior means and match to team names
posterior_samples <- extract(fit)
theta_means <- apply(posterior_samples$theta, 2, mean)

posterior_df <- data.frame(
  Team = team_name_map,
  PosteriorMean3P = theta_means
)

# Plot posterior mean 3P% per team
ggplot(posterior_df, aes(x = reorder(Team, PosteriorMean3P), y = PosteriorMean3P)) +
  geom_col(fill = "steelblue") +
  coord_flip() +
  labs(title = "Posterior Mean Team 3P% (2025)", x = "Team", y = "Estimated 3P%") +
  theme_minimal()
```

### Results and Findings

The analysis results are summarized as follows:
1. The NBA has seen a rise in three-point shooting within the last 20 years, however, three-point percentage has not significantly risen.
2. There is a trend toward position-less basketball, where every player is expected to be proficient in three-point shooting. This is evident in the overall density of three-point percentage between 2005 and 2025.

### Limitations

Two seasons alone cannot guarantee persistence of trends; future analyses should model full year-by-year trends. These two seasons were selected in hopes this would make the data more digestible and easier to understand, but it comes with limitations. Just as one team per season can easily be an outlier, there is nothing to say that one or both of these seasons aren’t themselves outliers. It therefore has the ability to skew our conclusions on overall trends in the league. Also, this paper only examines the three-point shot. There are other methods of scoring in a basketball game, such as the two-point shot and a free throw. Variables like win-percentage or standing in the overall league could provide valuable insight into the actual effectiveness of the three-point shot.

### References

1. Basketball Reference. ”NBA Player Three-Point Shooting Stats,” https://www.basketball-reference.com/.
2. Stan Development Team. ”Stan Modeling Language Users Guide and Reference Manual,” https://mc-stan.org.
3. Syracuse University News. ”Deflation: Study Shows NBA 3-Point Shot Has Lost Its Value,” https://news.syr.edu/blog/2024/02/09/deflation-study-shows-nba-3-point-shot-has-lost-its-value/.
4. FOX Sports. ”NBA commissioner Adam Silver vows to fix 3-point shooting issue” https://www.foxsports.com/stories/nba/nba-commissioner-adam-silver-vows-fix-3-point-shooting-issue-we-it.
5. Overleaf, online latex editor. Overleaf, Online LaTeX Editor. (n.d.). https://www.overleaf.com/ 
