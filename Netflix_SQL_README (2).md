# 📊 Netflix Movies & TV Shows Exploration with SQL

![Netflix Logo](https://github.com/najirh/netflix_sql_project/blob/main/logo.png)

## 📌 Introduction

This project presents an in-depth exploration of Netflix’s catalog of movies and TV series using SQL. The aim is to derive actionable insights and solve data-driven questions using SQL queries. This README outlines the key goals, challenges, query implementations, discoveries, and takeaways from the project.

## 🎯 Project Goals

- Analyze the split between movies and TV shows.
- Discover which ratings are most prevalent.
- Explore releases by year, country, and duration.
- Classify and filter content based on specific patterns or keywords.

## 📁 Dataset Info

The dataset was obtained from Kaggle:

- 🔗 [Netflix Shows Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## 🧱 Database Schema

```sql
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix (
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);
```

## ❓ Key Business Questions & SQL Solutions

### 1. Movie vs TV Show Count

```sql
SELECT type, COUNT(*) FROM netflix GROUP BY type;
```
**Purpose:** Get an overview of how much of each type exists.

---

### 2. Most Frequent Rating by Content Type

```sql
WITH RatingStats AS (
    SELECT type, rating, COUNT(*) AS count
    FROM netflix
    GROUP BY type, rating
),
Ranked AS (
    SELECT *, RANK() OVER (PARTITION BY type ORDER BY count DESC) AS rnk
    FROM RatingStats
)
SELECT type, rating AS top_rating
FROM Ranked
WHERE rnk = 1;
```
**Purpose:** Identify the rating most commonly assigned to each category.

---

### 3. Movies Released in 2020

```sql
SELECT * FROM netflix WHERE release_year = 2020;
```
**Purpose:** Filter titles launched in a particular year.

---

### 4. Top 5 Countries by Content Volume

```sql
SELECT * 
FROM (
    SELECT UNNEST(STRING_TO_ARRAY(country, ',')) AS country, COUNT(*) AS count
    FROM netflix
    GROUP BY country
) AS sub
WHERE country IS NOT NULL
ORDER BY count DESC
LIMIT 5;
```
**Purpose:** Discover which countries contribute the most content.

---

### 5. Longest Duration Movie

```sql
SELECT *
FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC;
```
**Purpose:** Retrieve the movie with the maximum runtime.

---

### 6. Content Added in the Last 5 Years

```sql
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
```
**Purpose:** Check recent additions to Netflix.

---

### 7. Titles Directed by 'Rajiv Chilaka'

```sql
SELECT *
FROM (
    SELECT *, UNNEST(STRING_TO_ARRAY(director, ',')) AS dir
    FROM netflix
) AS sub
WHERE dir = 'Rajiv Chilaka';
```
**Purpose:** Find works from a specific director.

---

### 8. TV Shows With Over 5 Seasons

```sql
SELECT *
FROM netflix
WHERE type = 'TV Show' AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```
**Purpose:** Filter long-running TV series.

---

### 9. Content Count by Genre

```sql
SELECT UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre, COUNT(*) AS total
FROM netflix
GROUP BY genre;
```
**Purpose:** Understand genre distribution.

---

### 10. Years with Highest Average Releases (India)

```sql
SELECT 
    country, release_year, COUNT(show_id) AS total,
    ROUND(
        COUNT(show_id)::NUMERIC / 
        (SELECT COUNT(*) FROM netflix WHERE country = 'India')::NUMERIC * 100, 2
    ) AS avg_release
FROM netflix
WHERE country = 'India'
GROUP BY country, release_year
ORDER BY avg_release DESC
LIMIT 5;
```
**Purpose:** Rank years by relative output of Indian content.

---

### 11. Documentary Films

```sql
SELECT * FROM netflix WHERE listed_in LIKE '%Documentaries';
```
**Purpose:** Find all documentary-style content.

---

### 12. Entries Without a Listed Director

```sql
SELECT * FROM netflix WHERE director IS NULL;
```
**Purpose:** Detect missing director information.

---

### 13. Salman Khan's Movies (Last Decade)

```sql
SELECT *
FROM netflix
WHERE casts LIKE '%Salman Khan%'
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```
**Purpose:** Track recent appearances by the actor.

---

### 14. Most Featured Actors in Indian Movies

```sql
SELECT UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor, COUNT(*) 
FROM netflix
WHERE country = 'India'
GROUP BY actor
ORDER BY COUNT(*) DESC
LIMIT 10;
```
**Purpose:** Identify top-billed stars in Indian content.

---

### 15. Content Categorization by Keywords

```sql
SELECT category, COUNT(*) AS count
FROM (
    SELECT 
        CASE 
            WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) AS labeled
GROUP BY category;
```
**Purpose:** Classify content as "Good" or "Bad" based on themes.

---

## 🔍 Insights & Summary

- **Diversity in Content:** A well-balanced variety of movies and TV series with different genres and durations.
- **Rating Patterns:** Common ratings reveal insights into viewer targeting.
- **Global Distribution:** Notable contributions from key content-producing countries.
- **Keyword Categorization:** Helps in profiling and understanding content nature.

This SQL-driven exploration offers a powerful perspective on Netflix's content and can support strategic content planning or personal recommendations.

