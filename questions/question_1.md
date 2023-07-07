# Question 1

A SQL query

```
SELECT count(*) as num_players, Rank
FROM Players
JOIN Levels ON (Players.Level_ID = Levels.Level_ID)
GROUP BY LevelName;
```

## What is wrong with this query, and why?

> Assumption: Using Postgresql database

## Answer

- Column and table names don't conform to "legal" lowercase. Postgres will automatically lowercase identifiers if not surrounded by double quotes. All references to columns and tables will need to utilise double quotes.

- `Rank` is not an aggregate or part of the `GROUP BY` clause. This is invalid SQL. The query should `GROUP BY` `Rank` to achieve the desired result.

- `LevelName` in the `GROUP BY` clause is an invalid column name.

- Assuming a `Level_Name` is unique to a `Level_ID` and the user is happy to describe levels by their ID the `JOIN` clause is unnecessary and will hinder the performance of the query. The `JOIN` can be removed. Instead `GROUP BY` `Players.Level_ID`.

- If the `Level_Name` is required, then `JOIN` on `Levels` to `SELECT` `Level_Name` and `GROUP_BY` `Levels.Level_Name`.

- The query doesn't return `Level_ID` or `Level_Name` so the user will be unable to determine which row corresponds to which level.

- `count(*)` is likely less performant than `count(Player_ID)` as `Player_ID` im assuming is the primary key and is indexed. Using `Player_ID` the DBMS need only query the index instead of the actual rows.

- `count(*)` will return 1 for levels where there are 0 players. `count(Player_ID)` will return 0.

- The default `JOIN` in Postgresql is performing an inner join. To return a complete dataset of all levels accompanied by `num_players` a `RIGHT OUTER JOIN` should be used.

- Optional: Add `ORDER BY Levels.Level_Name, Players.Rank desc` to order the results by `Level_Name` or `ORDER_BY num_players desc` to order the results by most populated levels and ranks.

## What would you do to fix it?

Incorporating all of the above suggestions and assuming the user wants to describe levels by `Level_Name` you get the following query:

```
SELECT 
    "Levels"."Level_Name",
    "Players"."Rank",
    COUNT("Players"."Player_ID") AS num_players
FROM "Players"
RIGHT OUTER JOIN "Levels" ON ("Players"."Level_ID" = "Levels"."Level_ID")
GROUP BY "Players"."Rank", "Levels"."Level_Name"
ORDER BY "Levels"."Level_Name", "Players"."Rank" desc;
```
