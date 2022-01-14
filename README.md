# SQL-Challenge
Posting all the SQL practice solutions for #100daysofcode challenge
### WEEK 1 
1. Given the tables above, select the top 3 departments with at least ten employees and rank them according to the percentage of their employees making over 100K in salary.

![employees_salaries](https://user-images.githubusercontent.com/7839090/149487374-2029285a-a8ad-434e-b663-57d566e0d948.PNG)

```
SELECT
  SUM(CASE WHEN a.salary > 100000 THEN 1 ELSE 0) / COUNT(DISTINCT a.id) AS perc_100k,
  b.name AS department_name,
  COUNT(DISTINCT a.id) AS num_employees
FROM 
  employee a JOIN departments b ON a.id = b.id 
GROUP BY 2
HAVING COUNT(a.id) >= 10 
ORDER BY 1 DESC LIMIT 2;
```

2. Given a table of students and their SAT test scores, write a query to return the two students with the closest test scores with the score difference.If there are multiple students with the same minimum score difference, select the student name combination that is higher in the alphabet. 
![close sat scores](https://user-images.githubusercontent.com/7839090/149487846-9eced7c8-339c-41f8-8f58-a482b0f2abe0.PNG)

```
SELECT 
    s1.student AS student1
    , s2.student AS student2
    , ABS(s1.score - s2.score) AS score_diff
FROM scores s1
CROSS JOIN scores s2
WHERE s1.id < s2.id
ORDER BY 3,1,2
LIMIT 1;
```

3. Write a query that calculates the difference between the highest salaries found in the marketing and engineering departments. Output just the absolute difference in salaries.
![salary differences](https://user-images.githubusercontent.com/7839090/149487881-65eaf664-9fe6-456b-987a-951dbd2ee572.PNG)

```
SELECT
  ABS(
  (SELECT MAX(salary) FROM db_employee a JOIN db_dept b ON a.department_id = b.id WHERE department = 'marketing') - 
  (SELECT MAX(salary) FROM db_employee a JOIN db_dept b ON a.department_id = b.id WHERE department = 'engineering')) AS sal_diff ;
```

4. Acceptance Rate by Date:
What is the overall friend acceptance rate by date? Your output should have the rate of acceptances by the date the request was sent. Order by the earliest date to latest.
Assume that each friend request starts by a user sending (i.e., user_id_sender) a friend request to another user (i.e., user_id_receiver) thats logged in the table with action = 'sent'. If the request is accepted, the table logs action = 'accepted'. If the request is not accepted, no record of action = 'accepted' is logged.
![acceptance rate](https://user-images.githubusercontent.com/7839090/149487905-51cbf177-ddcf-4855-bb42-490b14171484.PNG)


```
SELECT 
  a.date, 
  count(b.user_id_receiver)/count(a.user_id_sender)::decimal
FROM
  (SELECT date, user_id_sender, user_id_receiver
  FROM fb_friend_requests
  WHERE action='sent') a
LEFT JOIN
  (SELECT date, user_id_sender, user_id_receiver
  FROM fb_friend_requests
  WHERE action='accepted') b
ON 
a.user_id_sender=b.user_id_sender 
AND a.user_id_receiver=b.user_id_receiver;
```

5. Highest Energy Consumption: 
Find the date with the highest total energy consumption from the Meta/Facebook data centers. Output the date along with the total energy consumption across all data centers.
![high consumption](https://user-images.githubusercontent.com/7839090/149487944-7a0e8bfc-58fd-49ea-a226-72c3e4c7aff2.PNG)

```
WITH union_cte AS (
SELECT *
    FROM fb_eu_energy
    UNION
    SELECT *
    FROM fb_asia_energy
    UNION
    SELECT *
    FROM fb_na_energy), 
totals as (
	SELECT date, SUM(consumption) AS total_consumption 
	FROM 
	union_cte GROUP BY 1)

SELECT * FROM totals WHERE total_consumption = (SELECT MAX(total_consumption) FROM totals);
```

6. Popularity Percentage: 
Find the popularity percentage for each user on Meta/Facebook. The popularity percentage is defined as the total number of friends the user has divided by the total number of users on the platform, then converted into a percentage by multiplying by 100.
Output each user along with their popularity percentage. Order records in ascending order by user id.
The 'user1' and 'user2' column are pairs of friends.
![facebook_friends](https://user-images.githubusercontent.com/7839090/149488122-80ea96b7-0a6e-4d86-ae41-190f96848695.PNG)

Hint:

- You’ll need to create two subqueries or CTEs to calculate total unique friends-pair and total number of friend per user
- To calculate total unique friends-pair, UNION the user1 and user2 columns. This will de-duplicate any users that are repeated in the column. You can ensure uniqueness by adding a DISTINCT. To calculate the total count of unique friends-pair, perform a count(*) on the subquery.
- To calculate the total number of friends per user, create one column that contains all users and a second column that contains all their friends. You can accomplish this by user1, user2 UNION user2, user1.
- JOIN the two subqueries together so that you can calculate the percentage of friends over total users in the platform. You’ll need to JOIN using a 1=1 relation as the key. The resulting table will be a list of users, their friends, and total number of users on the platform.
- Lastly, implement the percentage popularity formula by counting the number of friends per user and dividing by the total number of users on the platform.

```
WITH users_union AS
  (SELECT user1,
          user2
   FROM facebook_friends
   UNION SELECT user2 AS user1,
                user1 AS user2
   FROM facebook_friends)
SELECT user1,
       count(*)::float /
  (SELECT count(DISTINCT user1)
   FROM users_union)*100 AS popularity_percent
FROM users_union
GROUP BY 1
ORDER BY 1;
```




