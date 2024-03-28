# 🛖AGE OF EMPIRES 4 RANKED STATISTICAL ANALYSIS

Age Of Empires 4 is a complex real-time strategy game set in the middle-ages where players require immense knowledge about many mechanics of the game to perform competitively. At the time of the most recent season (S7), AOE4 has 16 unique civilizations and ~30 maps; each civilization with unique bonuses, units, upgrades, and approaches to gameplay, each map with unique resources, land/sea formations, and a range of geographical difference. 

## Project Overview
- [AOE 4 STATS DASHBOARD ON TABLEAU]()    
- [JUMP ➡️ TREND ANALYSIS]()    
- [JUMP ➡️ LINEAR REGRESSION MODEL]()    

#### ❓The problem
Parts of the AOE4 community have expressed their concerns about a variety of things: some civilizations are too strong, some maps are overtly advantageous to some civilizations, and some unique upgrades of exclusive units are overpowered. 
Arguments the balance of civilziations are ongoing, and with every patch, the developers buff or nerf different parts of the game with the goal to balance the game so it is fair to all players of all levels. **But with so much emotional biases in ranked play and many game factors to consider, how can we determine if the game really is balanced or not?**


#### 📝The approach For a Solution
With the start of each season, all players' ranked ladder title and ratings from the previous season are cleared to zero. Meaning analyzing season 5 (6-16-23 to 11-14-23) and season 6 (11-14-23 to 3-20-24) match and player data can provide us with very valuable insights into the balancing of the game and give an idea of what the current season 7 might look like. With visualization inspiration from [AOE4 WORLD](https://aoe4world.com/), I will do a deep dive into these categories for seasons 5 and 6 ranked play match data:
- Player distribution in solo queue by rank title/points    
- Player win rates in solo queue by rank title/points    
- Civilization win rates in ranked 1v1 by rank title/points    
- Civilization overall matchup win rates in 1v1 ranked    
- Civilization win rates in 1v1 ranked by maps     
- Civilization pick rate in 1v1 ranked by rank title/points     
- Average Game duration in 1v1 ranked by rank title/points    



## 🎮Data Source:
1. Season 5-6 JSON 1v1 ranked game data on [AOE4 WORLD DATA DUMP!](https://aoe4world.com/dumps)   
   - Contains every ranked 1v1 ladder game played in season 5-6 with data on time of matches, players, maps, civilizations, and other relevant information.
2. Data scraped: season 5-6 player ratings and rankings
   - Season 5 leaderboards scraped from [AOE4 WORLD SEASON 5 LEADERBOARDS](https://aoe4world.com/leaderboard/rm_solo?season=5)
   - Season 6 leaderboards scraped from [AOE4 WORLD SEASON 6 LEADERBOARDS](https://aoe4world.com/leaderboard/rm_solo?season=6)
   - Contains information on all ranked players' placement and summary of stats at the end of season 5 and season 6 before reset


## 🔨Tools:
1. R for data scraping, preparing, cleaning, and analyzing statistics
   - ```library("tidyverse")```
   - ```library("data.table")```
   - ```library("magrittr")```
   - ```library("reshape2")```
2. Tableau for informative dashboard creation, visualizations, and trend discovery

### Data Preperation & Cleaning
*Note: Only S5 data cleaning & analysis code will be documented on github, the process for S6 data analysis is almost identical*   

The JSON files from AOE4 World contained a lot of nested lists. Turning the files into data frames were easy however some columns had 3 layers of nested dictionaries which required extraction and seperation.    

#### The following is the process of turning layers of nested dictionaries (season 5 JSON files) into a readable dataframe and renaming/reordering for analysis.
```R
# Load all packages needed
install.packages("rjson")
library("rjson")
library("tidyverse")
library("data.table")
library("magrittr")
library("reshape2")

# Read the json file 
jdata_s5 = fromJSON(file ='C://Users//gwu//Desktop//GA DataA//R-Project Files//AOE 4//Raw RM_1v1_JSON files//games_rm_1v1_s5.json')

# Turn large list from json file into data frame
raw_df_s5 = as.data.frame(do.call(rbind,jdata_s5))

# Teams column has the data of both players in a nested list of lists of dictionaries
#get the list of teams separate from the rest of the df
teams_s5 = raw_df_s5$teams

# Separate player 1 and player 2 from teams nested lists
p1_s5 = lapply(teams_s5,function(l) l[[1]][[1]])
p2_s5 = lapply(teams_s5,function(l) l[[2]][[1]])

# Player 1 and player 2 are now lists of dictionaries,separate them into dfs
p1_s5_df = rbindlist(p1_s5)
p2_s5_df = rbindlist(p2_s5)

# Rename columns to identify p1 and p2
p1_s5_df <- p1_s5_df %>%
  dplyr::rename("p1 ID" = profile_id, "p1 result" = result, 
         "p1 civ" = civilization,"p1 civ randomized" = civilization_randomized,
         "p1 mmr" = mmr, "p1 mmr dif" = mmr_diff)

p2_s5_df <- p2_s5_df %>%
  dplyr::rename("p2 ID" = profile_id, "p2 result" = result, 
         "p2 civ" = civilization,"p2 civ randomized" = civilization_randomized,
         "p2 mmr" = mmr, "p2 mmr dif" = mmr_diff)

# Drop rating and rating diff columns as they are all NA
p1_s5_df = p1_s5_df %>%
  subset(select = -c(rating,rating_diff))

p2_s5_df = p2_s5_df %>%
  subset(select = -c(rating,rating_diff))

# Drop unneeded columns from raw_df_s5 to combine p1 and p2 with overall data
dropped_df_s5 = raw_df_s5 %>%
  subset(select = -c(teams,kind,patch))

# Add player1, player2, and game dfs together
player_df_s5 = cbind(p1_s5_df,p2_s5_df)
RM_1v1_s5 = cbind(dropped_df_s5, player_df_s5)

RM_1v1_s5 = unnest(RM_1v1_s5)

write_csv(RM_1v1_s5,"C:\\Users\\gwu\\Desktop\\GA DataA\\R-Project files\\AOE 4\\Cleaned RM 1v1 CSV Files\\AOE4_RM_1v1_s5.csv")
```
Unused columns were dopped, like "patch" and "kind", every game in both datasets were ranked 1v1.

The data sets were validated for games within the range of the seasons. 4 matches were excluded due to NA and unknown values.

#### Merging player leader boards data with match data to get one big dataframe, validating and then renaming/reordering columns for organization.

```R
# Read data
match = read.csv("C:\\Users\\gwu\\Desktop\\GA DataA\\R Project files\\AOE 4\\Cleaned RM 1v1 CSVs\\AOE4_RM_1v1_s5.csv")
lb = read.csv("C:\\Users\\gwu\\Desktop\\GA DataA\\R Project files\\AOE 4\\Raw Leaderboard Excel Files\\leadersboards_rm_solo_points_s5.csv.gzip.csv")

# Drop mmr columns because missing values
match = subset(match, select = -c(p1.mmr, p1.mmr.dif, p2.mmr, p2.mmr.dif))

# Season 5 solo queue player win rates by rank title/points
lb$winrate = (lb$wins_count/lb$games_count)*100 #calculate win rate
lb$group = sub("\\_.*","",lb$rank_level) #group general rank levels

# Display the winning civilization of each match
match$civ.won <- ifelse(match$p1.result == "win", match$p1.civ, match$p2.civ)


## Find out each civilization pick and win rate by rank groups
# Separate p1 and p2 match data because need to merge by profile ID and each row has both players' ID
p1_match = subset(match, select = c(game_id,duration,map,p1.ID,p1.result,p1.civ,civ.won))
p2_match = subset(match, select = c(game_id,duration,map,p2.ID,p2.result,p2.civ,civ.won))

# Merge matches with each player information from leader boards (lb)
p1_match_lb = merge(p1_match,lb, by.x = 'p1.ID', by.y = 'profile_id')
p2_match_lb = merge(p2_match,lb, by.x = 'p2.ID', by.y = 'profile_id')

# Merge both dfs by game id, now have player rank info and match info in one df.
all = merge(p1_match_lb,p2_match_lb, by = 'game_id')

# Clean and organize naming/ordering
all = subset(all, select = c(game_id,duration.x,map.x,name.x,p1.result,p1.civ,rank.x,
                             rating.x,group.x,games_count.x,winrate.x,name.y,
                             p2.result,p2.civ,rank.y,rating.y,group.y,games_count.y,
                             winrate.y, civ.won.x)) %>% rename(
                               game.id = game_id,duration.sec = duration.x, map = map.x, p1.name = name.x,
                               p1.rank = rank.x, p1.rating = rating.x, p1.rank.group = group.x,
                               p1.games.count = games_count.x, p1.winrate = winrate.x,
                               p2.name = name.y, p2.rank = rank.y, p2.rating = rating.y,
                               p2.rank.group = group.y, p2.games.count = games_count.y,
                               p2.winrate = winrate.y, civ.won = civ.won.x
                             )

write.csv(all,file = "C:\\Users\\gwu\\Desktop\\GA DataA\\R Project files\\AOE 4\\Season 5 Analysis\\AOE4 S5 1V1 RANKED DATA.csv")
```


### Data Analysis - [AOE4 STATS DASHBOARD]()
*Note: Most analysis and callculations were done in R, then exported out to Tableau for visualizations and dashboard creation to study trends.*

#### The 2 players in each match can differ in rank group, meaning that gold ranked players can play with silver or even platinum players. With only 6 rank groups in the ranked ladder, a one rank difference is a lot. *Meaning that players who rank higher than their opponents are more likely to win because of individual skill and not necessarily because of civilization strengths and match-up.*    

To eliminate bias, when calculating win rates for civilizations by ranks and maps, only matches where both players are the same rank are included.

#### Civilization win rate in each rank group where both players are in the same rank group for each match.
```R
## Calculate win rate for each civilization in each rank group where both players are the same rank
# subset of all games where both players have the same rank group
all_same_rank = subset(all, p1.rank.group == p2.rank.group)

write.csv(all_same_rank, file = 'C:\\Users\\gwu\\Desktop\\GA DataA\\R Project files\\AOE 4\\Season 5 Analysis\\S5 1v1 SAME RANK.CSV')

# Create a function to calculate civ win rates for each player
calculate_win_rate <- function(civ, rank_group) {
  total_games <- sum((all_same_rank$p1.civ == civ & all_same_rank$p1.rank.group == rank_group) | (all_same_rank$p2.civ == civ & all_same_rank$p2.rank.group == rank_group))
  wins <- sum(all_same_rank$civ.won == civ & (all_same_rank$p1.civ == civ & all_same_rank$p1.rank.group == rank_group | all_same_rank$p2.civ == civ & all_same_rank$p2.rank.group == rank_group))
  if(total_games > 0) {
    win_rate <- (wins / total_games) * 100
  } else {
    win_rate <- 0
  }
  return(win_rate)
}

# Get unique civilizations and rank groups
unique_civs <- unique(c(all_same_rank$p1.civ, all_same_rank$p2.civ))
unique_rank_groups <- unique(c(all_same_rank$p1.rank.group, all_same_rank$p2.rank.group))

# Create a matrix to store win rates
win_rate_matrix <- matrix(NA, nrow = length(unique_civs), ncol = length(unique_rank_groups))
rownames(win_rate_matrix) <- unique_civs
colnames(win_rate_matrix) <- unique_rank_groups

# Fill win rate matrix
for (civ in unique_civs) {
  for (rank_group in unique_rank_groups) {
    win_rate_matrix[civ, rank_group] <- calculate_win_rate(civ, rank_group)
  }
}

# Melt the win rate matrix into long format
melted_df <- melt(win_rate_matrix, varnames = c("civ", "rank.group"), value.name = "win.rate")

# Convert melted df to a df with 'rank', 'civ', and 'winrate' as columns
win_rate_df <- data.frame(
  rank = melted_df$rank.group,
  civ = melted_df$civ,
  winrate = melted_df$win.rate
)

write.csv(win_rate_df, file = 'C:\\Users\\gwu\\Desktop\\GA DataA\\R Project files\\AOE 4\\Season 5 Analysis\\S5 CIV WINRATE BY RANK.CSV')
```

#### Civilization pick rate in each rank group (all match data used).
```R
## Civilization pick rate in each rank group using all games

all_long <- all %>%
  pivot_longer(matches("^p\\d"),
               names_pattern = "(p\\d).(.*)", names_to = c("player", ".value"))

#pick count for each civ in each rank group
civ_pick_counts <- all_long %>%
  group_by(rank.group, civ) %>%
  summarize(pick_count = sum(!is.na(civ)), .groups = "drop")

civ_pickrate <- (civ_pick_counts %>%
                   group_by(rank.group) %>%
                   mutate(total_picks = sum(pick_count)) %>%
                   ungroup()) %>% mutate(pick_rate = pick_count / total_picks)

write.csv(civ_pickrate, file = "C:\\Users\\gwu\\Desktop\\GA DataA\\R Project files\\AOE 4\\Season 5 Analysis\\S5 CIV PICKRATE BY RANK.CSV")
```

#### Civilization win rate in each rank group on each map where both players are in the same rank group for each match.
```R
## Civilization win rates for all rank groups on each map
# Create a function to calculate civ win rates for each player in each map
calculate_win_rate_map <- function(civ, rank_group, map_name) {
  total_games <- sum(((all_same_rank$p1.civ == civ & all_same_rank$p1.rank.group == rank_group & all_same_rank$map == map_name) | 
                        (all_same_rank$p2.civ == civ & all_same_rank$p2.rank.group == rank_group & all_same_rank$map == map_name)))
  wins <- sum(all_same_rank$civ.won == civ & 
                (((all_same_rank$p1.civ == civ & all_same_rank$p1.rank.group == rank_group & all_same_rank$map == map_name) | 
                    (all_same_rank$p2.civ == civ & all_same_rank$p2.rank.group == rank_group & all_same_rank$map == map_name))))
  if(total_games > 0) {
    win_rate <- (wins / total_games) * 100
  } else {
    win_rate <- 0
  }
  return(win_rate)
}

# Get unique maps, civilizations, and rank groups
unique_maps <- unique(all_same_rank$map)
unique_civs <- unique(c(all_same_rank$p1.civ, all_same_rank$p2.civ))
unique_rank_groups <- unique(c(all_same_rank$p1.rank.group, all_same_rank$p2.rank.group))

# Create a list to store win rate matrices for each map
win_rate_matrices <- list()

# Iterate over each unique map
for (map_name in unique_maps) {
  # Create a matrix to store win rates for the current map
  win_rate_matrix <- matrix(NA, nrow = length(unique_civs), ncol = length(unique_rank_groups))
  rownames(win_rate_matrix) <- unique_civs
  colnames(win_rate_matrix) <- unique_rank_groups
  
  # Fill win rate matrix for the current map
  for (civ in unique_civs) {
    for (rank_group in unique_rank_groups) {
      win_rate_matrix[civ, rank_group] <- calculate_win_rate_map(civ, rank_group, map_name)
    }
  }
  
  # Save the win rate matrix for the current map
  win_rate_matrices[[map_name]] <- win_rate_matrix
}

# Create an empty df to store the results
civ_winrates_maps <- data.frame(rank.group = character(), civ = character(), map = character(), win.rate = numeric(), stringsAsFactors = FALSE)

# Iterate over each map
for (map_name in unique_maps) {
  # Extract win rate matrix for the current map
  win_rate_matrix <- win_rate_matrices[[map_name]]
  
  # Iterate over each combination of civilization and rank group
  for (civ in unique_civs) {
    for (rank_group in unique_rank_groups) {
      
      win_rate <- win_rate_matrix[civ, rank_group]
      
      if (!is.na(win_rate)) {
        civ_winrates_maps <- rbind(civ_winrates_maps, data.frame(rank.group = rank_group, civ = civ, map = map_name, win.rate = win_rate))
      }
    }
  }
}

# Reset row names
rownames(civ_winrates_maps) <- NULL

write.csv(civ_winrates_maps, file = "C:\\Users\\gwu\\Desktop\\GA DataA\\R Project files\\AOE 4\\Season 5 Analysis\\S5 CIV WINRATES BY MAP_RANK.CSV")
```

#### Each civilization match-up
```R
## Civilization match up win rates
df = read.csv('C:\\Users\\gwu\\Desktop\\GA DataA\\R Project files\\AOE 4\\Season 5 Analysis\\S5 1v1 SAME RANK.CSV')

# Create a function to calculate win rates
calculate_win_rate <- function(df, civ1, civ2) {
  # Filter dataframe for the specific matchup
  matchup_df <- df[(df$p1.civ == civ1 & df$p2.civ == civ2) | (df$p1.civ == civ2 & df$p2.civ == civ1), ]
  
  # Calculate win rate
  win_rate <- sum(matchup_df$civ.won == civ1) / nrow(matchup_df)
  
  return(win_rate)
}

# Get unique civilizations
unique_civs <- unique(c(df$p1.civ, df$p2.civ))

# Create an empty dataframe to store win rates and game ID counts
win_rates_df <- data.frame(
  civilization1 = character(),
  civilization2 = character(),
  win_rate = numeric(),
  game_id_count = numeric(),
  stringsAsFactors = FALSE
)

# Calculate win rates and game ID counts for each unique matchup
for (i in 1:length(unique_civs)) {
  for (j in 1:length(unique_civs)) {
    civ1 <- unique_civs[i]
    civ2 <- unique_civs[j]
    matchup <- paste(civ1, civ2, sep = "-")
    
    # Check if the matchup is between the same civilizations
    if (civ1 == civ2) {
      # Calculate win rate for the same civilization matchup
      same_civ_data <- df[(df$p1.civ == civ1 & df$p2.civ == civ2), ]
      win_rate <- sum(same_civ_data$civ.won == civ1) / nrow(same_civ_data)
      game_id_count <- nrow(same_civ_data)
    } else {
      # Calculate win rate for different civilization matchup
      matchup_data <- df[(df$p1.civ == civ1 & df$p2.civ == civ2) | (df$p1.civ == civ2 & df$p2.civ == civ1), ]
      win_rate <- sum(matchup_data$civ.won == civ1) / nrow(matchup_data)
      game_id_count <- nrow(matchup_data)
    }
    
    # Append to the win_rates_df dataframe
    win_rates_df <- rbind(win_rates_df, data.frame(civilization1 = civ1, civilization2 = civ2, win_rate = win_rate, game_id_count = game_id_count))
  }
}

write.csv(win_rates_df,file = 'C:\\Users\\gwu\\Desktop\\GA DataA\\R Project files\\AOE 4\\Season 5 Analysis\\S5 CIVILIZATION MATCH UP WIN RATES.CSV')
```

