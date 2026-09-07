# SQL-IMDB-Project
USE imdb;
SELECT *FROM director_mapping;
SELECT *FROM genre;
SELECT *FROM movie;
SELECT *FROM ratings;
SELECT *FROM role_mapping;

-- Find the total number of rows in each table of the schema?
SELECT COUNT(*) FROM director_mapping;
SELECT COUNT(*) FROM genre;
SELECT COUNT(*) FROM movie;
SELECT COUNT(*) FROM ratings;
SELECT COUNT(*) FROM role_mapping;

SELECT Table_name, table_rows
FROM information_schema.tables
WHERE table_schema='imdb';

-- Which columns in the movie table have null values
SELECT *FROM movie
WHERE 
     country IS NULL
     OR worlwide_gross_income IS NULL
     OR languages IS NULL
     OR production_company IS NULL;

SELECT
   SUM(CASE WHEN id IS NULL THEN 1  ELSE 0 END) AS id_nulls_count,
   SUM(CASE WHEN title IS NULL THEN 1  ELSE 0 END) AS title_nulls_count,
   SUM(CASE WHEN year IS NULL THEN 1  ELSE 0 END) AS year_nulls_count,
   SUM(CASE WHEN date_published IS NULL THEN 1  ELSE 0 END) date_published_nulls_count,
   SUM(CASE WHEN duration IS NULL THEN 1  ELSE 0 END) AS duration_nulls_count,
   SUM(CASE WHEN country IS NULL THEN 1  ELSE 0 END) AS country_nulls_count,
   SUM(CASE WHEN worlwide_gross_income IS NULL THEN 1  ELSE 0 END) worlwide_gross_income_nulls_count,
   SUM(CASE WHEN languages IS NULL THEN 1  ELSE 0 END) AS languages_nulls_count,
   SUM(CASE WHEN production_company IS NULL THEN 1 ELSE 0 END) AS production_company_nulls_count
FROM movie;

-- Find the total number of movies released each year? How does the trend look month wise
SELECT year, COUNT(id) AS number_of_movies
FROM movie
GROUP BY year
ORDER BY year;

SELECT 
MONTH(date_published) AS month_num,
COUNT(id) AS number_of_movies
FROM movie
GROUP BY month_num
ORDER BY  number_of_movies DESC;

-- How many movies were produced in the USA or India in the year 2019??
SELECT COUNT(*) AS movie_count
FROM movie
WHERE year=2019
AND (country LIKE '%USA%'
OR country LIKE '%INDIA%');


SELECT COUNT(id) AS movie_count
FROM movie
WHERE country='USA' OR 'INDIA' 
AND year=2019;


-- Find the unique list of the genres present in the data set?
SELECT DISTINCT genre
FROM genre;

USE imdb;

-- Which genre had the highest number of movies produced overall
SELECT genre, COUNT(movie_id) AS movie_count
FROM genre
GROUP BY genre
ORDER BY movie_count DESC
LIMIT 1;

-- How many movies belong to only one genre?
SELECT COUNT(*) AS movie_count
FROM (
SELECT movie_id
FROM genre
GROUP BY movie_id
HAVING COUNT(genre)=1
) AS single_genre_movies;

-- .What is the average duration of movies in each genre? 
SELECT g.genre,
ROUND(AVG(m.duration),2) AS avg_duration
FROM movie AS m
INNER JOIN genre g ON m.id=g.movie_id 
GROUP BY g.genre
ORDER BY avg_duration DESC;

-- What is the rank of the ‘thriller’ genre of movies among all the genres in terms of number of movies produced
SELECT *
FROM (
       SELECT
       genre,
          COUNT(movie_id) AS movie_count,
          RANK() OVER (ORDER BY COUNT(movie_id) DESC) AS genre_rank
FROM genre
GROUP BY genre   
) AS ranked_genres 
WHERE genre = 'thriller'; 

-- Find the minimum and maximum values in  each column of the ratings table except the movie_id column?
SELECT
      MIN(avg_rating) AS min_avg_rating,
      MAX(avg_rating) AS max_avg_rating,
      MIN(total_votes) AS min_total_votes,
      MAX(total_votes) AS max_total_votes,
      MIN(median_rating) AS min_median_rating,
      MAX(median_rating) AS mix_median_rating
FROM ratings;

-- Which are the top 10 movies based on average rating?
WITH movie_summary AS (
  SELECT
    m.title,
    r.avg_rating,
          RANK() OVER (ORDER BY r.avg_rating DESC) AS movie_rank
          FROM ratings AS r
       INNER JOIN 
       movie AS m ON m.id=r.movie_id)
 SELECT *FROM movie_summary
 WHERE movie_rank<=10;
 
 USE imdb;
 
 -- Summarise the ratings table based on the movie counts by median ratings.
 SELECT median_rating, COUNT(movie_id) AS movie_count
 FROM ratings
 GROUP BY median_rating
 ORDER BY movie_count DESC;
 
 -- Which production house has produced the most number of hit movies (average rating > 8)??
SELECT production_company, 
       COUNT(movie_id) AS movie_count,
       DENSE_RANK() OVER(ORDER BY COUNT(movie_id) DESC) AS prod_company_rank
FROM movie AS m
JOIN ratings AS r ON m.id=r.movie_id
WHERE r.avg_rating>8 
AND m.production_company IS NOT NULL
GROUP BY production_company
ORDER BY movie_count DESC;

USE imdb;
 
 -- How many movies released in each genre during March 2017 in the USA had more than 1,000 votes?
 SELECT g.genre, COUNT(m.id) AS movie_count
 FROM movie AS m
 JOIN genre AS g ON m.id=g.movie_id
 JOIN ratings AS r ON m.id=r.movie_id
 WHERE m.year=2017
 AND MONTH(m.date_published)=3
 AND m.country LIKE '%USA%'
 AND r.total_votes>1000
 GROUP BY g.genre
 ORDER BY movie_count DESC;
 
 -- Q15. Find movies of each genre that start with the word ‘The’ and which have an average rating > 8?
SELECT g.genre, m.title, r.avg_rating
FROM movie AS m
JOIN genre AS g ON m.id=g.movie_id
JOIN ratings AS r ON m.id=r.movie_id
WHERE m.title LIKE '%THE%'
AND r.avg_rating >8
ORDER BY r.avg_rating DESC;

-- Q16. Of the movies released between 1 April 2018 and 1 April 2019, how many were given a median rating of 8?
SELECT COUNT(*) AS movie_count
FROM movie AS m
JOIN ratings AS r ON m.id=r.movie_id
WHERE m.date_published BETWEEN '2018-04-01' AND '2019-04-01'
AND r.median_rating = 8;

USE imdb;
 
 -- Q17. Do German movies get more votes than Italian movies? 
 SELECT languages, SUM(total_votes) AS total_votes
 FROM movie m
 JOIN ratings r ON m.id=r.movie_id
 WHERE languages LIKE '%GERMAN%'
 OR languages LIKE '%ITALIAN%'
 GROUP BY languages;
 
 WITH languagesvotes AS (
 SELECT
     CASE
     WHEN m.languages LIKE '%GERMAN%' THEN 'German'
     WHEN m.languages LIKE '%ITALIAN%' THEN 'Italian'
     END AS movie_language,
     r.total_votes
FROM movie m
JOIN ratings r ON m.id=r.movie_id
WHERE languages LIKE '%GERMAN%'
 OR languages LIKE '%ITALIAN%'   
 )
 SELECT movie_language, SUM(total_votes) AS total_votes
 FROM languagesvotes
GROUP BY movie_language
LIMIT 1;

-- Q18. Which columns in the names table have null values??
SELECT 
     COUNT(CASE WHEN name IS NULL THEN 1 END) AS name_null,
     COUNT(CASE WHEN height IS NULL THEN 1 END) AS height_null,
     COUNT(CASE WHEN date_of_birth IS NULL THEN 1 END) AS date_of_birth_null,
     COUNT(CASE WHEN known_for_movies IS NULL THEN 1 END) AS known_for_movies_null
FROM names;

-- Q19. Who are the top three directors in the top three genres whose movies have an average rating > 8?
WITH topgenres AS (
SELECT genre g
FROM genre g
JOIN ratings r ON g.movie_id=r.movie_id
WHERE r.avg_rating>8
GROUP BY genre
ORDER BY COUNT(g.movie_id) DESC
LIMIT 3
)
SELECT n.name AS director_name, COUNT(d.movie_id) AS movie_count
FROM director_mapping AS d
JOIN names AS n ON d.name_id=n.id
JOIN ratings AS r ON d.movie_id=r.movie_id
JOIN genre g ON d.movie_id=g.movie_id
WHERE r.avg_rating > 8 
GROUP BY n.name  
ORDER BY movie_count DESC
LIMIT 3;

-- Q20. Who are the top two actors whose movies have a median rating >= 8?
WITH topactors AS (
SELECT n.name AS actor_name, COUNT(m.id) AS movie_count,
DENSE_RANK() OVER (ORDER BY COUNT(m.id) DESC) AS actor_rank
FROM role_mapping AS rm
JOIN movie AS m ON m.id=rm.movie_id
JOIN ratings AS r ON m.id=r.movie_id
JOIN names AS n ON rm.name_id=n.id
WHERE r.median_rating>=8
     AND rm.category='actor'
GROUP BY n.name
)
SELECT actor_name, movie_count
FROM topactors
WHERE actor_rank<=2;

-- Q21. Which are the top three production houses based on the number of votes received by their movies?
SELECT production_company, SUM(total_votes) AS vote_count,
DENSE_RANK() OVER(ORDER BY SUM(total_votes) DESC) AS prod_comp_rank
FROM movie AS m
JOIN ratings AS r ON m.id=r.movie_id
GROUP BY production_company
ORDER BY vote_count DESC
LIMIT 3;

-- Q22. Rank actors with movies released in India based on their average ratings. Which actor is at the top of the list?
SELECT n.name AS actor_name, SUM(r.total_votes) AS total_votes,
                 COUNT(m.id) AS movie_count,
                 SUM(avg_rating) AS actor_avg_rating,
				 DENSE_RANK() OVER(ORDER BY SUM(avg_rating) DESC) AS actor_rank
FROM names AS n
JOIN role_mapping AS rm ON n.id=rm.name_id
JOIN movie AS m ON rm.movie_id=m.id
JOIN ratings AS r ON m.id=r.movie_id                  
WHERE m.country LIKE '%INDIA%'
AND rm.category = 'actor'
GROUP BY n.name
HAVING movie_count >=5;

SELECT n.name AS actor_name, 
SUM(total_votes) AS vote_count, COUNT(m.id) AS movie_count,
ROUND(SUM(avg_rating*total_votes)/SUM(total_votes),2) AS actor_avg_rating,
RANK() OVER(ORDER BY ROUND(SUM(avg_rating*total_votes)/SUM(total_votes),2) DESC) AS actor_rank
FROM movie m
JOIN ratings r ON m.id=r.movie_id
JOIN role_mapping rm ON rm.movie_id=m.id
JOIN names n ON n.id=rm.name_id
WHERE category='actor'
AND country LIKE '%India%'
GROUP BY n.name
HAVING movie_count>=5;
 
-- Q23.Find out the top five actresses in Hindi movies released in India based on their average ratings? 
SELECT n.name AS actresses_name, COUNT(m.id) AS movie_count,
ROUND(SUM(avg_rating*total_votes)/SUM(total_votes),2) AS actresses_avg_rating,
RANK() OVER(ORDER BY ROUND(SUM(avg_rating*total_votes)/SUM(total_votes),2)DESC) AS actresses_rank
FROM movie m
JOIN ratings AS r ON m.id=r.movie_id
JOIN role_mapping AS rm ON r.movie_id=rm.movie_id
JOIN names AS n ON rm.name_id=n.id
WHERE category= 'actresses'
AND country LIKE '%INDIA%'
GROUP BY n.name
HAVING movie_count>=5;

-- Q24 Select thriller movies as per avg rating and classify them in the following category?
            
SELECT title AS movie_title, avg_rating,
    CASE
    WHEN avg_rating>8 THEN 'Superhit'
    WHEN avg_rating BETWEEN 7 AND 8 THEN 'Hit'
    WHEN avg_rating BETWEEN 5 AND 7 THEN 'One-time-watch'
    ELSE 'Flop'
END AS movie_rating
FROM movie m
JOIN ratings as r ON m.id=r.movie_id
JOIN genre AS g ON r.movie_id=g.movie_id
WHERE genre='thriller';

USE imdb;

-- Q25. What is the genre-wise running total and moving average of the average movie duration? 
SELECT genre, ROUND(AVG(duration),2) AS avg_duration,
SUM(ROUND(AVG(duration),2)) OVER (ORDER BY genre ROWS UNBOUNDED PRECEDING) AS running_total_duration,
AVG(ROUND(AVG(duration),2)) OVER (ORDER BY genre ROWS 10 PRECEDING) AS moving_avg_duration
FROM movie m
JOIN genre g ON m.id=g.movie_id
GROUP BY genre
ORDER BY genre
LIMIT 5;

-- Q26. Which are the five highest-grossing movies of each year that belong to the top three genres? 
SELECT genre, year, title, worlwide_gross_income
FROM(
SELECT  g.genre, m.title, m.year, m.worlwide_gross_income,
RANK() OVER (PARTITION BY m.year ORDER BY CAST(REPLACE(REPLACE(m.worlwide_gross_income, '$',''), ' ','') AS UNSIGNED) DESC
)AS movie_rank
FROM movie m
JOIN genre g ON m.id=g.movie_id
WHERE g.genre IN (
SELECT genre
FROM (
SELECT genre
FROM genre
GROUP BY genre
ORDER BY COUNT(movie_id) DESC
LIMIT 3
) AS temp
)
) AS ranked_movies  
WHERE movie_rank <=5
ORDER BY year DESC, movie_rank ASC;

-- Q27.  Which are the top two production houses that have produced the highest number of hits (median rating >= 8) among multilingual movies?
WITH prod_houses AS (
SELECT production_company, COUNT(m.id) AS movie_count,
DENSE_RANK() OVER(ORDER BY COUNT(m.id) DESC) AS prod_comp_rank
FROM movie m
JOIN ratings AS r ON m.id=r.movie_id
WHERE median_rating>=8
AND POSITION(',' IN languages)>0
AND production_company IS NOT NULL
GROUP BY production_company)
SELECT *FROM prod_houses
WHERE prod_comp_rank<=2;

-- Q28. Who are the top 3 actresses based on number of Super Hit movies (average rating >8) in drama genre?
WITH actress_summary AS (
SELECT n.name AS actresses_name, SUM(total_votes) AS total_votes, COUNT(m.id) AS movie_count,
DENSE_RANK() OVER(ORDER BY COUNT(m.id) DESC) AS actress_rank
FROM movie m
JOIN genre AS g ON m.id=g.movie_id
JOIN ratings AS r ON m.id=r.movie_id
JOIN role_mapping AS rm ON m.id=rm.movie_id
JOIN names AS n ON rm.name_id=n.id
WHERE avg_rating>8
AND genre='Drama'
AND rm.category='actress'
GROUP BY n.name)
SELECT * FROM actress_summary
WHERE actress_rank<=3;

/* Q29. Get the following details for top 9 directors (based on number of movies)
Director id
Name
Number of movies
Average inter movie duration in days
Average movie ratings
Total votes
Min rating
Max rating
total movie durations*/

WITH director_summary AS (
SELECT dm.name_id AS director_id, n.name AS director_name, duration, avg_rating, total_votes, date_published, m.id AS movie_id,
LEAD(date_published) OVER(PARTITION BY n.name ORDER BY date_published) AS next_publish_date
FROM movie AS m
JOIN ratings AS r ON m.id=r.movie_id
JOIN director_mapping AS dm ON r.movie_id=dm.movie_id
JOIN names AS n ON dm.name_id=n.id)
SELECT director_id, director_name,
COUNT(movie_id) AS number_of_movies,
ROUND(SUM(DATEDIFF(next_publish_date, date_published))/(COUNT(movie_id)-1),2) AS avg_inter_movie_days,
ROUND(SUM(avg_rating*total_votes)/SUM(total_votes),2) AS avg_rating,
SUM(total_votes) AS total_votes,
MIN(avg_rating) AS min_avg_rating,
MAX(avg_rating) AS max_avg_rating,
SUM(duration) AS total_duration
FROM director_summary
GROUP BY director_id, director_name
ORDER BY number_of_movies DESC
LIMIT 9;



