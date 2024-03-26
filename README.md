# 🛖AGE OF EMPIRES 4 GAME STATISTICAL ANALYSIS

Age Of Empires 4 is a complex real-time strategy game set in the middle-ages where players require immense knowledge about many mechanics of the game to perform competitively. At the time of the most recent season (S7), AOE4 has 16 unique civilizations and ~30 maps; each civilization with unique bonuses, units, upgrades, and approaches to gameplay, each map with unique resources, land/sea formations, and a range of geographical difference. 

## Project Overview

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
- R for data scraping, preparing, cleaning, analyzing, and discovering trends
- Tableau for dashboard creation and visualizations

### Data Preperation & Cleaning
The JSON files from AOE4 World contained a lot of nested lists. Turning the files into dataframes were easy however some columns had 3 layers of nested dictionaries which required extraction and seperation.    

#### The following is the process of turning layers of nested dictionaries (season 5 & 6 JSON files) into a readable dataframe and renaming/reordering for analysis.
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

#### Merging player leader boards data with match data to get one big dataframe, validating and then renaming columns for organization

```R
#read data
match = read.csv("C:\\Users\\gwu\\Desktop\\GA DataA\\R Project files\\AOE 4\\Cleaned RM 1v1 CSVs\\AOE4_RM_1v1_s5.csv")
lb = read.csv("C:\\Users\\gwu\\Desktop\\GA DataA\\R Project files\\AOE 4\\Raw Leaderboard Excel Files\\leadersboards_rm_solo_points_s5.csv.gzip.csv")

#drop mmr columns because missing values
match = subset(match, select = -c(p1.mmr, p1.mmr.dif, p2.mmr, p2.mmr.dif))

#Season 5 solo queue player win rates by rank title/points
lb$winrate = (lb$wins_count/lb$games_count)*100 #calculate win rate
lb$group = sub("\\_.*","",lb$rank_level) #group general rank levels

#display the winning civilization of each match
match$civ.won <- ifelse(match$p1.result == "win", match$p1.civ, match$p2.civ)


##find out each civilization pick and win rate by rank groups
#separate p1 and p2 match data because need to merge by profile ID and each row has both players' ID
p1_match = subset(match, select = c(game_id,duration,map,p1.ID,p1.result,p1.civ,civ.won))
p2_match = subset(match, select = c(game_id,duration,map,p2.ID,p2.result,p2.civ,civ.won))

#merge matches with each player information from leader boards (lb)
p1_match_lb = merge(p1_match,lb, by.x = 'p1.ID', by.y = 'profile_id')
p2_match_lb = merge(p2_match,lb, by.x = 'p2.ID', by.y = 'profile_id')

#merge both dfs by game id, now have player rank info and match info in one df.
all = merge(p1_match_lb,p2_match_lb, by = 'game_id')

#clean and organize naming/ordering
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

Columns are renamed and ordered for ease of use

### Data Analysis
Most statistical analysis was done in R, then exported out to Tableau for visualizations and dashboard creation.   

See [AOE4 S5 & S6 Dashboard]()

#### Calculating each civilization's win rate and pick rate for each rank group. Calculating each civilization's win rate for each map and for each rank group
```R
all = read.csv("C:\\Users\\gwu\\Desktop\\GA DataA\\R Project files\\AOE 4\\Season 5 Analysis\\AOE4 S5 1V1 RANKED DATA.csv")

##Win and pick rate of each civilization for each rank group

#win rate
civ_winrate = all |>
  pivot_longer(matches("^p\\d"),
               names_pattern = "(p\\d).(.*)", names_to = c("player", ".value")) |>
  summarize(winrate = mean(civ == civ.won), .by = c(rank.group, civ)) |>
  arrange(desc(rank.group))

write.csv(civ_winrate, file = "C:\\Users\\gwu\\Desktop\\GA DataA\\R Project files\\AOE 4\\Season 5 Analysis\\S5 CIV WINRATE BY RANK.CSV")

#pick rate

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

##calculate civilization win rates on each map for every rank group

civ_map_winrate <- all %>%
  pivot_longer(matches("^p\\d"),
               names_pattern = "(p\\d).(.*)", names_to = c("player", ".value")) %>%
  group_by(rank.group, civ, map) %>%
  summarize(winrate = mean(civ == civ.won), .groups = "drop") %>%
  arrange(desc(rank.group))

write.csv(civ_map_winrate, file = "C:\\Users\\gwu\\Desktop\\GA DataA\\R Project files\\AOE 4\\Season 5 Analysis\\S5 CIV WINRATE BY MAP_RANK.CSV")
```
