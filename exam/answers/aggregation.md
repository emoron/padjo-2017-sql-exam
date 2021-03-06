# Section B. Aggregation

These questions test your grasp of `GROUP BY` and aggregation functions (e.g. `MAX`, `MIN`, `AVG`, `SUM`, `COUNT`). There are a couple questions that require the `HAVING` clause; to make things a little easier, these questions specifically mention `HAVING`.

As before, pay attention to column arrangement, sort order, and the use of aliases to get the desired column names for the expected results. 

Add these answers to the Google Doc you created for the previous section, i.e. `padjo-2017-final-sql-exam-your_sunet_id`.




Previous related work:

- [SQL practice with SFPD incidents and group by aggregation ](https://gist.github.com/dannguyen/7e4fcfc6b91dd7749a23560912ee4e1e#file-03-grouping-md)
- [SQL practice with SFPD incidents and the HAVING clause](https://gist.github.com/dannguyen/7e4fcfc6b91dd7749a23560912ee4e1e#sql-practice-with-sfpd-incidents-and-the-having-clause-4-of-4)


<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [`legislators` table](#legislators-table)
  - [B01. Count the number of men and women currently in Congress](#b01-count-the-number-of-men-and-women-currently-in-congress)
  - [B02. List the days of the year in which more than one Republican House rep has a birthday](#b02-list-the-days-of-the-year-in-which-more-than-one-republican-house-rep-has-a-birthday)
  - [B03. Count the number of men and women, and get their average age, by political party and type (i.e. Senators vs. Representatives)](#b03-count-the-number-of-men-and-women-and-get-their-average-age-by-political-party-and-type-ie-senators-vs-representatives)
- [`pres_states` table](#pres_states-table)
  - [B04. The number of states won by each respective party, as well as the minimum, average, and maximum margins of victory, in 2016](#b04-the-number-of-states-won-by-each-respective-party-as-well-as-the-minimum-average-and-maximum-margins-of-victory-in-2016)
- [`pres_districts` table](#pres_districts-table)
  - [B05. The number of districts, by winning party and election year, in which the margin of victory was less than 5 percent](#b05-the-number-of-districts-by-winning-party-and-election-year-in-which-the-margin-of-victory-was-less-than-5-percent)
- [`disbursements` table](#disbursements-table)
  - [B06. List the frequency and total amount (in millions) of legislators' expenditures by category, for categories with at least $1M in total amount](#b06-list-the-frequency-and-total-amount-in-millions-of-legislators-expenditures-by-category-for-categories-with-at-least-1m-in-total-amount)
  - [B07. List the 5 legislators who had the highest total expenditures for second quarter of 2017](#b07-list-the-5-legislators-who-had-the-highest-total-expenditures-for-second-quarter-of-2017)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->



## `legislators` table


### B01. Count the number of men and women currently in Congress

Order the results by `gender`, i.e. the count for `'F'` should come before `'M'`.


In the expected results below, note the *alias* used as the column name for the aggregate count values. 




#### Answer
~~~sql
SELECT
  gender
  , COUNT(*) AS count
FROM 
  legislators
GROUP BY gender
ORDER BY gender ASC;
~~~



#### Expected results
| gender | count |
| ------ | ----- |
| F      | 106   |
| M      | 429   |


-------------


### B02. List the days of the year in which more than 3 Republicans share as a birthday

This exercise requires using the `HAVING` clause, which is used to filter an aggregate value, which you can't with just `WHERE`.

Order the days in descending order of `birthdays`, and then in chronological order.

Note that "day of the year" of someone's *birthday* is not the same as someone's actual **birth date**. The former is something like *May 9*, i.e. `05-09`, for someone who was born on *May 9, 2015*, i.e. `2015-05-09`.

Use a text-transforming function to get that bit of the `birthday` value needed to then group by -- [we've used it before plenty of times](https://stackoverflow.com/questions/10413055/how-to-get-substring-in-sqlite).




#### Answer
~~~sql
SELECT 
  SUBSTR(birthday, 6) AS day
  , COUNT(*) AS birthdays
FROM legislators
WHERE 
  party = 'Republican'
GROUP BY day
HAVING 
  birthdays > 3
ORDER BY 
  birthdays DESC
  , day ASC;
~~~




#### Expected results
| day   | birthdays |
| ----- | --------- |
| 03-01 | 6         |
| 08-24 | 5         |
| 02-16 | 4         |
| 03-03 | 4         |







### B03. Count the number of men and women, and get their average age, by political party and type (i.e. Senators vs. Representatives)

Order the results by `type`, then `party`, then `gender`.

Round the average age to the nearest tenth of a year (with the `ROUND` function).

While there is a "proper" way to do date calculations, that's not necessary here. The "age" for any given legislator can be calculated with arithmetic, using `2017` as the reference year (i.e. their age, roughly, in the year 2017) and subtracting the *year* component of their `birthday`.

For example, here's how to get a list of birthdays and their ages for 2017:

~~~sql
SELECT birthday,
  (2017 - SUBSTR(birthday, 1, 4)) AS age
FROM legislators;
~~~


#### Answer
~~~sql
SELECT
  type
  , party
  , gender
  , ROUND(AVG(2017 - SUBSTR(birthday, 1, 4)), 1) 
       AS average_age
  , COUNT(*) AS count
FROM 
  legislators
GROUP BY 
  party, type, gender
ORDER BY
  type ASC, party ASC, gender ASC;
~~~

#### Expected results
| type | party       | gender | average_age | count |
| ---- | ----------- | ------ | ----------- | ----- |
| rep  | Democrat    | F      | 63.1        | 63    |
| rep  | Democrat    | M      | 59.9        | 132   |
| rep  | Republican  | F      | 54.4        | 22    |
| rep  | Republican  | M      | 56.8        | 218   |
| sen  | Democrat    | F      | 61.8        | 16    |
| sen  | Democrat    | M      | 62.6        | 30    |
| sen  | Independent | M      | 74.5        | 2     |
| sen  | Republican  | F      | 60.4        | 5     |
| sen  | Republican  | M      | 62.9        | 47    |


## `pres_states` table

### B04. The number of states won by each respective party, as well as the minimum, average, and maximum margins of victory, in 2016

Round the average to the nearest tenth. List the results in descending order of states won.




#### Answer
~~~sql

SELECT
  winning_party
  , COUNT(*) AS states_won
  , MIN(winning_margin) AS smallest_margin
  , ROUND(AVG(winning_margin), 1) AS average_margin
  , MAX(winning_margin) AS biggest_margin
FROM 
  pres_states
WHERE
  year = 2016
GROUP BY 
  winning_party
ORDER BY
  states_won DESC;
~~~

#### Expected results
| winning_party | states_won | smallest_margin | average_margin | biggest_margin |
| ------------- | ---------- | --------------- | -------------- | -------------- |
| R             | 30         | 0.3             | 18.9           | 46.3           |
| D             | 21         | 0.3             | 17.7           | 86.8           |



## `pres_districts` table

### B05. The number of districts, by winning party and election year, in which the margin of victory was less than 5 percent




#### Answer
~~~sql
SELECT
  year
  , winning_party
  , COUNT(*) as district_count
FROM 
  pres_districts
WHERE
  winning_margin < 5
GROUP BY
  year, winning_party
ORDER BY
   winning_party ASC, year ASC;
~~~

#### Expected results
| year | winning_party | district_count |
| ---- | ------------- | -------------- |
| 2012 | D             | 19             |
| 2016 | D             | 20             |
| 2012 | R             | 26             |
| 2016 | R             | 20             |




## `disbursements` table






### B06. List the 5 legislators who had the highest total expenditures for second quarter of 2017

Remember that expenditures by legislators' offices (as opposed to committees) means that the `bioguide_id` column is not `NULL`.



#### Answer
~~~sql

SELECT
  bioguide_id
  , SUM(amount) AS total_amount
FROM 
  disbursements
WHERE 
  quarter = '2017Q2'
  AND bioguide_id IS NOT NULL
GROUP BY
  bioguide_id
ORDER BY
  total_amount DESC
LIMIT 5;
~~~


#### Expected results
| BIOGUIDE_ID | total_amount |
| ----------- | ------------ |
| G000571     | 400013.72    |
| M000087     | 390346.68    |
| H001045     | 358931.15    |
| E000215     | 357237.84    |
| B001295     | 353784.77    |




### B07. List the frequency and total amount (in millions) of all expenditures by category, for categories with at least $1M in total amount


List the results in descending order of amount spent. 

For `total_millions`, round to the nearest tenth of a million. To get the amount in millions, do some arithmetic (i.e. divide by `1000000`)

This exercise requires the `HAVING` clause.

Correction: previously, this question and headline asked for expenditures associated with legislators, i.e. requiring you to filter like this:


```
    WHERE bioguide IS NOT NULL
```

However, I had meant for the calculation and results to include all the data.


#### Answer
~~~sql
SELECT
  category
  , COUNT(*) AS total_things
  , ROUND(SUM(amount) / 1000000, 1)  AS total_millions
FROM 
  disbursements
WHERE
  bioguide_
GROUP BY
  category
HAVING
  total_millions > 1
ORDER BY
  total_millions DESC;
~~~

#### Expected results
| CATEGORY                       | total_things | total_millions |
| ------------------------------ | ------------ | -------------- |
| PERSONNEL COMPENSATION         | 33018        | 332.6          |
| PERSONNEL BENEFITS             | 14833        | 121.8          |
| OTHER SERVICES                 | 11166        | 37.0           |
| RENT  COMMUNICATION  UTILITIES | 42293        | 33.6           |
| EQUIPMENT                      | 6812         | 25.4           |
| SUPPLIES AND MATERIALS         | 37661        | 15.9           |
| TRAVEL                         | 42597        | 11.9           |
| PRINTING AND REPRODUCTION      | 7972         | 4.5            |
| FRANKED MAIL                   | 5316         | 3.4            |
| BENEFITS TO FORMER PERSONNEL   | 6            | 1.1            |

