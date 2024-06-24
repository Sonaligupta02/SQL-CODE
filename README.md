# SQL-CODE
#### 1) Provide the query for all movies that were released either during the month of August 1997, or on the day of February 1, 1998.

``` sql
 -- Select the title and release date of movies
SELECT title, release_date
FROM movies
-- Filter for movies released in August 1997 or on February 1, 1998
WHERE (EXTRACT(YEAR FROM release_date) = 1997 AND EXTRACT(MONTH FROM release_date) = 8)
   OR (release_date = '1998-02-01');
```

#### 2) Write a query that outputs the genre names of the movie “Toy Story”, and the name of the movie. We want your filter to be based on the name of the movie.
``` sql
-- Select the name of genres and titles of movies
SELECT g.name AS genres_name, m.title AS movie_name
FROM "genres_movies" gm 
LEFT JOIN "movies" m ON m.id = gm.movie_id
LEFT JOIN "genres" g ON g.id = gm.genre_id
-- Filter for movies with titles starting with 'Toy Story'
WHERE m.title LIKE 'Toy Story%';
```

#### 3) Is there any scenario where the same user provided multiple ratings on the same movie? Answer this along with the query. 
Answer: No scenario where the same user provided multiple ratings on the same movie
``` sql
SELECT user_id, movie_id, COUNT(rating) as num_ratings
FROM "ratings"
GROUP BY user_id, movie_id
HAVING COUNT(rating) > 0
order by 1 asc
```

#### 4) Write a query that outputs, (1) the age, (2) the gender, and (3) the id of the 10 users that provided the most ratings in 1998, along with (4) the number of ratings they provided (5) the number of distinct movies they rated (6) the average rating they gave to the movies. 
How would you adapt or change your query to output the 1st, the 2nd, the 9th and the 10th users who provided the biggest amount of ratings? 
  ``` sql
-- Filter and aggregate ratings data for the year 1998
WITH ratings_1998 AS (
    SELECT
        r.user_id, 
        COUNT(r.id) AS num_ratings, 
        COUNT(DISTINCT r.movie_id) AS num_distinct_movies, 
        AVG(r.rating) AS avg_rating 
    FROM ratings r
    WHERE EXTRACT(YEAR FROM r.rated_at) = 1998
    GROUP BY r.user_id
),

-- Rank users based on the number of ratings provided in 1998
ranked_users AS (
    SELECT
        user_id, 
        num_ratings, 
        num_distinct_movies, 
        avg_rating,
        ROW_NUMBER() OVER (ORDER BY num_ratings DESC) AS user_rank
    FROM ratings_1998
)

-- Select the 1st, 2nd, 9th, and 10th users based on the ranking
SELECT
    u.id AS user_id, 
    u.age, 
    u.gender, 
    ru.num_ratings, 
    ru.num_distinct_movies, 
    ru.avg_rating 
FROM ranked_users ru
JOIN users u ON ru.user_id = u.id
WHERE ru.user_rank IN (1, 2, 9, 10)
ORDER BY ru.user_rank;

```
![image](https://github.com/Sonaligupta02/SQL-CODE/assets/42031754/041f9b52-7506-4ac5-acf8-de7c188723d3)


#### 5) Write a query that outputs, for the year of 1997, (1) the month number (2) the number of movies that were released in each month, (3) the percentage it represents, compared to the total amount of movies released during that year. 
 Note: to output the month number, you can use the TO_CHAR(timestamp, ‘MM’) function 
``` sql
WITH movies_1997 AS (
    -- Filter movies released in the year 1997
    SELECT
        id,
        TO_CHAR(release_date, 'MM') AS month
    FROM movies
    WHERE EXTRACT(YEAR FROM release_date) = 1997
),

monthly_counts AS (
    -- Count the number of movies released in each month
    SELECT
        month,
        COUNT(id) AS num_movies
    FROM movies_1997
    GROUP BY month
)

-- Final query to calculate the percentage of movies released each month
SELECT
    mc.month AS month_number,
    mc.num_movies,
   round((mc.num_movies * 100.0 / (SELECT COUNT(id) FROM movies_1997)),2) AS percentage_of_total
FROM monthly_counts mc
ORDER BY mc.month;

```
<img width="1067" alt="Screenshot 2024-06-24 at 23 48 46" src="https://github.com/Sonaligupta02/SQL-CODE/assets/42031754/e925f724-3f34-4627-a186-9cdfee37b654">



#### 6) We’re trying to figure out which age bucket is most active when rating movies. Consider 4 buckets: users where (1) age is between 0 and 20. (2) age is between 21 and 40. (3) age is between 41 and 60 (4) age is between 61 and 80 (5) age is between 81 and 100. Write a query that outputs: (1) the age bucket (2) the number of ratings provided by the bucket (3) the number of raters in the bucket (4) the average number of ratings per rater in the bucket (5) the average rating in the bucket. 
``` sql
-- CTE to classify users into age buckets
WITH age_buckets AS (
    SELECT
        CASE
            WHEN age BETWEEN 0 AND 20 THEN '0-20'
            WHEN age BETWEEN 21 AND 40 THEN '21-40'
            WHEN age BETWEEN 41 AND 60 THEN '41-60'
            WHEN age BETWEEN 61 AND 80 THEN '61-80'
            WHEN age BETWEEN 81 AND 100 THEN '81-100'
            ELSE 'other'
        END AS age_bucket,
        u.id AS user_id
    FROM users u
),

-- CTE to aggregate ratings data by age bucket
ratings_by_age_bucket AS (
    SELECT
        ab.age_bucket,
        COUNT(r.id) AS num_ratings,
        COUNT(DISTINCT r.user_id) AS num_raters,
       round(AVG(r.rating),2) AS avg_rating
    FROM age_buckets ab
    JOIN ratings r ON ab.user_id = r.user_id
    GROUP BY ab.age_bucket
)

-- Final query to calculate statistics for each age bucket
SELECT
    age_bucket,
    num_ratings,
    num_raters,
    round((num_ratings / num_raters),2) AS avg_ratings_per_rater,
    avg_rating
FROM ratings_by_age_bucket
ORDER BY avg_rating desc;

```
<img width="1001" alt="Screenshot 2024-06-24 at 23 54 46" src="https://github.com/Sonaligupta02/SQL-CODE/assets/42031754/d407ee67-e6d5-4497-b145-fc37d89d12b3">



