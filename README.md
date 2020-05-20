# sportradar_scraper
Code in order to scrape the SportRadar Play By Play JSON using your proper credentials!
# Merging with nflstatR

**In order to merge this SportRadar data with R there a few steps you have to follow.**

Our first link is that the reference column inside of our cleaned data is the play_id that nflstatR uses in their pbp data. Now, we can't simply merge just using this reference number as the same play_id is used across multiple games. In our Scraped DataFrame we created a for_r column that we can as an additional join factor. Inside of R, mutate a new column into your pbp dataset called whatever you would like that combines the home_team,away_team and week. </body><br>
**For example: all_nfl_pbp <-all_nfl_pbp %>% mutate(for_r = paste0(home_team,away_team,week))**


Now, we will load in our csv of choice (whole season/single game) and create an inner join. Heres where the merge becomes interesting. When we parse the SportRadar JSON we pull multiple stat lines for each play. This means our final product will have 2-3 (sometimes even 4 rows) dedicated to it. However, the beauty of our merge is that our pbp data will turn its one row of original data into the same number of rows dedicated to a play. (e.g Kickoff has two stat types, so we now have two rows of nflstatr pbp data). The merge will look like this:
**sports_radar_pbp <- inner_join(all_nfl_pbp, sport_radar, by = c('game_link' = 'for_r', 'week', 'play_id' = 'reference'))**


Of course, you are probably now wondering about the fact that you have loads of duplicate rows of data throwing off the stats that you would like to see. The solution to this is in the emphasis**stat_type** column inside our newly merged data. If I would like to see only passes for example, I can filter my dataset to only show the stat_type == "passes". This will return only pass plays, but will only be one row per play. We can do this for defensive plays, kick returns, receptions, etc.

Heres an example of pre filtered data:
| play_id        | stat_type           | desc  |
| ------------- |:-------------:| -----:|
| 50   | rush | (15:00) 33-A.Jones left tackle to GB 25 for no gain  |
| 50      | defense      |   (15:00) 33-A.Jones left tackle to GB 25 for no gain  |
| 71 | pass | (14:33) 12-A.Rodgers pass short left to 33-A.Jones to GB 25 for no gain (58-R.Smith). |
| 71 | defense | (14:33) 12-A.Rodgers pass short left to 33-A.Jones to GB 25 for no gain (58-R.Smith). |

Obviously, once we filter to our stat_type we will be able to treat it as we would with the traditional nflstatR data.

Lets do a quick double check to make sure everything makes sense. For this we will check our QB stats against their real life stats. Heres our code for the check:

**pass_checks <- sports_radar_pbp %>% filter(pass == 1, stat_type == "pass", sack.x == 0, qb_scramble == 0) %>% group_by(passer) %>% summarize(plays = n(), yards_gained = sum(yards_gained,na.rm = TRUE)) %>% filter(plays>50) %>% arrange(desc(yards_gained))**

