# SQL_PROJECT_ON_NETFLIX
 Data Analysis on 8808 Rows in PostgreSQL (pgAdmin 4) This project involves performing data analysis on a dataset containing 8808 rows using PostgreSQL in the pgAdmin 4 environment. The dataset could represent various types of information, such as sales data, customer records, inventory management, or any domain-specific dataset.
![image](https://github.com/user-attachments/assets/1270e614-25ff-4d42-baf4-9da6db714eeb)
# Overview
______________________________________________________________________________
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

# Objectives
________________________________________________________________________________
1. Analyze the distribution of content types (movies vs TV shows).
2. Identify the most common ratings for movies and TV shows.
3. List and analyze content based on release years, countries, and durations.
4. Explore and categorize content based on specific criteria and keywords.
# Tools & Technologies:
1. PostgreSQL: The database management system used for querying and managing the dataset.
2. pgAdmin 4: The integrated development environment (IDE) used for connecting to PostgreSQL, creating and executing SQL queries, and managing the database schema.
3. SQL: Structured Query Language used to perform all data manipulation and retrieval tasks.

# Project Steps:
1. Dataset Upload & Schema Design: Loading the 8808 rows dataset into PostgreSQL and creating tables with appropriate column types.
2. Data Preprocessing: Using SQL commands to clean and prepare the data for analysis, ensuring no invalid or redundant records exist.
3. Data Analysis: Writing SQL queries to perform aggregations, filtering, and sorting to uncover valuable insights (such as average sales, customer demographics, etc.).
4. Performance Tuning: Indexing key columns and optimizing SQL queries for better performance.
5. Reporting: Generating reports and visualizations (if applicable) based on the insights derived from the dataset.
# -- netflix project 
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
	show_id	VARCHAR(8),
	type VARCHAR(10),
	title VARCHAR(150),
	director VARCHAR(220),
	casts VARCHAR(1000),
	country VARCHAR(150),
	date_added VARCHAR(50),
	release_year INT,
	rating VARCHAR(15),
	duration VARCHAR(15),
	listed_in VARCHAR(100),
	description VARCHAR(250)
);

SELECT * FROM netflix ;

SELECT 
  DISTINCT type
FROM netflix;

-- 15 Business Problems & Solutions

1. Count the number of Movies vs TV Shows.

SELECT 
	type,
	COUNT(*) as total_content
FROM netflix 
GROUP BY type
2. Find the most common rating for movies and TV shows


SELECT 
	type,
	rating
FROM
(
SELECT 
 	type,
	rating,
	COUNT(*),
	RANK() OVER(PARTITION BY type ORDER BY COUNT(*)DESC) as ranking
FROM netflix
GROUP BY 1,2
) as t1
WHERE
	RANKING = 1


3. List all movies released in a specific year (e.g., 2020)
------filter 2020
SELECT * FROM netflix
WHERE 
	type= 'Movie'
	AND
	release_year = 2020






4. Find the top 5 countries with the most content on Netflix
SELECT 
	UNNEST(STRING_TO_ARRAY(country,',')) as new_country
FROM netflix

SELECT
	UNNEST(STRING_TO_ARRAY(country,',')) as new_country,
	count(show_id) as total_content
FROM netflix
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5




5. Identify the longest movie

SELECT * FROM netflix 
WHERE 
	type = 'Movie'
    AND 
	duration = (SELECT MAX(duration) FROM netflix)
6. Find content added in the last 5 years


SELECT *
FROM netflix
WHERE
	TO_DATE(date_added,'Month DD, YYYY' )
>= CURRENT_DATE - INTERVAL '5 years'
SELECT CURRENT_DATE - INTERVAL '5 years'

7. Find all the movies/TV shows by director 'Rajiv Chilaka'!
 SELECT * FROM netflix 
WHERE director LIKE  '%Rajiv Chilaka%'


-- 8. List all TV shows with more than 5 seasons

SELECT *
FROM netflix
WHERE 
	TYPE = 'TV Show'
	AND
	SPLIT_PART(duration, ' ', 1)::INT > 5


-- 9. Count the number of content items in each genre

SELECT 
	UNNEST(STRING_TO_ARRAY(listed_in, ',')) as genre,
	COUNT(*) as total_content
FROM netflix
GROUP BY 1


-- 10. Find each year and the average numbers of content release by India on netflix. 
-- return top 5 year with highest avg content release !


SELECT 
	country,
	release_year,
	COUNT(show_id) as total_release,
	ROUND(
		COUNT(show_id)::numeric/
								(SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100 
		,2
		)
		as avg_release
FROM netflix
WHERE country = 'India' 
GROUP BY country, 2
ORDER BY avg_release DESC 
LIMIT 5


-- 11. List all movies that are documentaries
SELECT * FROM netflix
WHERE listed_in LIKE '%Documentaries'



-- 12. Find all content without a director
SELECT * FROM netflix
WHERE director IS NULL


-- 13. Find how many movies actor 'Salman Khan' appeared in last 10 years!

SELECT * FROM netflix
WHERE 
	casts LIKE '%Salman Khan%'
	AND 
	release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10


-- 14. Find the top 10 actors who have appeared in the highest number of movies produced in India.



SELECT 
	UNNEST(STRING_TO_ARRAY(casts, ',')) as actor,
	COUNT(*)
FROM netflix
WHERE country = 'India'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10

/*
Question 15:
Categorize the content based on the presence of the keywords 'kill' and 'violence' in 
the description field. Label content containing these keywords as 'Bad' and all other 
content as 'Good'. Count how many items fall into each category.
*/


SELECT 
    category,
	TYPE,
    COUNT(*) AS content_count
FROM (
    SELECT 
		*,
        CASE 
            WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) AS categorized_content
GROUP BY 1,2
ORDER BY 2


