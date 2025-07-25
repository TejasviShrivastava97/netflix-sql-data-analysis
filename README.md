# 🎬 Netflix Movies and TV Shows Data Analysis using SQL

## 📌 Overview
This project presents a detailed SQL-based analysis of Netflix’s Movies and TV Shows dataset. Using a structured approach, the project uncovers key insights into content types, genres, ratings, countries, and more. It is aimed at developing and showcasing SQL skills essential for business and data analyst roles.

---

## 🎯 Objectives
- Determine the distribution of Movies vs TV Shows.
- Identify the most common content ratings.
- Analyze content by release year, country, and duration.
- Explore trends by director, actor, and genre.
- Categorize and flag content based on specific keywords.

---

## 📂 Dataset
- **Source**: [Netflix Dataset on Kaggle](https://www.kaggle.com/datasets/shivamb/netflix-shows)
- **Format**: CSV
- **Loaded into**: PostgreSQL (but can also be used in MySQL)

---

## 🧱 Schema

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

---

## 💼 Business Problems Solved

1. **Movies vs TV Shows Count**
2. **Most Common Rating by Type**
3. **Content Released in 2020**
4. **Top 5 Countries with Most Content**
5. **Longest Movie on Netflix**
6. **Content Added in Last 5 Years**
7. **All Content by Director ‘Rajiv Chilaka’**
8. **TV Shows with More Than 5 Seasons**
9. **Content Count by Genre**
10. **Top 5 Years with Highest Content Released in India**
11. **All Documentaries**
12. **Content with No Listed Director**
13. **Movies Featuring 'Salman Khan' in Last 10 Years**
14. **Top 10 Indian Actors by Appearances**
15. **Categorization by Keywords: 'Kill' or 'Violence'**

All SQL queries are available in the `sql_queries/` folder.

---

## 🔍 Key Findings

- **Content Split**: Netflix hosts a rich variety of Movies and TV Shows.
- **Ratings**: TV-14 and TV-MA are among the most frequent ratings.
- **Geographic Trends**: India and the US lead in content production.
- **Genres**: Drama, International Movies, and Documentaries are highly prevalent.
- **Actor Insights**: Frequent contributors can be identified by country and genre.
- **Content Safety**: Basic keyword filters can highlight potentially sensitive content.

---

## 🧰 Tools Used
- **SQL**: PostgreSQL (queries are transferable to MySQL)
- **Platform**: Kaggle (for dataset)
- **Editor**: VS Code / pgAdmin / DBeaver

---

## 📁 Project Structure
```
netflix-sql-data-analysis/
│
├── sql_queries/             # SQL query scripts
├── data/                    # Dataset files (optional or path-based)
├── README.md                # Project documentation
```

---

## 📃 License
This project is released under the [MIT License](https://opensource.org/licenses/MIT).

---

## 🙋‍♂️ About Me
I’m an aspiring data analyst passionate about solving real-world problems with data. This project showcases my SQL skills and analytical thinking.

Feel free to connect or explore more of my work on [GitHub](#), and stay tuned for more projects in data analytics.

---

## 💻 SQL Solutions for Business Problems

### 1. Movies vs TV Shows Count
```sql
SELECT type, COUNT(*) 
FROM netflix 
GROUP BY type;
```

### 2. Most Common Rating by Type
```sql
WITH RatingCounts AS (
    SELECT type, rating, COUNT(*) AS rating_count
    FROM netflix
    GROUP BY type, rating
),
RankedRatings AS (
    SELECT type, rating, rating_count,
           RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rank
    FROM RatingCounts
)
SELECT type, rating AS most_frequent_rating
FROM RankedRatings
WHERE rank = 1;
```

### 3. Content Released in 2020
```sql
SELECT * 
FROM netflix 
WHERE release_year = 2020;
```

### 4. Top 5 Countries with Most Content
```sql
SELECT *
FROM (
    SELECT UNNEST(STRING_TO_ARRAY(country, ',')) AS country, COUNT(*) AS total_content
    FROM netflix
    GROUP BY 1
) AS t
WHERE country IS NOT NULL
ORDER BY total_content DESC
LIMIT 5;
```

### 5. Longest Movie on Netflix
```sql
SELECT * 
FROM netflix 
WHERE type = 'Movie' 
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC 
LIMIT 1;
```

### 6. Content Added in the Last 5 Years
```sql
SELECT * 
FROM netflix 
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
```

### 7. All Content by Director ‘Rajiv Chilaka’
```sql
SELECT * 
FROM (
    SELECT *, UNNEST(STRING_TO_ARRAY(director, ',')) AS director_name 
    FROM netflix
) AS t 
WHERE director_name = 'Rajiv Chilaka';
```

### 8. TV Shows with More Than 5 Seasons
```sql
SELECT * 
FROM netflix 
WHERE type = 'TV Show' 
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```

### 9. Content Count by Genre
```sql
SELECT UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre, COUNT(*) AS total_content 
FROM netflix 
GROUP BY 1;
```

### 10. Top 5 Years with Highest Content Released in India
```sql
SELECT country, release_year, COUNT(show_id) AS total_release,
       ROUND(COUNT(show_id)::numeric / 
            (SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100, 2) AS avg_release
FROM netflix
WHERE country = 'India'
GROUP BY country, release_year
ORDER BY avg_release DESC
LIMIT 5;
```

### 11. All Documentaries
```sql
SELECT * 
FROM netflix 
WHERE listed_in LIKE '%Documentaries';
```

### 12. Content Without a Director
```sql
SELECT * 
FROM netflix 
WHERE director IS NULL;
```

### 13. Movies Featuring 'Salman Khan' in Last 10 Years
```sql
SELECT * 
FROM netflix 
WHERE casts LIKE '%Salman Khan%' 
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```

### 14. Top 10 Indian Actors by Appearances
```sql
SELECT UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor, COUNT(*) 
FROM netflix 
WHERE country = 'India' 
GROUP BY actor 
ORDER BY COUNT(*) DESC 
LIMIT 10;
```

### 15. Categorization by Keywords: 'Kill' or 'Violence'
```sql
SELECT category, COUNT(*) AS content_count 
FROM (
    SELECT CASE 
             WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad' 
             ELSE 'Good' 
           END AS category 
    FROM netflix
) AS categorized_content 
GROUP BY category;
```

