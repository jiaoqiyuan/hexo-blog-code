---
title: sqlzoo-practice
date: 2018-10-22 16:17:14
tags:
categories: 数据库
thumbnail: https://zhidao91.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2015/11/databaseDevelopment.jpg
---

sqlzoo是一个[SQL语句练习网站][1]。


# SELECT from Nobel

- 1. 更改查詢以顯示1950年諾貝爾獎的獎項資料。
```
SELECT *
  FROM nobel
where yr=1950
```

- 2. 顯示誰贏得了1962年文學獎(Literature)。
```
SELECT winner
  FROM nobel
 WHERE yr = 1962
   AND subject = 'Literature'
```

- 3. 顯示“愛因斯坦”('Albert Einstein') 的獲獎年份和獎項。
```
SELECT yr, subject
  FROM nobel
 WHERE winner='Albert Einstein'
```

- 4. 顯示2000年及以後的和平獎(‘Peace’)得獎者。
```
SELECT winner
  FROM nobel
 WHERE yr>1999
  AND subject='Peace'
```

- 5. 顯示1980年至1989年(包含首尾)的文學獎(Literature)獲獎者所有細節（年，主題，獲獎者）。
```
select *
from nobel
where yr >=1980 and yr <= 1989
and subject='Literature'
```

- 6. 顯示總統獲勝者的所有細節：

	- 西奧多•羅斯福 Theodore Roosevelt
	- 伍德羅•威爾遜 Woodrow Wilson
	- 吉米•卡特 Jimmy Carter
```
SELECT * FROM nobel
 WHERE winner IN ('Theodore Roosevelt',
                  'Woodrow Wilson',
                  'Jimmy Carter')
```

- 7. 顯示名字為John 的得獎者。 (注意:外國人名字(First name)在前，姓氏(Last name)在後)
```
select winner
from nobel
where winner like 'John%'
```

- 8. 顯示1980年物理學(physics)獲獎者，及1984年化學獎(chemistry)獲得者。
```
select *
from nobel
where (yr=1980 and subject='physics') or (yr=1984 and subject='chemistry')
```

- 9. 查看1980年獲獎者，但不包括化學獎(Chemistry)和醫學獎(Medicine)。
```
select *
from nobel
where subject not in ('chemistry', 'medicine') and yr=1980
```

- 10. 顯示早期的醫學獎(Medicine)得獎者（1910之前，不包括1910），及近年文學獎(Literature)得獎者（2004年以後，包括2004年）。
```
select *
from nobel
where (yr<1910 and subject='Medicine')
or (yr >= 2004 and subject='Literature')
```

- 11. Find all details of the prize won by PETER GRÜNBERG
```
select *
from nobel
where winner='PETER GRÜNBERG'
```

- 12. 查找尤金•奧尼爾EUGENE O'NEILL得獎的所有細節 Find all details of the prize won by EUGENE O'NEILL
```
select *
from nobel
where winner='EUGENE O\'NEILL'
```

- 13. 騎士列隊 Knights in order
	- 列出爵士的獲獎者、年份、獎頁(爵士的名字以Sir開始)。先顯示最新獲獎者，然後同年再按名稱順序排列。
	```
	select winner, yr, subject
	from nobel
	where winner like 'Sir%'
	order by yr DESC, winner	
	```

- 14. The expression subject IN ('Chemistry','Physics') can be used as a value - it will be 0 or 1.

	- Show the 1984 winners and subject ordered by subject and winner name; but list Chemistry and Physics last.
	```
	SELECT winner, subject
	FROM nobel
 	WHERE yr=1984
 	ORDER BY subject in ('Physics', 'Chemistry'), subject, winner
	```


# SELECT IN SELECT

- 1. List each country name where the population is larger than that of 'Russia'.
	```
	SELECT name FROM world
	WHERE population >
		(SELECT population FROM world
		WHERE name='Russia')
	```

- 2. Show the countries in Europe with a per capita GDP greater than 'United Kingdom'.
	```
	select name
	from world
	where continent = 'Europe' 
	and gdp/population > (select gdp/population from world where name = 'United Kingdom')
	```

- 3. List the name and continent of countries in the continents containing either Argentina or Australia. Order by name of the country.
	```
	SELECT name, continent 
	FROM world
	WHERE continent IN 
	(SELECT continent FROM world WHERE name IN ('Argentina', 'Australia')) 
	ORDER BY name
	```

- 4. Which country has a population that is more than Canada but less than Poland? Show the name and the population.
	```
	SELECT name, population
	FROM world
	WHERE population > (SELECT population FROM world WHERE name = 'Canada') 
	AND population < (select population FROM world WHERE name='Poland')
	```

- 5. Germany (population 80 million) has the largest population of the countries in Europe. Austria (population 8.5 million) has 11% of the population of Germany.
	```
	SELECT name, CONCAT(ROUND(population*100/(SELECT population FROM world WHERE name = 'Germany'), 0), '%')
	FROM world
	WHERE continent = 'Europe'
	```

- 6. Which countries have a GDP greater than every country in Europe? [Give the name only.] (Some countries may have NULL gdp values)
	```
	SELECT name 
	FROM world
	WHERE gdp > (
	SELECT MAX(gdp) FROM world WHERE continent='Europe'
	AND gdp IS NOT NULL) 
	```

- 7. Find the largest country (by area) in each continent, show the continent, the name and the area:
	```
	SELECT continent, name, area FROM world x
	WHERE area>= ALL
    (SELECT area FROM world y
	WHERE y.continent=x.continent
		AND area>0)
	```

- 8. List each continent and the name of the country that comes first alphabetically.
	```
	SELECT continent, name
	FROM world x
	WHERE name <= ALL (
	SELECT name FROM world y WHERE x.continent=y.continent)
	```
	或者
	```
	SELECT continent, name
	FROM world x
	WHERE x.name = (
	SELECT name FROM world y WHERE x.continent=y.continent 
	ORDER BY name 
	LIMIT 1
	)
	```

- 9. Find the continents where all countries have a population <= 25000000. Then find the names of the countries associated with these continents. Show name, continent and population.
	```
	SELECT name, continent, population
	FROM world x
	WHERE 25000000 >= ALL (
	SELECT y.population FROM world y WHERE x.continent=y.continent
	AND y.population >=0
	)
	```

- 10. Some countries have populations more than three times that of any of their neighbours (in the same continent). Give the countries and continents.
	```
	SELECT name, continent
	FROM world x
	WHERE x.population/3 >= ALL (
	SELECT population FROM world y WHERE x.continent=y.continent AND y.name <> x.name
	)
	```

# SUM and COUNT
    This tutorial is about aggregate functions such as COUNT, SUM and AVG. An aggregate function takes many values and delivers just one value. For example the function SUM would aggregate the values 2, 4 and 5 to deliver the single value 11.

- 1. Show the total population of the world.
    ```
    SELECT SUM(population)
    FROM world
    ```

- 2. List all the continents - just once each.
    ```
    SELECT DISTINCT continent
    FROM world
    ```

- 3. Give the total GDP of Africa
    ```
    SELECT SUM(gdp)
    FROM world
    WHERE continent='Africa'
    ```

- 4. How many countries have an area of at least 1000000
    ```
    SELECT COUNT(name)
    FROM world
    WHERE area >= 1000000
    ```

- 5. What is the total population of ('Estonia', 'Latvia', 'Lithuania')
    ```
    SELECT SUM(population)
    FROM world
    WHERE name in ('Estonia', 'Latvia', 'Lithuania')
    ```

- 6. For each continent show the continent and number of countries.
    ```
    SELECT continent, COUNT(name)
    FROM world
    GROUP BY continent
    ```

- 7. For each continent show the continent and number of countries with populations of at least 10 million.
    ```
    SELECT continent, COUNT(name)
    FROM world
    WHERE population >= 10000000
    GROUP BY continent
    ```


- 8. List the continents that have a total population of at least 100 million.
    ```
    SELECT continent
    FROM world
    GROUP BY continent
    HAVING SUM(population) >= 100000000
    ```

## SUM and COUNT Quiz

选择题答案：

| Q\A | 1 | 2 | 3 | 4 | 5 |
|:--:|:--:|:--:|:--:|:--:|:--:|
| 1  | | | √ | | |
| 2  | √ | | | | |
| 3  | | | | √ | |
| 4  | | | | | √ |
| 5  | | √ | | | |
| 6  | | | | | √ |
| 7  | | | | √ | |
| 8  | | | | √ | |


# JOIN
- 1. The first example shows the goal scored by a player with the last name 'Bender'. The * says to list all the columns in the table - a shorter way of saying matchid, teamid, player, gtime

Modify it to show the matchid and player name for all goals scored by Germany. To identify German players, check for: teamid = 'GER'
    ```
    SELECT matchid, player FROM goal
    WHERE teamid='GER'
    ```

- 2. From the previous query you can see that Lars Bender's scored a goal in game 1012. Now we want to know what teams were playing in that match.

Notice in the that the column matchid in the goal table corresponds to the id column in the game table. We can look up information about game 1012 by finding that row in the game table.

Show id, stadium, team1, team2 for just game 1012
    ```
    SELECT id,stadium,team1,team2
    FROM game ga 
    WHERE id=1012
    ```


- 3.You can combine the two steps into a single query with a JOIN.
```
SELECT *
  FROM game JOIN goal ON (id=matchid)
```
The FROM clause says to merge data from the goal table with that from the game table. The ON says how to figure out which rows in game go with which rows in goal - the id from goal must match matchid from game. (If we wanted to be more clear/specific we could say 
ON (game.id=goal.matchid)

The code below shows the player (from the goal) and stadium name (from the game table) for every goal scored.

Modify it to show the player, teamid, stadium and mdate for every German goal.
    ```
    SELECT player, teamid, stadium, mdate
    FROM game JOIN goal ON (game.id=goal.matchid and goal.teamid='GER')
    ```

- 4. Use the same JOIN as in the previous question.

Show the team1, team2 and player for every goal scored by a player called Mario player LIKE 'Mario%'
    ```
    SELECT team1, team2, player
    FROM game JOIN goal
    ON (id=matchid AND player LIKE 'Mario%')
    ```

- 5. The table eteam gives details of every national team including the coach. You can JOIN goal to eteam using the phrase goal JOIN eteam on teamid=id

Show player, teamid, coach, gtime for all goals scored in the first 10 minutes gtime<=10
    ```
    SELECT player, teamid, coach, gtime
    FROM goal JOIN eteam
    ON (teamid=id AND gtime<=10)
    ```

- 6. To JOIN game with eteam you could use either
game JOIN eteam ON (team1=eteam.id) or game JOIN eteam ON (team2=eteam.id)

Notice that because id is a column name in both game and eteam you must specify eteam.id instead of just id

List the the dates of the matches and the name of the team in which 'Fernando Santos' was the team1 coach.
	```
	SELECT mdate, teamname
	FROM game JOIN eteam
	ON (team1=eteam.id AND coach='Fernando Santos')
	```

- 7. List the player for every goal scored in a game where the stadium was 'National Stadium, Warsaw'
	```
	SELECT player
	FROM goal JOIN game
	ON (matchid = id AND stadium='National Stadium, Warsaw')
	```
- 8. The example query shows all goals scored in the Germany-Greece quarterfinal.
Instead show the name of all players who scored a goal against Germany.
	```
	SELECT DISTINCT player
	FROM goal JOIN game
    ON matchid=id AND teamid <> 'GER' AND (team1='GER' OR team2='GER')
	```

- 9. Show teamname and the total number of goals scored.
	```
	SELECT teamname, COUNT(gtime)
	FROM eteam JOIN goal ON id=teamid
	GROUP BY teamname
	```

- 10. Show the stadium and the number of goals scored in each stadium.
	```
	SELECT stadium, COUNT(*) as goals
	FROM game JOIN goal
	ON id=matchid AND (team1=teamid OR team2=teamid)
	GROUP BY stadium
	```

- 11. For every match involving 'POL', show the matchid, date and the number of goals scored.
	```
	SELECT matchid, mdate, COUNT(*)
	FROM game JOIN goal
	ON id=matchid AND (team1 = 'POL' OR team2='POL')
	GROUP BY matchid, mdate
	```

- 12. For every match where 'GER' scored, show matchid, match date and the number of goals scored by 'GER'
	```
	SELECT matchid, mdate, COUNT(*)
	FROM game JOIN goal
	ON id=matchid AND(teamid='GER')
	GROUP BY matchid, mdate
	```

- 13. List every match with the goals scored by each team as shown. This will use "CASE WHEN" which has not been explained in any previous exercises.

|mdate|team1|score1|team2|score2|
|:-:|:-:|:-:|:-:|:-:|
|1|July|2012|	ESP	4|	ITA	0|
|10| June| 2012|	ESP	1|	ITA	1|
|10| June| 2012|	IRL	1|	CRO	3|
Notice in the query given every goal is listed. If it was a team1 goal then a 1 appears in score1, otherwise there is a 0. You could SUM this column to get a count of the goals scored by team1. Sort your result by mdate, matchid, team1 and team2.

```
SELECT mdate, team1,
SUM(CASE WHEN teamid=team1 THEN 1 ELSE 0 END) AS score1,
team2, 
SUM(CASE WHEN teamid=team2 THEN 1 ELSE 0 END ) AS score2
FROM game LEFT JOIN goal ON matchid = id
GROUP BY mdate, team1, team2
ORDER BY mdate, matchid, team1, team2
```


[1]: https://zh.sqlzoo.net