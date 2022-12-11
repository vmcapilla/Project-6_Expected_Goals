# Complete Project 6: Expected vs Scored Goals in the World Cup 2018

# Objective
Practice my Big Query skills to extract the main trends in terms of goals expected and goals scored.

# Data source
The data has been extracted from the repository (https://fbref.com/es/comps/1/2018/horario/Marcadores-y-partidos-de-2018-World-Cup) as csv

# Programs and packages
A first cleaning of the data has been performed in excel and then all the analysis has been performed in BigQuery.

# Processes

## In excel
1. Transform the data to columns
2. Eliminate the empty columns
3. Transform England to gb
4. Create another Local column and replace it with the function right() to get the emoji name of the countries
6. Replace the Visitor column with left() to get the same result and make both columns uniform
7. Change the first xG for xGL (xG Local) and the second for xGV (xG Visitor)
8. Delete columns F, K, N, O
9. Change the Result column into Local Goals and Away Goals with the right() and left() columns
10. Create the LpG and VpG columns as the goals scored in penalties for both teams
11. Upload the file to BigQuery

# Analysis

To carry out the analysis, I need the statistics of each team, both home and away. To do so, I will perform the following query before each of them in order to be able to join both results.

## Total expected goals for and against for each team
```
WITH away AS (
SELECT 
  Visitante AS Team,
  sum(xGV) AS xG_Created,
  sum(xGL) AS xG_Conceded,
  count(xGV) AS Games_A

 FROM 
 `wc2018` 

GROUP BY
  Team
), home AS (
SELECT 
  Local AS Team,
  sum(xGL) AS xG_Created,
  sum(xGV) AS xG_Conceded,
  count(xGV) AS Games_H

 FROM 
 `wc2018` 

GROUP BY
  Team
)
SELECT
  A.Team AS Team,
  round(sum(A.xG_Created) + sum(H.xG_Created), 2) AS xG_Tot_Created
  sum(Games_H) + sum(Games_A) AS Games2

FROM
  home H
JOIN away A ON A.Team = H.Team

GROUP BY
  Team

ORDER BY
  xG_Tot_Created DESC
  
LIMIT
  5
```
#Output Top 10 xG Created
```
Team	xG_Tot_Created
be	13.4
br	11.7
gb	11.2
hr	11.1
fr	9.2
```
#Output Top 10 xG Conceded
```
Team	xG_Tot_Created	xG_Tot_Conceded
be	8.7
tn	8.6
gb	7.7
hr	7.7
ru	7.5
```

We can see that Belgium (be) is very good at creating chances but also concedes quite a few against them. In order to make a more realistic analysis, let's divide the data by the number of matches:
## Expected goals per game for and against for each team
```
WITH away AS (
SELECT 
  Visitante AS Team,
  sum(xGV) AS xG_Created,
  sum(xGL) AS xG_Conceded,
  count(xGV) AS Games_A

 FROM 
 `wc2018` 

GROUP BY
  Team
), home AS (
SELECT 
  Local AS Team,
  sum(xGL) AS xG_Created,
  sum(xGV) AS xG_Conceded,
  count(xGV) AS Games_H

 FROM 
 `wc2018` 

GROUP BY
  Team
)
SELECT
  A.Team AS Team,
  round(((sum(A.xG_Created) + sum(H.xG_Created))/(sum(Games_H) + sum(Games_A))), 2) AS xG_Gm_Created,
  round(((sum(A.xG_Conceded) + sum(H.xG_Conceded))/(sum(Games_H) + sum(Games_A))), 2) AS xG_Tot_Conceded,
  sum(Games_H) + sum(Games_A) AS Games

FROM
  home H
JOIN away A ON A.Team = H.Team

GROUP BY
  Team

ORDER BY
  xG_Gm_Created DESC
LIMIT
  5
```
#Output
```
Team	xG_Gm_Created	xG_Tot_Conceded	Games
br	2.34	0.6	5
es	2.05	1.02	4
be	1.91	1.24	7
de	1.83	1.3	3
gb	1.6	1.1	7
```
#Output Top 5 xG Conceded (Teams that have received the most opportunities)
```
Team	xG_Gm_Created	xG_Gm_Conceded	Games
tn	1.4	2.87	3
kr	0.97	2.3	3
pa	0.73	2.3	3
mx	1.32	1.85	4
eg	0.83	1.8	3
```
#Output Tail 5 xG Concedeed (Teams that have received the least opportunities)
```
Team	xG_Gm_Created	xG_Gm_Conceded	Games
uy	1.44	0.44	5
br	2.34	0.6	5
sn	0.97	0.67	3
fr	1.31	0.69	7
au	1.07	0.77	3
```
We can see that in these statistics Brazil (br) is one of the best, although it did not win the title.
## Table with games played (PJ), games won (W), games drawn (D), games lost (L), goals scored (GF), expected goals scored (xGF), goals conceded (GC) and expected goals conceded (xGC).
```
WITH away AS (
SELECT 
  Visitante AS Team,
  COUNT(Visitante) AS PJ,
  SUM(AG) AS GF,
  SUM(LG) AS GC,
  COUNTIF(AG > LG) AS W,
  COUNTIF(AG = LG) AS D,
  COUNTIF(AG < LG) AS L,
  SUM(xGV) AS xGF,
  SUM(xGL) AS xGC
 FROM 
 `wc2018` 

GROUP BY
  Team
), home AS (
SELECT 
  Local AS Team,
  COUNT(Local) AS PJ,
  SUM(LG) AS GF,
  SUM(AG) AS GC,
  COUNTIF(LG > AG) AS W,
  COUNTIF(LG = AG) AS D,
  COUNTIF(LG < AG) AS L,
  SUM(xGL) AS xGF,
  SUM(xGV) AS xGC

 FROM 
 `wc2018` 

GROUP BY
  Team
)
SELECT
  A.Team AS Team,
  CAST( ROUND( SUM( A.PJ)+SUM( H.PJ)) AS INT) AS PJ,
  SUM(A.W) + SUM(H.W) AS W,
  SUM(A.D) + SUM(H.D) AS D,
  SUM(A.L) + SUM(H.L) AS L,
  SUM(A.GF)+SUM(H.GF) AS GF,
  ROUND(SUM(A.xGF)+SUM(H.xGF), 2) AS xGF,
  SUM(A.GC)+SUM(H.GC) AS GC,
  ROUND(SUM(A.xGC)+SUM(H.xGC), 2) AS xGC
FROM
  home H
JOIN away A ON A.Team = H.Team

GROUP BY
  Team
```
#Output
```
Team	PJ	W	D	L	GF	xGF	GC	xGC
fr	7	6	1	0	14	9.2	6	4.8
uy	5	4	0	1	7	7.2	3	2.2
br	5	3	1	1	8	11.7	3	3
be	7	6	0	1	16	13.4	6	8.7
se	5	3	0	2	6	7.4	4	4.8
hr	7	4	2	1	14	11.1	9	7.7
ru	5	2	2	1	11	5	7	7.5
eg	3	0	0	3	2	2.5	6	5.4
ma	3	0	1	2	2	2.8	4	5.1
pt	4	1	2	1	6	4.7	6	4.5
ar	4	1	1	2	6	4.9	9	6.5
pe	3	1	0	2	2	2.3	2	3.9
cr	3	0	1	2	2	2.5	5	4.9
de	3	1	0	2	2	5.5	4	3.9
tn	3	1	0	2	5	4.2	8	8.6
co	4	2	1	1	6	4	3	5.7
pl	3	1	0	2	2	3.3	5	3.8
ir	3	1	1	1	2	3.1	2	3.9
dk	4	1	3	0	3	2.9	2	5.9
ng	3	1	0	2	3	3	4	3.7
rs	3	1	0	2	2	3.2	4	4.4
kr	3	1	0	2	3	2.9	3	6.9
gb	7	3	1	3	12	11.2	8	7.7
jp	4	1	1	2	6	4.3	7	6.3
sa	3	1	0	2	2	3.1	7	4.2
es	4	1	3	0	7	8.2	6	4.1
au	3	0	1	2	2	3.2	5	2.3
is	3	0	1	2	2	4.4	5	3.5
mx	4	2	0	2	3	5.3	6	7.4
ch	4	1	2	1	5	5.1	5	6.5
sn	3	1	1	1	4	2.9	4	2
pa	3	0	0	3	2	2.2	11	6.9
```

I save this table as 'wc2018_all_games_stats' for future reference.  

## Top 5 and Tail 5 ratio of goals scored vs. expected goals
Teams that have been luckier with respect to expected goals or are superior to the median team.
```
SELECT 

Team,
GF,
xGF,
ROUND(GF/xGF, 2) AS xGF_Ratio,

FROM 

`wc2018_all_games_stats` 

ORDER BY
  xGF_Ratio DESC
  
LIMIT 5
```
#Output Top 5 xGF_Ratio (ORDER BY xGF_Ratio DESC)
```
Team	GF	xGF	xGF_Ratio
ru	11	5	2.2
fr	14	9.2	1.52
co	6	4	1.5
jp	6	4.3	1.4
sn	4	2.9	1.38
```
#Output Tail 5 xGF_Ratio (ORDER BY xGF_Ratio)
```
Team	GF	xGF	xGF_Ratio
de	2	5.5	0.36
is	2	4.4	0.45
mx	3	5.3	0.57
pl	2	3.3	0.61
au	2	3.2	0.63
```
I am very surprised to see germany (de) in the tail stats, they generated many chances but could not finish them properly, as for the top in favor I am very surprised by russia (ru) which explains their good ranking (4).

## Top 5 and Tail 5 ratio of goals conceded vs. expected goals conceded
```
SELECT 

Team,
GC,
xGC,
ROUND(GC/xGC, 2) AS xGC_Ratio


FROM 

`wc2018_all_games_stats` 

ORDER BY
  xGC_Ratio
LIMIT 5
```
#Output Top 5 xGC_Ratio (ORDER BY xGC_Ratio)
```
Team	GC	xGC	xGC_Ratio
dk	2	5.9	0.34
kr	3	6.9	0.43
pe	2	3.9	0.51
ir	2	3.9	0.51
co	3	5.7	0.53
```
#Output Tail 5 xGC_Ratio (ORDER BY xGC_Ratio DESC)
```
Team	GC	xGC	xGC_Ratio
au	5	2.3	2.17
sn	4	2	2
sa	7	4.2	1.67
pa	11	6.9	1.59
es	6	4.1	1.46
```
In this case I am surprised that Spain (es) is doing so badly and Denmark (dk) so well. Croatia (kr) was a very solid block during the whole tournament.
