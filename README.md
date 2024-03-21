# 🛖AGE OF EMPIRES 4 GAME STATISTICAL ANALYSIS

Age Of Empires 4 is a very complex real-time strategy game set in the middle-ages where players require immense knowledge about many mechanics of the game to perform competitively. At the time of the most recent season, AOE4 has 16 unique civilizations and ~30 maps; each civilization with unique bonuses, units, upgrades, and approaches to gameplay, each map with unique resources, land/sea formations, and a range of geographical difference. 

## Project Overview

#### ❓The problem
Parts of the AOE4 community have expressed their concerns about a variety of things: some civilizations are too strong, some maps are overtly advantageous to some civilizations, and some unique upgrades of exclusive units are overpowered. 
Arguments the balance of civilziations are ongoing, and with every patch, the developers buff or nerf different parts of the game to attmept at making the game more balanced and fair for players. **But with so much emotional biases in ranked play and many game factors to consider, how can we determine if the game really is balanced or not?**


#### 📝The approach For a Solution
With the start of each season, all players' ranked ladder title and ratings from the previous season are cleared to zero. Meaning analyzing season 5 (6-16-23 to 11-14-23) and season 6 (11-14-23 to 3-20-24) match and player data can provide us with very valuable insights into the balancing of the game and give an idea of what the current season 7 might look like. With visualization inspiration from [AOE4 WORLD](https://aoe4world.com/), I will do a deep dive into these categories for both seasons:
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
- R for data scraping, preparing, cleaning, analyzing, and discovering trends
- Tableau for dashboard creation and visualizations

### Data Preperation & Cleaning
The JSON files from AOE4 World contained a lot of nested lists. Turning the files into dataframes were easy however some columns had 3 layers of nested dictionaries which required extraction and seperation.    

The following is the process of turning layers of nested dictionaries (season 5 & 6 JSON files) into a readable dataframe and renaming/reordering for analysis.
```R
#install.packages("rjson")
library("rjson")
library("data.table")
library("tidyverse")

#read the json file 
jdata_s5 = fromJSON(file ='C://Users//gwu//Desktop//GA DataA//R-Project Files//AOE 4//Raw RM_1v1_JSON files//games_rm_1v1_s5.json')

#turn large list from json file into data frame
raw_df_s5 = as.data.frame(do.call(rbind,jdata_s5))

#teams column has the data of both players in a nested list of lists of dictionaries
#get the list of teams separate from the rest of the df
teams_s5 = raw_df_s5$teams

#separate player 1 and player 2 from teams nested lists
p1_s5 = lapply(teams_s5,function(l) l[[1]][[1]])
p2_s5 = lapply(teams_s5,function(l) l[[2]][[1]])

#player 1 and player 2 are now lists of dictionaries,separate them into dfs
p1_s5_df = rbindlist(p1_s5)
p2_s5_df = rbindlist(p2_s5)

#rename columns to identify p1 and p2
p1_s5_df <- p1_s5_df %>%
  dplyr::rename("p1 ID" = profile_id, "p1 result" = result, 
         "p1 civ" = civilization,"p1 civ randomized" = civilization_randomized,
         "p1 mmr" = mmr, "p1 mmr dif" = mmr_diff)

p2_s5_df <- p2_s5_df %>%
  dplyr::rename("p2 ID" = profile_id, "p2 result" = result, 
         "p2 civ" = civilization,"p2 civ randomized" = civilization_randomized,
         "p2 mmr" = mmr, "p2 mmr dif" = mmr_diff)

#drop rating and rating diff columns as they are all NA
p1_s5_df = p1_s5_df %>%
  subset(select = -c(rating,rating_diff))

p2_s5_df = p2_s5_df %>%
  subset(select = -c(rating,rating_diff))

#drop unneeded columns from raw_df_s5 to combine p1 and p2 with overall data
dropped_df_s5 = raw_df_s5 %>%
  subset(select = -c(teams,kind,patch))

#add player1, player2, and game dfs together
player_df_s5 = cbind(p1_s5_df,p2_s5_df)
RM_1v1_s5 = cbind(dropped_df_s5, player_df_s5)

RM_1v1_s5 = unnest(RM_1v1_s5)

#export out to csv
write_csv(RM_1v1_s5,"C:\\Users\\gwu\\Desktop\\GA DataA\\R-Project files\\AOE 4\\Cleaned RM 1v1 CSV Files\\AOE4_RM_1v1_s5.csv")
```
Unused columns were dopped, like "patch" and "kind", every game in both datasets were ranked 1v1

The data sets were validated for games within the range of the seasons, and checked for inconsistent/missing values.

**add in season 6 JSON ranked 1v1 data when AOE4 WORLD releases it**


Columns are renamed and ordered for ease of use

### Data Analysis

