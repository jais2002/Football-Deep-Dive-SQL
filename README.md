<div align="center">
  <h1> Football Deep Dive SQL</h1>
</div>

<div align="center">
<h1> <img src="https://github.com/user-attachments/assets/a43cec8b-3e6f-46f3-8a5e-8dcea43e6225" alt="description" width="500" height="300" style="border-bottom: 2px solid grey;"/></h1>
</div>

**Tools Used:** SQLite, HTML, Excel, Tableau

**Problem:** 

Football clubs, analysts, and fans are constantly seeking valuable insights from player and match data to make informed decisions, whether it's for player recruitment, performance analysis, or tactical planning. However, football datasets can be vast and complex, containing a large variety of metrics (such as goals, assists, player ratings, match events and physical attributes). Clubs are struggling to efficiently analyze this data to identify top performers, emerging talent, and patterns that influence match outcomes. 


**Solution:**

To help football clubs and analysts extract valuable insights from their extensive datasets, I will utilize SQL to efficiently process and query the data, focusing on key metrics such as top scorers, average goals per game, player height, and the relationship between player performance and player ratings. SQL will allow me to filter, aggregate, and analyze large volumes of data quickly, uncovering trends and performance indicators. 
<h1> </h1>

<h2> Preparation - What tables are we working with?</h2>

```sql
SELECT *
FROM sqlite_master
WHERE type='table';
```
![image](https://github.com/user-attachments/assets/79dd324f-d7e6-4e54-b033-2ab66053d364)
<h1> </h1>

<h2> 1 - First we want to look at what leagues/countries are included in the data set</h2>

```sql
SELECT *
FROM League
JOIN Country ON Country.id = League.country_id;
```

![image](https://github.com/user-attachments/assets/4995be93-d0bd-460e-96f1-ee4280f5c524)

<h2> 2 - In how much detail can we look into individual matches</h2>

```sql
SELECT Match.id, 
Country.name AS country_name, 
League.name AS league_name, 
season, 
stage, 
date,
HT.team_long_name AS  home_team,
AT.team_long_name AS away_team,
home_team_goal, 
away_team_goal                                        
FROM Match
JOIN Country on Country.id = Match.country_id
JOIN League on League.id = Match.league_id
LEFT JOIN Team AS HT on HT.team_api_id = Match.home_team_api_id
LEFT JOIN Team AS AT on AT.team_api_id = Match.away_team_api_id
WHERE country_name = 'England'
ORDER by date
LIMIT 5;
```
<h1> <img src="https://github.com/user-attachments/assets/776ceb5a-d570-4fcb-b057-0646bab168cc" alt="description" width="2000" height="110" style="border-bottom: 2px solid grey;"/></h1>

<h2> 3 - Now let's compare the 2015/16 football season across Europe's biggest leagues, comparing the number of goals scored</h2>

```sql
SELECT Country.name AS country_name, 
League.name AS league_name, 
season,
count(distinct HT.team_long_name) AS number_of_teams,
ROUND(avg(home_team_goal),2) AS avg_home_team_goals, 
ROUND(avg(away_team_goal),2) AS avg_away_team_goals, 
ROUND(avg(home_team_goal+away_team_goal),2) AS avg_goals, 
sum(home_team_goal+away_team_goal) AS total_goals        
FROM Match
JOIN Country on Country.id = Match.country_id
JOIN League on League.id = Match.league_id
LEFT JOIN Team AS HT on HT.team_api_id = Match.home_team_api_id
LEFT JOIN Team AS AT on AT.team_api_id = Match.away_team_api_id
WHERE country_name in ('Spain', 'Germany', 'France', 'Italy', 'England')
AND season = '2015/2016'
GROUP BY Country.name, League.name, season
ORDER BY Country.name, League.name, season DESC;
```
<h1> <img src="https://github.com/user-attachments/assets/5971b5ee-8230-4ebd-83af-0bfab54f00f9" alt="description" width="2000" height="115" style="border-bottom: 2px solid grey;"/></h1>

<h2> 4 - Is there a correlation between players height and their ratings?</h2>

```sql
SELECT CASE
WHEN ROUND(height)<165 then 165
WHEN ROUND(height)>195 then 195
ELSE ROUND(height)
END AS calc_height, 
COUNT(height) AS distribution, 
ROUND((avg(PA_Grouped.avg_overall_rating)),2) AS avg_overall_rating,
ROUND((avg(PA_Grouped.avg_potential)),2) AS avg_potential,
ROUND(AVG(weight),2) AS avg_weight 
FROM PLAYER
LEFT JOIN (SELECT Player_Attributes.player_api_id, 
avg(Player_Attributes.overall_rating) AS avg_overall_rating,
avg(Player_Attributes.potential) AS avg_potential  
FROM Player_Attributes
GROUP BY Player_Attributes.player_api_id) 
AS PA_Grouped ON PLAYER.player_api_id = PA_Grouped.player_api_id
GROUP BY calc_height
ORDER BY calc_height;
```

![image](https://github.com/user-attachments/assets/24d5912c-5975-46fd-983a-3b7dd358822a)

<div align="center">
<h1> <img src="https://github.com/user-attachments/assets/006f0d7c-ad79-447e-b379-a1f82e31bb3c" alt="description" width="500" height="300" style="border-bottom: 2px solid grey;"/></h1>
</div>

<h2> 5 - Finally we can use Tableau to visualize the data, expressing the Top 20 players amongst key attributes, in this case, Sprint Speed and Finishing</h2>

```sql
SELECT 
player_name,
height,
weight,
MAX(Player_Attributes.overall_rating) AS overall_rating,
Player_Attributes.sprint_speed,
Player_Attributes.finishing
FROM Player
left JOIN Player_Attributes on Player_Attributes.player_api_id = Player.player_api_id
GROUP BY player_name, height, weight
ORDER BY Player_Attributes.overall_rating DESC
LIMIT 20;
```

![Sheet 1](https://github.com/user-attachments/assets/65486301-73b5-45eb-af4c-47358f4f5836)

<h2> Conclusion</h2>

Through an in-depth analysis of football data, valuable insights into player performance and trends were uncovered. The examination of top scorers and average goals per game across different seasons provided a clear view of goal-scoring patterns, helping to identify key players and standout seasons. This information is crucial for clubs and analysts in assessing player impact and team strategy. Furthermore, player attributes such as height and weight were analyzed in relation to their overall ratings, revealing interesting trends about how physical characteristics might influence player performance and ratings over time.

The correlation between player height and performance was particularly notable, shedding light on whether taller players tend to achieve higher ratings or excel in specific positions. Additionally, the analysis of average player ratings over time can assist clubs in scouting emerging talents and making informed decisions on player recruitment or transfers. By leveraging this data, clubs can fine-tune their tactics, focusing on high-performing players while also understanding the impact of various attributes on success.

Finally, this project highlights the importance of data-driven analysis in football, demonstrating how insights derived from key performance metrics and player characteristics can provide a competitive edge in understanding game dynamics, improving team performance, and enhancing player development.



