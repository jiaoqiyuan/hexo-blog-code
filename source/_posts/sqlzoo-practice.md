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


