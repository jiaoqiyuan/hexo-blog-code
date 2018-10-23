---
title: sqlzoo-practice
date: 2018-10-22 16:17:14
tags:
categories: 数据库
thumbnail: https://zhidao91.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2015/11/databaseDevelopment.jpg
---


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