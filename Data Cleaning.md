# LEFT, RIGHT, LENGTH

- In the `accounts` table, there is a column holding the **website** for each company. The last three digits specify what type of web address they are using. A list of extensions (and pricing) is provided [here](https://iwantmyname.com/domains). Pull these extensions and provide how many of each website type exist in the `accounts` table.

```javascript
SELECT RIGHT(website, 3) AS domain, 
       COUNT(*) num_companies
FROM accounts
GROUP BY 1
ORDER BY 2 DESC;
```

- There is much debate about how much the name (or even the first letter of a company name) matters. Use the `accounts` table to pull the first letter of each company name to see the distribution of company names that begin with each letter (or number).

```javascript
SELECT LEFT(UPPER(name), 1) AS first_letter, 
       COUNT(*) num_companies
FROM accounts
GROUP BY 1
ORDER BY 2 DESC;
```

- Use the `accounts` table and a **CASE** statement to create two groups: one group of company names that start with a number and a second group of those company names that start with a letter. What proportion of company names start with a letter?

```javascript
SELECT SUM(num) nums, 
       SUM(letter) letters
FROM (SELECT name, CASE WHEN LEFT(UPPER(name), 1) IN ('0','1','2','3','4','5','6','7','8','9') 
                       THEN 1 ELSE 0 END AS num, 
         CASE WHEN LEFT(UPPER(name), 1) IN ('0','1','2','3','4','5','6','7','8','9') 
                       THEN 0 ELSE 1 END AS letter
      FROM accounts) t1;
```
> There are 350 company names that start with a letter and 1 that starts with a number. This gives a ratio of 350/351 that are company names that start with a letter or 99.7%.
- Consider vowels as **a, e, i, o, and u**. What proportion of company names start with a vowel, and what percent start with anything else?

```javascript
SELECT SUM(vowels) vowels, SUM(other) other
FROM (SELECT name, CASE WHEN LEFT(UPPER(name), 1) IN ('A','E','I','O','U') 
                        THEN 1 ELSE 0 END AS vowels, 
          CASE WHEN LEFT(UPPER(name), 1) IN ('A','E','I','O','U') 
                       THEN 0 ELSE 1 END AS other
         FROM accounts) t1;
```
> There are 80 company names that start with a vowel and 271 that start with other characters. Therefore, 80/351 or 22.8% are vowels, 77.2% of company names do not start with vowels.
---
# POSITION, STRPOS, & SUBSTR

**POSITION(',' IN city_state)** takes a character and a column, and provides the index where that character is for each row       \
**STRPOS(city_state, ',')** provides the same result as POSITION

> The index of the first position is 1 in SQL
- Use the `accounts` table to create **first** and **last** name columns that hold the first and last names for the **primary_poc**.
```javascript
SELECT LEFT(primary_poc, STRPOS(primary_poc, ' ') -1 ) first_name, 
       RIGHT(primary_poc, LENGTH(primary_poc) - STRPOS(primary_poc, ' ')) last_name
FROM accounts;
```

- Now see if you can do the same thing for every rep **name** in the `sales_reps` table. Again provide **first** and **last** name columns.
```javascript
SELECT LEFT(name, STRPOS(name, ' ') -1 ) first_name, 
       RIGHT(name, LENGTH(name) - STRPOS(name, ' ')) last_name
FROM sales_reps;
```
---
# CONCAT, Piping ||

CONCAT(first_name, ' ', last_name)\
first_name || ' ' || last_name

- Each company in the `accounts` table wants to create an email address for each **primary_poc**. The email address should be the first name of the primary_poc `.` last name primary_poc `@` company name `.com`.
```javascript
WITH t1 AS (
 SELECT LEFT(primary_poc, STRPOS(primary_poc, ' ') -1 ) first_name,  
        RIGHT(primary_poc, LENGTH(primary_poc) - STRPOS(primary_poc, ' ')) last_name, 
        name
 FROM accounts)
SELECT first_name, last_name, 
       CONCAT(first_name, '.', last_name, '@', name, '.com')
FROM t1;
```

- You may have noticed that in the previous solution some of the company names include spaces, which will certainly not work in an email address. See if you can create an email address that will work by removing all of the spaces in the account **name**, but otherwise your solution should be just as in question 1. Some helpful documentation is [here](https://www.postgresql.org/docs/8.1/functions-string.html).

```javascript
WITH t1 AS (
 SELECT LEFT(primary_poc, STRPOS(primary_poc, ' ') -1 ) first_name,  
        RIGHT(primary_poc, LENGTH(primary_poc) - STRPOS(primary_poc, ' ')) last_name, 
        name
 FROM accounts)
SELECT first_name, last_name,
       CONCAT(first_name, '.', last_name, '@', REPLACE(name, ' ', ''), '.com')
FROM  t1;
```

- We would also like to create an initial password, which they will change after their first log in. The first password will be the `first letter of the **primary_poc**'s first name (lowercase)`, then the `last letter of their first name (lowercase)`, the `first letter of their last name (lowercase)`, the `last letter of their last name (lowercase)`, the `number of letters in their first name`, the `number of letters in their last name`, and then the `name of the company they are working with, all capitalized with no spaces`.

```javascript
WITH t1 AS (
 SELECT LEFT(primary_poc, STRPOS(primary_poc, ' ') -1 ) first_name,  
        RIGHT(primary_poc, LENGTH(primary_poc) - STRPOS(primary_poc, ' ')) last_name, 
        name
 FROM accounts)
SELECT first_name, last_name, 
       CONCAT(first_name, '.', last_name, '@', name, '.com'), 
       LEFT(LOWER(first_name), 1) || RIGHT(LOWER(first_name), 1) || LEFT(LOWER(last_name), 1) || RIGHT(LOWER(last_name), 1) || LENGTH(first_name) || LENGTH(last_name) || REPLACE(UPPER(name), ' ', '')
FROM t1;
```
---
# TO_DATE, CAST, Casting with ::

**DATE_PART('month', TO_DATE(month, 'month'))** change a month name into the number associated with that particular month.

**CAST(date_column AS DATE)** change a string to a date. CAST could change lots of column types, [other examples](https://www.postgresqltutorial.com/postgresql-cast/).

**date_column::DATE**

> LEFT, RIGHT, and TRIM(remove characters from the beginning and end of a string) are used to select only certain elements of strings, but using them to select a number or date will treat them as strings for the purpose of the function.
Other string functions covered in [Postgres literature](https://www.postgresql.org/docs/9.1/functions-string.html)