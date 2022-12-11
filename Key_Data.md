# Objective
Practice my Big Query skills to extract the main trends in terms of goals expected and goals scored.
# Data source
The data has been extracted from the repository (https://fbref.com/es/comps/1/2018/horario/Marcadores-y-partidos-de-2018-World-Cup) as csv
# Programs and packages
A first cleaning of the data has been performed in excel and then all the analysis has been performed in BigQuery.
# Processes
1. Cleaning the data in excel
2. Creating the table in BigQuery
3. Creating a table from a query
# Main queries and results
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
Output Top 5xG Created
```
Team	xG_Tot_Created
be	13.4
br	11.7
gb	11.2
hr	11.1
fr	9.2
```
Output Top 5xG Conceded
```
Team	xG_Tot_Conceded
be	8.7
tn	8.6
gb	7.7
hr	7.7
ru	7.5
```
## Table with games played (PJ), games won (W), games drawn (D), games lost (L), goals scored (GF), expected goals scored (xGF), goals conceded (GC) and expected goals conceded (xGC)
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
 Output:
  ```
  Team	PJ	W	D	L	GF	xGF	GC	xGC
fr	7	6	1	0	14	9.2	6	4.8
uy	5	4	0	1	7	7.2	3	2.2
br	5	3	1	1	8	11.7	3	3
be	7	6	0	1	16	13.4	6	8.7
se	5	3	0	2	6	7.4	4	4.8
hr	7	4	2	1	14	11.1	9	7.7
...
```
## Top 5 and Tail 5 ratio of goals scored vs. expected goals and goals conceded vs. expected goals conceded
```
SELECT 

Team,
GF,	#GC for Goal Conceded
xGF,	#xGC for Goal Conceded
ROUND(GF/xGF, 2) AS xGF_Ratio,	#GC/xGC AS xGC_Ratio for Goal Conceded

FROM 

`wc2018_all_games_stats` 

ORDER BY
  xGF_Ratio DESC	#xGC_Ratio for Goal Conceded
  
LIMIT 5
```
Output Top 5 xGF_Ratio (ORDER BY xGF_Ratio DESC)
```
Team	GF	xGF	xGF_Ratio
ru	11	5	2.2
fr	14	9.2	1.52
co	6	4	1.5
jp	6	4.3	1.4
sn	4	2.9	1.38
```
Output Tail 5 xGF_Ratio (ORDER BY xGF_Ratio)
```
Team	GF	xGF	xGF_Ratio
de	2	5.5	0.36
is	2	4.4	0.45
mx	3	5.3	0.57
pl	2	3.3	0.61
au	2	3.2	0.63
```
Output Top 5 xGC_Ratio (ORDER BY xGC_Ratio)
```
Team	GC	xGC	xGC_Ratio
dk	2	5.9	0.34
kr	3	6.9	0.43
pe	2	3.9	0.51
ir	2	3.9	0.51
co	3	5.7	0.53
```
Output Tail 5 xGC_Ratio (ORDER BY xGC_Ratio DESC)
```
Team	GC	xGC	xGC_Ratio
au	5	2.3	2.17
sn	4	2	2
sa	7	4.2	1.67
pa	11	6.9	1.59
es	6	4.1	1.46
```

 
 
