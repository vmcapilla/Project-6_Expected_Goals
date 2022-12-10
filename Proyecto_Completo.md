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

## In BigQuery
