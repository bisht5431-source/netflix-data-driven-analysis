# netflix-data-driven-analysis

<div align="center">

  <img width="2226" height="678" alt="logo" src="https://github.com/user-attachments/assets/d700891d-05c2-4d4d-b980-5258b7a5bf1c" />

  <br><br>

  # 📺 Netflix Content Analysis — SQL Project

  <p>
    <img src="https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql&logoColor=white" />
    <img src="https://img.shields.io/badge/SQL-Advanced-0052CC?style=for-the-badge&logo=databricks&logoColor=white" />
    <img src="https://img.shields.io/badge/Netflix-E50914?style=for-the-badge&logo=netflix&logoColor=white" />
    <img src="https://img.shields.io/badge/Data%20Analysis-✔-2E7D32?style=for-the-badge" />
    <img src="https://img.shields.io/badge/Queries-15%20Business%20Problems-6A1B9A?style=for-the-badge" />
    <img src="https://img.shields.io/badge/Rows-8%2C807%20Titles-E65100?style=for-the-badge" />
  </p>

  <p>
    <img src="https://img.shields.io/github/stars/bisht5431-source?style=social" />
    <img src="https://img.shields.io/github/followers/bisht5431-source?style=social" />
  </p>

  > **Solving 15 real-world business problems on Netflix's content library using advanced PostgreSQL — window functions, CTEs, string manipulation, and more.**

</div>

---

## 📌 Project Overview

This project performs an **end-to-end SQL analysis** on Netflix's global content dataset using **PostgreSQL**. The goal is to extract meaningful business insights, uncover content trends, and answer real questions about movies, TV shows, directors, actors, genres, countries, and ratings.

| 🔢 Dataset Size | 📁 Tables | 📋 Queries | 🛠️ Tool |
|---|---|---|---|
| 8,807 titles | 1 (`netflix`) | 15 business problems | PostgreSQL |

> 🎯 **Purpose** — Demonstrate advanced SQL skills including JOINs, aggregations, CTEs, window functions, string manipulation, date logic, and conditional categorisation on a real-world dataset.

---

## 🗂️ Dataset Structure

The table `netflix` contains **12 columns** covering every piece of content on Netflix:

| Column | Data Type | Description |
|---|---|---|
| `show_id` | `VARCHAR` | Unique identifier — Primary Key |
| `type` | `VARCHAR` | `Movie` or `TV Show` |
| `title` | `VARCHAR` | Name of the content |
| `director` | `VARCHAR` | Director(s) — may be NULL |
| `casts` | `VARCHAR` | Actors/actresses featured |
| `country` | `VARCHAR` | Country of production |
| `date_added` | `VARCHAR` | Date content was added to Netflix |
| `release_year` | `INT` | Original year of release |
| `rating` | `VARCHAR` | Content rating: PG, R, TV-MA, etc. |
| `duration` | `VARCHAR` | Minutes (Movies) or Seasons (TV Shows) |
| `listed_in` | `VARCHAR` | Genres / categories |
| `description` | `VARCHAR` | Short summary of the content |

---

## 🔍 15 Business Problems & SQL Solutions

### 🟢 Basic Level

---

#### ❓ Q1 — Count the number of Movies vs TV Shows

```sql
SELECT
    type,
    COUNT(*) AS total_content
FROM netflix
GROUP BY type;
```

| type | total_content |
|---|---|
| Movie | 6,131 |
| TV Show | 2,676 |

> 💡 **Insight:** Movies make up **70%** of Netflix's content library — TV Shows account for the remaining 30%.

---

#### ❓ Q2 — Find the most common rating for Movies and TV Shows

```sql
SELECT type, rating
FROM (
    SELECT
        type,
        rating,
        COUNT(*),
        RANK() OVER (PARTITION BY type ORDER BY COUNT(*) DESC) AS ranking
    FROM netflix
    GROUP BY type, rating
) AS t1
WHERE ranking = 1;
```

| type | rating |
|---|---|
| Movie | TV-MA |
| TV Show | TV-MA |

> 💡 **Insight:** **TV-MA** (Mature Audience) is the dominant rating across both content types — Netflix skews toward adult content.

---

#### ❓ Q3 — List all Movies released in 2020

```sql
SELECT *
FROM netflix
WHERE type = 'Movie'
  AND release_year = 2020;
```

> 💡 **Use Case:** Filter content by release year for catalogue freshness analysis or year-over-year comparison.

---

#### ❓ Q4 — Find the Top 5 Countries with the most content on Netflix

```sql
SELECT
    UNNEST(STRING_TO_ARRAY(country, ',')) AS new_country,
    COUNT(show_id) AS total_content
FROM netflix
GROUP BY new_country
ORDER BY total_content DESC
LIMIT 5;
```

| Country | Total Content |
|---|---|
| United States | 2,818 |
| India | 972 |
| United Kingdom | 419 |
| Canada | 259 |
| France | 211 |

> 💡 **Insight:** The **US dominates** with 32% of all content. India ranks 2nd — reflecting Netflix's strong push into the South Asian market.

---

#### ❓ Q5 — Identify the Longest Movie

```sql
SELECT *
FROM netflix
WHERE type = 'Movie'
  AND duration = (
      SELECT MAX(duration) FROM netflix WHERE type = 'Movie'
  );
```

> 💡 **Technique:** Subquery inside WHERE to dynamically find the maximum — no hardcoding required.

---

#### ❓ Q6 — Find content added in the Last 5 Years

```sql
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
```

> 💡 **Technique:** `TO_DATE()` converts text dates, `INTERVAL` enables dynamic filtering without hardcoded years.

---

### 🟡 Intermediate Level

---

#### ❓ Q7 — Find all Movies/TV Shows by Director 'Rajiv Chilaka'

```sql
SELECT *
FROM netflix
WHERE director ILIKE '%Rajiv Chilaka%';
```

> 💡 **Technique:** `ILIKE` performs case-insensitive pattern matching — essential for messy real-world text data.

---

#### ❓ Q8 — List all TV Shows with more than 5 Seasons

```sql
SELECT *
FROM netflix
WHERE type = 'TV Show'
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```

> 💡 **Technique:** `SPLIT_PART()` extracts the numeric season count from a text field like `"6 Seasons"`, then casts to INT for comparison.

---

#### ❓ Q9 — Count the number of content items in each Genre

```sql
SELECT
    UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre,
    COUNT(show_id) AS total_content
FROM netflix
GROUP BY genre
ORDER BY total_content DESC;
```

> 💡 **Technique:** `UNNEST` + `STRING_TO_ARRAY` splits comma-separated genre tags into individual rows for accurate counting.

---

#### ❓ Q10 — Find each year and the average % of content releases in India

```sql
SELECT
    country,
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        COUNT(show_id)::NUMERIC /
        (SELECT COUNT(show_id) FROM netflix WHERE country = 'India') * 100, 2
    ) AS avg_release
FROM netflix
WHERE country = 'India'
GROUP BY country, release_year
ORDER BY avg_release DESC
LIMIT 5;
```

> 💡 **Insight:** Identifies peak years of Indian content production on Netflix — useful for content strategy planning.

---

### 🔴 Advanced Level

---

#### ❓ Q11 — List all Movies that are Documentaries

```sql
SELECT *
FROM netflix
WHERE listed_in ILIKE '%Documentaries%';
```

---

#### ❓ Q12 — Find all content without a Director

```sql
SELECT *
FROM netflix
WHERE director IS NULL;
```

> 💡 **Insight:** Reveals data quality gaps — content missing director information may need editorial review.

---

#### ❓ Q13 — Find how many Movies actor 'Salman Khan' appeared in during the last 10 years

```sql
SELECT *
FROM netflix
WHERE casts ILIKE '%Salman Khan%'
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```

> 💡 **Technique:** Combines `ILIKE` text search with `EXTRACT(YEAR)` date logic for actor-specific trend analysis.

---

#### ❓ Q14 — Find the Top 10 Actors appearing in the highest number of Indian Movies

```sql
SELECT
    UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor,
    COUNT(*) AS appearances
FROM netflix
WHERE country = 'India'
GROUP BY actor
ORDER BY appearances DESC
LIMIT 10;
```

> 💡 **Technique:** `UNNEST` splits comma-separated cast lists into individual actors, enabling aggregate counting per person.

---

#### ❓ Q15 — Categorise content as 'Good' or 'Bad' based on keywords

```sql
WITH new_table AS (
    SELECT *,
        CASE
            WHEN description ILIKE '%kill%'
              OR description ILIKE '%violence%' THEN 'Bad_Content'
            ELSE 'Good_Content'
        END AS category
    FROM netflix
)
SELECT
    category,
    COUNT(*) AS total_content
FROM new_table
GROUP BY category;
```

> 💡 **Technique:** CTE + `CASE` + `ILIKE` combines multiple skills for content moderation classification at scale.

---

## 🚀 SQL Techniques Used

| Technique | Used In |
|---|---|
| `COUNT()` `GROUP BY` `ORDER BY` | Q1, Q4, Q9, Q10, Q14 |
| `RANK() OVER (PARTITION BY)` | Q2 |
| `CTE (WITH clause)` | Q15 |
| `SPLIT_PART()` | Q8 |
| `UNNEST()` + `STRING_TO_ARRAY()` | Q4, Q9, Q14 |
| `TO_DATE()` + `INTERVAL` | Q6 |
| `EXTRACT(YEAR FROM ...)` | Q13 |
| `ILIKE` pattern matching | Q7, Q11, Q13, Q14, Q15 |
| `CASE WHEN` | Q15 |
| Subquery in `WHERE` | Q5, Q10 |
| `IS NULL` filter | Q12 |
| `CAST / ::INT` type conversion | Q8 |

---

## 📊 Key Business Insights

```
📌 Movies dominate at 70% of all Netflix content
📌 TV-MA is the #1 rating — Netflix skews toward mature content  
📌 United States leads with 32% of all titles globally
📌 India ranks #2 in content volume — a priority growth market
📌 Content without directors flagged — data quality opportunity
📌 Top Indian actors identified — supports talent partnership decisions
```

---

## 📁 Repository Structure

```
netflix-sql-analysis/
│
├── 📄 README.md          ← You are here
├── 📄 Netflix.sql        ← All 15 SQL queries
└── 🖼️ logo.png           ← Netflix logo (banner image)
```

---

## 🛠️ How to Run This Project

```bash
# Step 1 — Clone the repository
git clone https://github.com/bisht5431-source/netflix-sql-analysis.git

# Step 2 — Open pgAdmin 4 or any PostgreSQL client

# Step 3 — Create the database
CREATE DATABASE netflix_db;

# Step 4 — Create the table and import data
-- Run the CREATE TABLE section inside Netflix.sql

# Step 5 — Execute all 15 queries
-- Run Netflix.sql query by query
```

---

## 📬 Connect With Me

<div align="center">

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/dataanalyst-manish)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/bisht5431-source)
[![Email](https://img.shields.io/badge/Email-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:alphainsights123@gmail.com)

</div>

---

<div align="center">

**⭐ If this project helped you, please star the repository — it helps other SQL learners find it.**

*Built by Manish Bisht — Data Analyst · SQL · Power BI · Python · Excel*

</div>
