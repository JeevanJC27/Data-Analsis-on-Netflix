# Netflix Data Analysis Using SQL

![LOGO](https://github.com/user-attachments/assets/c2ea1305-1ee6-40ea-8d7b-b0d48a8044b8)

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## Schema
``` SQL
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
    show_id      VARCHAR(5) PRIMARY KEY,
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   date,
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);
```

## Business Problems

1. Count the number of Movies vs TV Shows
2. Find the most common rating for movies and TV shows
3. List all movies released in a specific year (e.g., 2020)
4. Find the top 5 countries with the most content on Netflix
5. Identify the longest movie
6. Find content added in the last 5 years
7. Find all the movies/TV shows by director 'Rajiv Chilaka'!
8. List all TV shows with more than 5 seasons
9. Count the number of content items in each genre
10.Find each year and the average numbers of content release in India on netflix. 
return top 5 year with highest avg content release!
11. List all movies that are documentaries
12. Find all content without a director
13. Find how many movies actor 'Salman Khan' appeared in last 10 years!
14. Find the top 10 actors who have appeared in the highest number of movies produced in India.
15.Categorize the content based on the presence of the keywords 'kill' and 'violence' in 
the description field. Label content containing these keywords as 'Bad' and all other 
content as 'Good'. Count how many items fall into each category.

### 1. Count the number of Movies vs TV Shows
``` SQL
SELECT 
	type,
	COUNT(*) AS Total
FROM netflix
GROUP BY type;
```

### 2. Find the most common rating for movies and TV shows
``` SQL
SELECT 
	type,
	common_rating,
	total_rating_count
FROM (
	SELECT 
		type,
		rating AS common_rating,
		count(*) as total_rating_count,
		ROW_NUMBER() OVER(PARTITION BY type ORDER BY COUNT(*) DESC) AS rw
	FROM netflix
	GROUP BY type, rating
) as a
WHERE rw = 1;
```

### 3. List all movies released in a specific year (e.g., 2020)
``` SQL
SELECT 
	title
FROM netflix
WHERE release_year = 2020;
```

### 4. Find the top 5 countries with the most content on Netflix
``` SQL
SELECT
	TRIM(UNNEST(STRING_TO_ARRAY(country,','))) as country,
	COUNT(*) as total_content
FROM netflix
GROUP BY UNNEST(STRING_TO_ARRAY(country,','))
ORDER BY COUNT(*) DESC
LIMIT 5;
```

5. Identify the longest movie

SELECT
	title,
	duration
FROM netflix
WHERE type = 'Movie' 
	AND REPLACE(duration,'min','')::int = (
										SELECT
											MAX(REPLACE(duration,'min','')::int)
										FROM netflix
										WHERE type = 'Movie'
										);

6. Find content added in the last 5 years

SELECT
	title
FROM netflix
WHERE date_part('year',date_added) in(
									SELECT
										DISTINCT date_part('year',date_added)
									FROM netflix
									WHERE date_part('year',date_added) IS NOT NULL
									ORDER BY date_part('year',date_added) DESC
									LIMIT 5
									);

7. Find all the movies/TV shows by director 'Rajiv Chilaka'!

SELECT 
	*
FROM netflix
WHERE director = 'Rajiv Chilaka';

8. List all TV shows with more than 5 seasons

SELECT
	title,
	duration
FROM netflix
WHERE type = 'TV Show' AND left(duration,1)::int  > 5
ORDER BY duration;

9. Count the number of content items in each genre

SELECT
	TRIM(UNNEST(STRING_TO_ARRAY(listed_in,','))) as genre,
	COUNT(*) as total_content
FROM netflix
GROUP BY TRIM(UNNEST(STRING_TO_ARRAY(listed_in,',')))
ORDER BY total_content DESC;

10.Find each year and the average numbers of content release in India on netflix.
return top 5 year with highest avg content release!
	
SELECT
	release_year,
	COUNT(*) as total_content,
 	CEIL(COUNT(*)*100 ::numeric / (SELECT count(*) FROM netflix WHERE country = 'India')::numeric) as avg_content
FROM netflix
WHERE country = 'India'
GROUP BY release_year
ORDER BY avg_content DESC
LIMIT 5;

11. List all movies that are documentaries

SELECT
	title
FROM netflix
WHERE listed_in = 'Documentaries';

12. Find all content without a director

SELECT 
	title
FROM netflix 
WHERE director is null;

13. Find how many movies actor 'Salman Khan' appeared in last 10 years!

SELECT 
	title,casts
FROM netflix 
WHERE casts like '%Salman Khan%'
	AND date_part('year',date_added) IN (SELECT
																				DISTINCT date_part('year',date_added)
																			FROM netflix
																			WHERE date_part('year',date_added) IS NOT NULL
																			ORDER BY date_part('year',date_added) DESC
																			LIMIT 10
																			);


14. Find the top 10 actors who have appeared in the highest number of movies produced in India.

WITH CTE AS (
	SELECT
		director,
		COUNT(*) as total_movie_produced
	FROM netflix
	WHERE country = 'India' AND director IS NOT NULL
	GROUP BY director
	ORDER BY total_movie_produced DESC
	LIMIT 1
)
SELECT 
	UNNEST(STRING_TO_ARRAY(casts, ',')) as Actor,
	COUNT(*) as Total_apperance
FROM netflix
WHERE director = (SELECT director FROM CTE)
GROUP BY UNNEST(STRING_TO_ARRAY(casts, ','))
ORDER BY Total_apperance DESC
LIMIT 10;
	
15.
Categorize the content based on the presence of the keywords 'kill' and 'violence' in 
the description field. Label content containing these keywords as 'Bad' and all other 
content as 'Good'. Count how many items fall into each category.

SELECT 
	category,
	type,
	Count(*) as content_count
FROM (
	SELECT 
		*,
		CASE 
			WHEN description LIKE '%kill%' OR description LIKE '%violen%' THEN 'BAD' ELSE 'GOOD'
		END AS category
	FROM netflix) as a
GROUP BY category, type;

