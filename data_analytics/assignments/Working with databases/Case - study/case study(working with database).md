# Zomato Bangalore Restaurants — SQL Case Study

The queries below assume the Kaggle Zomato Bangalore CSV has been imported into a MySQL 8.0 table named `zomato` with its standard columns, including `name`, `location`, `rate`, `votes`, `cuisines`, `online_order`, and `approx_cost(for two people)`. If your table or column names differ, replace them accordingly. Ratings and costs are stored as text in many copies of this dataset, so the queries clean those values before using them.

## 1. Top 5 North Indian restaurants in Koramangala

This treats any location containing “Koramangala” as part of the area and excludes non-numeric ratings such as `NEW` or `-`.

```sql
SELECT
    name,
    location,
    cuisines,
    CAST(SUBSTRING_INDEX(TRIM(rate), '/', 1) AS DECIMAL(3, 1)) AS rating,
    votes
FROM zomato
WHERE LOWER(location) LIKE '%koramangala%'
  AND cuisines LIKE '%North Indian%'
  AND TRIM(rate) REGEXP '^[0-5]([.][0-9]+)?(/5)?$'
ORDER BY rating DESC, CAST(votes AS UNSIGNED) DESC, name
LIMIT 5;
```

## 2. Three most expensive cuisines by average cost for two

Cuisine entries are comma-separated in the source data. The query separates them so a restaurant serving multiple cuisines contributes its cost to each cuisine it serves. Costs that are blank or not numeric are excluded.

```sql
WITH RECURSIVE costed AS (
    SELECT
        TRIM(cuisines) AS cuisines,
        CAST(REPLACE(TRIM(`approx_cost(for two people)`), ',', '') AS UNSIGNED) AS cost_for_two
    FROM zomato
    WHERE cuisines IS NOT NULL
      AND TRIM(cuisines) <> ''
      AND TRIM(`approx_cost(for two people)`) REGEXP '^[0-9,]+$'
),
cuisine_parts AS (
    SELECT
        TRIM(SUBSTRING_INDEX(cuisines, ',', 1)) AS cuisine,
        cost_for_two,
        CAST(
            CASE
                WHEN LOCATE(',', cuisines) > 0
                THEN SUBSTRING(cuisines, LOCATE(',', cuisines) + 1)
                ELSE ''
            END AS CHAR(1000)
        ) AS remaining_cuisines
    FROM costed

    UNION ALL

    SELECT
        TRIM(SUBSTRING_INDEX(remaining_cuisines, ',', 1)) AS cuisine,
        cost_for_two,
        CAST(
            CASE
                WHEN LOCATE(',', remaining_cuisines) > 0
                THEN SUBSTRING(remaining_cuisines, LOCATE(',', remaining_cuisines) + 1)
                ELSE ''
            END AS CHAR(1000)
        ) AS remaining_cuisines
    FROM cuisine_parts
    WHERE remaining_cuisines <> ''
)
SELECT
    cuisine,
    ROUND(AVG(cost_for_two), 2) AS average_cost_for_two,
    COUNT(*) AS restaurant_count
FROM cuisine_parts
WHERE cuisine <> ''
GROUP BY cuisine
ORDER BY average_cost_for_two DESC
LIMIT 3;
```

## 3. Online-delivery restaurants rated below 3.0

The first query returns the restaurants for follow-up. It also shows cost and votes, which can help prioritize outreach.

```sql
WITH cleaned AS (
    SELECT
        name,
        location,
        cuisines,
        online_order,
        CAST(SUBSTRING_INDEX(TRIM(rate), '/', 1) AS DECIMAL(3, 1)) AS rating,
        CAST(REPLACE(TRIM(`approx_cost(for two people)`), ',', '') AS UNSIGNED) AS cost_for_two,
        CAST(votes AS UNSIGNED) AS votes
    FROM zomato
    WHERE TRIM(rate) REGEXP '^[0-5]([.][0-9]+)?(/5)?$'
      AND TRIM(`approx_cost(for two people)`) REGEXP '^[0-9,]+$'
)
SELECT name, location, cuisines, rating, cost_for_two, votes
FROM cleaned
WHERE LOWER(TRIM(online_order)) = 'yes'
  AND rating < 3.0
ORDER BY rating ASC, votes DESC, name;
```

Use this summary to check whether the low ratings cluster by location or cuisine before choosing a campaign:

```sql
WITH cleaned AS (
    SELECT
        location,
        cuisines,
        CAST(SUBSTRING_INDEX(TRIM(rate), '/', 1) AS DECIMAL(3, 1)) AS rating,
        CASE
            WHEN TRIM(`approx_cost(for two people)`) REGEXP '^[0-9,]+$'
            THEN CAST(REPLACE(TRIM(`approx_cost(for two people)`), ',', '') AS UNSIGNED)
            ELSE NULL
        END AS cost_for_two
    FROM zomato
    WHERE LOWER(TRIM(online_order)) = 'yes'
      AND TRIM(rate) REGEXP '^[0-5]([.][0-9]+)?(/5)?$'
)
SELECT
    location,
    cuisines,
    COUNT(*) AS low_rated_restaurants,
    ROUND(AVG(rating), 2) AS average_rating,
    ROUND(AVG(cost_for_two), 2) AS average_cost_for_two
FROM cleaned
WHERE rating < 3.0
GROUP BY location, cuisines
ORDER BY low_rated_restaurants DESC, average_rating ASC;
```

**Marketing recommendation:** Use the summary to target the actual pattern rather than assuming one. If poor ratings cluster in a location, offer localized promotions and work with those outlets on delivery handoff times. If they cluster around a cuisine, audit menu accuracy, packaging, and food quality for those dishes. If prices are high relative to the affected outlets, test value bundles or delivery-only offers. Pair any promotion with a feedback and service-recovery process; discounts alone are unlikely to resolve the causes of low ratings.

## 4. Restaurant counts by price segment

The boundaries follow the prompt: budget is below 400, mid-range is 400 through 800 inclusive, and premium is above 800. Rows without a usable cost are excluded.

```sql
WITH costed AS (
    SELECT
        CAST(REPLACE(TRIM(`approx_cost(for two people)`), ',', '') AS UNSIGNED) AS cost_for_two
    FROM zomato
    WHERE TRIM(`approx_cost(for two people)`) REGEXP '^[0-9,]+$'
)
SELECT
    CASE
        WHEN cost_for_two < 400 THEN 'Budget (below 400)'
        WHEN cost_for_two <= 800 THEN 'Mid-range (400-800)'
        ELSE 'Premium (above 800)'
    END AS market_segment,
    COUNT(*) AS restaurant_count
FROM costed
GROUP BY market_segment
ORDER BY MIN(cost_for_two);
```

## 5. Top 10 restaurant chains by number of outlets

The query groups names case-insensitively and normalizes repeated whitespace. It assumes each dataset row represents one outlet; the `outlet_count` is the number of rows for that chain.

```sql
WITH normalized_names AS (
    SELECT
        TRIM(REGEXP_REPLACE(name, '[[:space:]]+', ' ')) AS display_name,
        LOWER(TRIM(REGEXP_REPLACE(name, '[[:space:]]+', ' '))) AS chain_key
    FROM zomato
    WHERE name IS NOT NULL
      AND TRIM(name) <> ''
)
SELECT
    MIN(display_name) AS restaurant_chain,
    COUNT(*) AS outlet_count
FROM normalized_names
GROUP BY chain_key
ORDER BY outlet_count DESC, restaurant_chain
LIMIT 10;
```

**Validation:** Run the query after importing the CSV. Confirm that it returns at most 10 rows, each `outlet_count` equals the number of source rows with that normalized name, and the counts are in descending order. Name variants such as abbreviations or punctuation differences may still be counted as separate chains and should be reviewed before reporting.

## Execution note

The dataset was not present in the workspace when this case study was prepared, so the queries could not be run here and no restaurant names, cuisine averages, segment counts, or chain rankings are claimed as observed results. Import the Kaggle CSV into MySQL as `zomato` and run each query to produce those results.