# Disney+ Movies and TV Shows Data Analysis using SQL

![](https://github.com/saifulroni/Disneyplus_sql_project/blob/8dc363f08a9cb843f6405607c6ba3cc3e8c07607/Disney-Symbol.png)

## Overview
This project focuses on analyzing Disney+ movies and TV shows data using SQL. The aim is to extract actionable insights and answer critical business questions from the dataset.  
This README includes objectives, dataset details, SQL schema, business problems, solutions, and findings.

---

## Objectives

- Analyze and compare content types (Movies vs TV Shows).  
- Identify top genres, countries, and directors.  
- Find trends in release years and durations.  
- Detect keyword-based content categorization.  
- Extract top-performing actors and countries.

---

## Dataset

The data for this project is sourced from Kaggle:  
- **Dataset Link:** [Disney+ Dataset](https://www.kaggle.com/datasets/shivamb/disney-movies-and-tv-shows)

---

## Schema

```sql
DROP TABLE IF EXISTS disneyplus;
CREATE TABLE disneyplus (
    show_id       VARCHAR(10),
    type          VARCHAR(10),
    title         VARCHAR(200),
    director      VARCHAR(200),
    casts         VARCHAR(1000),
    country       VARCHAR(150),
    date_added    VARCHAR(50),
    release_year  INT,
    rating        VARCHAR(20),
    duration      VARCHAR(20),
    listed_in     VARCHAR(100),
    description   VARCHAR(250)
);
```

## Business Problems and Solutions

### 1. Find the Most Popular Genre Combination for Each Content Type

```sql
SELECT type,
	   listed_in,
	   count(*) AS content_count
FROM disneyplus
WHERE listed_in IS NOT NULL
	AND listed_in <> ''
GROUP BY type, listed_in
ORDER BY type, content_count DESC;
```

**Objective:** Identify which genres are most popular among movies and TV shows.

### 2. Find the most common rating for movies and TV shows.

```sql
SELECT type,
	   rating,
	   COUNT (*) AS content_count
FROM disneyplus
WHERE rating IS NOT NULL
	AND rating <> ''

GROUP BY type, rating
ORDER BY type, content_count DESC;
```

**Objective:** Determine the most frequent content rating per type.


### 3. Determine the year with the highest number of new TV Shows added to Disney+

```sql
SELECT release_year,
       COUNT(*) AS tv_show_count
FROM disneyplus
WHERE type = 'TV Show'
  AND release_year IS NOT NULL
GROUP BY release_year
ORDER BY tv_show_count DESC
LIMIT 1;
```

**Objective:** Identify the year Disney+ released the most TV shows.


### 4. Find the top 5 countries with the most content on Disney+.

```sql
WITH explore AS (
  SELECT TRIM(regexp_split_to_table(country, ',\s*')) AS country_name
  FROM disneyplus
  WHERE country IS NOT NULL
  AND country <> ''
),

country_counts AS (
  SELECT country_name,
  COUNT(*) AS content_count
  FROM explore
  WHERE country_name <> ''
  GROUP BY country_name
)

SELECT country_name, content_count
FROM country_counts
ORDER BY content_count DESC
LIMIT 5;
```

**Objective:** Discover which countries produce the most Disney+ content.


### 5. Find the average duration of movies by genre

```sql
WITH explore_listed_in AS (
  SELECT TRIM(regexp_split_to_table(listed_in, ',\s*')) AS listed_in_name,
  duration
  FROM disneyplus
  WHERE listed_in IS NOT NULL
  AND listed_in <> ''
  AND duration NOT LIKE 'Season%'
),

average_duration AS (
SELECT listed_in_name, 
ROUND(AVG(REGEXP_REPLACE(duration, '[^0-9]', '', 'g')::INT)) AS average_count
FROM explore_listed_in
GROUP BY listed_in_name
)

SELECT listed_in_name, average_count
FROM average_duration
ORDER BY average_count DESC;
```

**Objective:** Calculate average movie length across genres


### 6. Find content realeased in the last 3 years.

```sql
SELECT title, release_year
FROM disneyplus 
WHERE release_year >= (
	SELECT MAX (release_year)-2
	FROM disneyplus
)
ORDER BY release_year DESC;
```

**Objective:** View recent releases on Disney+

### 7. List all TV shows with more than 5 seasons

```sql
WITH tv_seasons AS(
SELECT *,
	   REGEXP_REPLACE (duration, '[^0-9]', '', 'g') ::INT as season_count
FROM disneyplus
WHERE type = 'TV Show'
)

SELECT title, duration
FROM tv_seasons
WHERE season_count > 5
ORDER BY season_count DESC;
```

**Objective:** Identify long-running TV shows.

### 8. Count the number of content items in each genre.

```sql
WITH genre_names AS (
SELECT
	TRIM(regexp_split_to_table(listed_in, ',\s*')) AS genre
FROM disneyplus
WHERE listed_in IS NOT NULL
	AND listed_in <> '')

SELECT genre, COUNT (*) AS genre_count
FROM genre_names
GROUP BY genre
ORDER BY genre_count DESC;
```

**Objective:** Explore genre diversity across Disney+ content

### 9. Find each year and the average number of content releases in India on Disney+. Return the top 5 years with the highest average content release.


```sql
WITH country_explore AS (
SELECT release_year,
	TRIM(regexp_split_to_table(country, ',\s*')) AS country_names
FROM disneyplus
WHERE country IS NOT NULL
	AND country <> '')

SELECT release_year, count(*) AS content_count
FROM country_explore
WHERE LOWER (country_names) = 'india'
GROUP BY release_year
ORDER BY content_count DESC
LIMIT 5;
```

**Objective:** Analyze India’s content contribution trend by year.


### 10. Find all content without a director.

```sql
SELECT *
FROM disneyplus
WHERE director IS NULL OR director = '';
```

**Objective:** Identify content lacking a listed director

### 11. Find how many movies actor 'Dwayne Johnson' appeared in during the last 10 years

```sql
WITH cast_explore AS (
  SELECT
    title,
    type,
    release_year,
    duration,
    director,
    listed_in,
    TRIM(regexp_split_to_table(casts, ',\s*')) AS cast_name
  FROM disneyplus
  WHERE casts IS NOT NULL
    AND casts <> ''
)

SELECT *
FROM cast_explore
WHERE LOWER(cast_name) = 'dwayne johnson'
  AND release_year >= (
    SELECT MAX(release_year) - 9
    FROM disneyplus
  )
ORDER BY release_year DESC;
```

**Objective:** Measure Dwayne Johnson’s recent Disney+ movie appearances.


### Find the top 10 actors who have appeared in the highest number of movies produced in India.

```sql
WITH cast_explore AS (
  SELECT
  	title,
	type,
    TRIM(regexp_split_to_table(casts, ',\s*')) AS cast_name
  FROM disneyplus
  WHERE casts IS NOT NULL
    AND casts <> ''
),

country_explore as(
	SELECT
		title,
		TRIM(regexp_split_to_table(country, ',\s*')) AS country_name
	FROM disneyplus
	WHERE country IS NOT NULL
		AND country <> ''
)

SELECT cast_explore.cast_name,
		count(*) AS movie_count
FROM cast_explore
JOIN country_explore
ON cast_explore.title = country_explore.title
WHERE LOWER (country_name) = 'india'
GROUP BY cast_explore.cast_name
ORDER BY movie_count DESC
LIMIT 10;
```

**Objective:** Identify the most featured actors in Indian Disney+ productions.

### 13. Categorize content based on the presence of the keywords 'fight' and 'wild' in the description field. Label content containing these keywords as 'action' and all other content as 'adventure'. Count how many items fall into each category.

```sql
SELECT 
	CASE 
		WHEN description ILIKE '%fight%' 
			OR description ILIKE '%wild%'
		THEN 'action'
		ELSE 'adventure'
	END AS content_category,
	count(*) AS item_count
FROM disneyplus
GROUP BY content_category;
```

**Objective:** Categorize titles by description keywords.

### 14. Identify the top 5 directors with the most content available on Disney+

```sql
WITH director_explore AS (
  SELECT
  	title,
	type,
    TRIM(regexp_split_to_table(director, ',\s*')) AS director_name
  FROM disneyplus
  WHERE director IS NOT NULL
    AND director <> ''
)

SELECT director_name,
	COUNT(*) AS content_count
FROM director_explore
GROUP BY director_name
ORDER BY content_count DESC
LIMIT 5;
```

**Objective:** Rank directors by total content

### 14. 15. Find all content where the title or description contains words like 'Christmas' or 'Holiday'

```sql
SELECT *
FROM disneyplus
WHERE title ILIKE '%Christmas%'
	OR title ILIKE '%Holiday%'
	OR description ILIKE '%Christmas%'
	OR description ILIKE '%Holiday%';
```

**Objective:** Retrieve festive content on Disney+


## Author - saifulroni

This project is part of my portfolio, showcasing the SQL skills essential for data analyst roles. If you have any questions, feedback, or would like to collaborate, feel free to get in touch. Thank you!
