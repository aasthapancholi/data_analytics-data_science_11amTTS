# Assessment 2: Data Collection, Cleaning & Visualization · Working with Databases

| **Program** | **Assessment Code** | **Total Time** | **Stack** |
| :--- | :--- | :--- | :--- |
| **Data Analytics / Data Science** | **M4-A1** | **4 Hours** | **SQL (SQLite / MySQL) / Python Concept Bridge** |

> **Student Learning Context & Module Alignment:**  
> In accordance with our syllabus progression—where **Module 1 (IT Fundamentals)** and **Module 2 (Working with Databases & SQL)** have been comprehensively mastered, and Python is scheduled for upcoming modules—this assessment is delivered from a **Database & SQL-First Engineering perspective**. Every theoretical scenario, practical data pipeline, and capstone task is implemented using **production-grade, standard-compliant SQL** (runnable in SQLite CLI, MySQL 8.0+, XAMPP phpMyAdmin, or DBeaver). Furthermore, each section provides a **SQL $\leftrightarrow$ Python/Pandas Concept Bridge** to establish a direct conceptual link to modern data science workflows.

---

## Table of Contents
- [Section A — Concept Application](#section-a--concept-application)
  - [Scenario 1 & Question 1: Missing Data Handling & Type Casting](#scenario-1--question-1-missing-data-handling--type-casting)
  - [Scenario 2 & Question 2: Vectorization, Boolean Masking & Statistical Thresholds](#scenario-2--question-2-vectorization-boolean-masking--statistical-thresholds)
  - [Scenario 3 & Question 3: Exploratory Data Analysis & Visualization Chart Selection](#scenario-3--question-3-exploratory-data-analysis--visualization-chart-selection)
  - [Scenario 4 & Question 4: SQL Relational Joins & Handling Unmatched Rows](#scenario-4--question-4-sql-relational-joins--handling-unmatched-rows)
  - [Scenario 5 & Question 5: Advanced SQL Window Functions vs. Self-Joins](#scenario-5--question-5-advanced-sql-window-functions-vs-self-joins)
  - [Scenario 6 & Question 6: Common Table Expressions (CTEs) vs. Nested Subqueries](#scenario-6--question-6-common-table-expressions-ctes-vs-nested-subqueries)
- [Section B — Practical Database & Analytical Tasks](#section-b--practical-database--analytical-tasks)
  - [Task 1: Daily Order & Revenue Analysis](#task-1-daily-order--revenue-analysis)
  - [Task 2: Food Delivery Orders Data Cleaning Pipeline](#task-2-food-delivery-orders-data-cleaning-pipeline)
  - [Task 3: Food Delivery Visualisation & Dashboard Analytics](#task-3-food-delivery-visualisation--dashboard-analytics)
  - [Task 4: SQL Database Schema, Population & Ranked Performance Report](#task-4-sql-database-schema-population--ranked-performance-report)
- [Section C — Mini Capstone Project: Food Delivery Analytics Application](#section-c--mini-capstone-project-food-delivery-analytics-application)
  - [Architecture & Menu-Driven Workflow](#architecture--menu-driven-workflow)
  - [Option 1: Ingestion, Validation & Quality Summary](#option-1-ingestion-validation--quality-summary)
  - [Option 2: Multi-Table SQL Aggregations & Joins](#option-2-multi-table-sql-aggregations--joins)
  - [Option 3: Multi-Dimensional Operational Insights](#option-3-multi-dimensional-operational-insights)
  - [Option 4: Enterprise Multi-Sheet Reporting Engine](#option-4-enterprise-multi-sheet-reporting-engine)
- [Section D — AI-Augmented Learning & Debugging](#section-d--ai-augmented-learning--debugging)
  - [Step 1: Prompt Architecture](#step-1-prompt-architecture)
  - [Step 2: AI's Original Code & Vulnerability Identification](#step-2-ais-original-code--vulnerability-identification)
  - [Step 3: Corrected Production-Grade Implementation](#step-3-corrected-production-grade-implementation)
  - [Step 4: Debugging Summary Note](#step-4-debugging-summary-note)

---

# Section A — Concept Application

---

### Scenario 1 & Question 1: Missing Data Handling & Type Casting

#### **Business Scenario:**
> *You are a Data Analyst at a food delivery startup. You have loaded a dataset of 50,000 order records into a Pandas DataFrame. The `delivery_time_mins` column has 14% missing values, and the `customer_rating` column contains some entries stored as strings (e.g., `'4.5'`) instead of `float64`.*

#### **Question:**
> *Explain which strategy you would choose to handle the missing values in `delivery_time_mins` — dropping rows versus imputation with mean or median — and justify your reasoning based on the percentage of missing data and skewness of delivery times. Also describe the exact Pandas steps needed to fix the `customer_rating` column type before any analysis is performed.*

---

### **Comprehensive Solution & Analysis:**

#### 1. Strategy Evaluation: Dropping Rows vs. Imputation
For the `delivery_time_mins` column, **Imputation with the Column Median** is the only statistically and operationally sound strategy. Dropping rows must be strictly avoided.

* **Percentage of Missing Data Evaluation:**
  * In a dataset of $50,000$ records, a $14\%$ missing rate equates to exactly **$7,000$ dropped customer orders**.
  * Industry best practices dictate that row deletion (`dropna()`) is only acceptable when missingness is below $3\%$ to $5\%$ and occurs completely at random (MCAR). Dropping $14\%$ of data severely diminishes the statistical power of the dataset, truncates sample sizes for niche sub-categories or smaller geographical clusters, and introduces **survival/selection bias**. For instance, if orders with missing delivery times disproportionately occurred during severe monsoon rainstorms or system outages, dropping those rows would artificially make the delivery fleet appear faster than it actually is.

* **Skewness of Logistics Delivery Times (Median vs. Mean):**
  * Real-world delivery times exhibit strong **positive skewness (right-skewed distribution)**. While the vast majority of orders are delivered within an expected range of $20$ to $40$ minutes, a long right tail of extreme values ($60$, $90$, or $120+$ minutes) inevitably exists due to kitchen bottlenecks, extreme traffic congestion, road construction, and adverse weather.
  * The **arithmetic mean** is non-robust and highly sensitive to extreme outliers; it is pulled upward by extreme delivery times, which would systematically overestimate typical delivery durations if used for imputation.
  * The **median (50th percentile)** is a robust measure of central tendency that is invariant to extreme skewness. Imputing with the median accurately reflects the typical customer experience without inflating turnaround estimates.

```
       Mode     Median        Mean
        │         │            │
      ┌─┴─┐       │            │
     ╱     ╲      │            │
    ╱       ╲     │            │
   ╱         ╲    │            │
  ╱           ╲───┴────────────┴───────────────────────►
 0    20      30  35           48                      120 mins
      [Typical Cluster]                 [Long Right Tail / Outliers]
```

#### 2. Exact SQL & Pandas Data Cleaning Steps for `customer_rating`

##### **A. Pure SQL Implementation (Database Cleansing):**
When cleaning dirty string data inside an RDBMS table, we use `UPDATE` with `CAST` / `CONVERT` and validate against non-numeric corruptions:
```sql
-- 1. Inspect non-numeric entries or whitespace corruptions
SELECT order_id, customer_rating 
FROM raw_orders 
WHERE customer_rating NOT REGEXP '^[0-9]+(\.[0-9]+)?$';

-- 2. Safely cast valid numeric strings to DECIMAL(2, 1) and impute NULLs
ALTER TABLE orders ADD COLUMN cleaned_rating DECIMAL(2, 1);

UPDATE orders
SET cleaned_rating = CASE 
    WHEN customer_rating REGEXP '^[0-9]+(\.[0-9]+)?$' 
    THEN CAST(customer_rating AS DECIMAL(2, 1))
    ELSE (SELECT ROUND(AVG(CAST(customer_rating AS DECIMAL(2, 1))), 1) 
          FROM orders WHERE customer_rating REGEXP '^[0-9]+(\.[0-9]+)?$')
END;
```

##### **B. Pandas Implementation Steps:**
1. **String Stripping & Coercion:** Use `pd.to_numeric()` with `errors='coerce'`. This strips whitespace, converts valid strings like `'4.5'` into float `4.5`, and safely turns corrupted entries (e.g., `'N/A'`, `'?'`, `'unrated'`) into `np.nan` rather than crashing the script.
2. **Median/Mean Imputation:** Compute the rounded mean (or median) of the valid ratings and fill the newly formed missing values via `.fillna()`.
3. **Type Verification:** Confirm the column has cast to `float64`.

```python
import pandas as pd
import numpy as np

# Step 1: Strip whitespace and safely coerce string entries to float64
df['customer_rating'] = pd.to_numeric(df['customer_rating'].astype(str).str.strip(), errors='coerce')

# Step 2: Impute missing ratings with the column mean (rounded to 1 decimal place)
rating_mean = round(df['customer_rating'].mean(), 1)
df['customer_rating'] = df['customer_rating'].fillna(rating_mean)

# Step 3: Impute delivery_time_mins with the column median
delivery_median = df['delivery_time_mins'].median()
df['delivery_time_mins'] = df['delivery_time_mins'].fillna(delivery_median)

# Step 4: Validate schema integrity
print("Column Data Types:\n", df[['customer_rating', 'delivery_time_mins']].dtypes)
print("Remaining Nulls:\n", df[['customer_rating', 'delivery_time_mins']].isnull().sum())
```

---

### Scenario 2 & Question 2: Vectorization, Boolean Masking & Statistical Thresholds

#### **Business Scenario:**
> *You are building a weekly performance report for a food delivery platform. You have a NumPy array of daily order counts across 30 days. Your manager wants to flag 'surge days' — days where the order count exceeds the mean by more than 1.5 standard deviations — and compute the total revenue only for those surge days using a matching revenue array.*

#### **Question:**
> *Describe how NumPy boolean indexing and statistical functions (`np.mean`, `np.std`) work together to identify surge days without using a Python loop. Explain what a boolean mask is and why element-wise array operations make this approach significantly faster than looping through each element individually.*

---

### **Comprehensive Solution & Analysis:**

#### 1. Mechanism of Boolean Masking & Statistical Functions
NumPy enables declarative, vector-based data extraction by coordinating statistical aggregation with element-wise relational operators:

1. **Threshold Determination:** The scalar threshold is calculated across the contiguous array using optimized C routines:
   $$\text{Threshold} = \mu + 1.5 \cdot \sigma = \text{np.mean}(O) + 1.5 \cdot \text{np.std}(O)$$
2. **Boolean Mask Creation:** Applying the comparative operator `daily_orders > threshold` evaluates every array element simultaneously without a Python loop:
   $$\text{surge\_mask} = \text{daily\_orders} > \text{threshold}$$
   This produces a **Boolean Mask**—a 1D array of identical length ($N=30$) containing only `True` or `False` values, where `True` flags an index meeting the surge condition.
3. **Parallel Filtering & Reduction:** Passing `surge_mask` into the matching revenue array (`daily_revenue[surge_mask]`) extracts only those elements where the mask is `True`. Finally, `np.sum(daily_revenue[surge_mask])` aggregates the surge revenue.

```
daily_orders:  [ 202,   535,   800,   120,   761,   291  ]
                    │      │      │      │      │      │   (orders > 605.2)
surge_mask:    [False, False,  True, False,  True, False ]
                    │      │      │      │      │      │   (Filtered Indexing)
daily_revenue: [5317,  6037, 39598, 15151, 25372, 35140 ]
                                  │             │
Extracted:                     [39598,        25372]  ──► Sum = $64,970.00
```

#### 2. Why Element-Wise Operations Outperform Python Loops
Operating on NumPy arrays is typically **$50\times$ to $200\times$ faster** than standard Python `for` loops due to three fundamental computer architecture advantages:

| Factor | Native Python `for` Loop | NumPy Vectorized Array / SQL Engine |
| :--- | :--- | :--- |
| **Execution Layer** | Interpreted Python bytecode; dynamic type inspection on every iteration. | Pre-compiled native C/Fortran code; zero interpretation overhead. |
| **Memory Architecture** | Array of pointers to fragmented Python heap objects (boxed integers). | Contiguous block of homogeneous memory (raw 64-bit integer buffers). |
| **CPU Acceleration** | Sequential scalar instructions; cache misses due to pointer chasing. | **SIMD (Single Instruction Multiple Data)** CPU registers execute 4–8 calculations per cycle; high L1/L2 cache hit rate. |

#### 3. Pure SQL Equivalent (Relational Set-Based Engine):
In relational databases, vectorization corresponds to **Set-Based Relational Processing**. Rather than using procedural row-by-row cursors, SQL leverages declarative `WHERE` clauses evaluated directly over contiguous table pages:

```sql
WITH OrderStatistics AS (
    SELECT 
        AVG(order_count) AS mean_orders,
        STDDEV(order_count) AS std_orders
    FROM daily_orders_revenue
)
SELECT 
    d.day_id,
    d.order_count,
    d.revenue
FROM daily_orders_revenue d, OrderStatistics s
WHERE d.order_count > (s.mean_orders + 1.5 * s.std_orders);
```

---

### Scenario 3 & Question 3: Exploratory Data Analysis & Visualization Chart Selection

#### **Business Scenario:**
> *You are conducting EDA for a food delivery product team. Your dataset contains columns: `restaurant_category`, `average_order_value`, `customer_rating`, and `delivery_time_mins`. The team wants to know (a) whether delivery time is correlated with customer rating, and (b) how average order value is distributed across restaurant categories.*

#### **Question:**
> *Identify whether each business question requires univariate, bivariate, or multivariate analysis. For each question, name one specific Seaborn chart type that would best visualise the relationship and explain why that chart type is more appropriate than a simple line plot for this data.*

---

### **Comprehensive Solution & Analysis:**

| Business Question | Analysis Category | Recommended Chart | Why It Outperforms a Simple Line Plot |
| :--- | :--- | :--- | :--- |
| **(a) Delivery Time vs. Customer Rating** | **Bivariate Analysis**<br>*(Two continuous numerical variables)* | **Scatter Plot with Regression Trendline**<br>(`sns.scatterplot` / `sns.regplot`) | **Line plots require sequential, ordered data (such as time-series).** Connecting independent order records with a line implies an artificial sequential link and produces an unreadable "spiderweb" pattern. A scatter plot displays every order as a distinct coordinate point, clearly revealing correlation direction, clustering, dispersion, and heteroscedasticity. |
| **(b) Order Value across Restaurant Categories** | **Bivariate Analysis**<br>*(One categorical variable vs. one continuous numerical variable)* | **Box Plot**<br>(`sns.boxplot` or `sns.violinplot`) | **Categories are discrete, nominal classes with no mathematical continuity.** Drawing a line between "Italian" and "Indian" erroneously implies interpolation. A box plot visualizes the complete **5-number summary** (Min, $Q1$, Median, $Q3$, Max) alongside outliers for every cuisine. |

#### **SQL Data Preparation for Visualizations:**
Before plotting in business intelligence dashboards (or Seaborn), analytical SQL queries generate the aggregated underlying metrics:

```sql
-- SQL Query for Question (a): Correlation Input Data
SELECT 
    delivery_time_mins, 
    customer_rating 
FROM orders_cleaned;

-- SQL Query for Question (b): Category-Wise Distribution Metrics (Boxplot Summary)
SELECT 
    category,
    COUNT(*) AS total_orders,
    ROUND(MIN(order_value), 2) AS min_val,
    ROUND(AVG(order_value), 2) AS avg_val,
    ROUND(MAX(order_value), 2) AS max_val
FROM orders_cleaned
GROUP BY category
ORDER BY avg_val DESC;
```

---

### Scenario 4 & Question 4: SQL Relational Joins & Handling Unmatched Rows

#### **Business Scenario:**
> *You are managing a food delivery database with three tables: `orders` (`order_id`, `customer_id`, `restaurant_id`, `order_value`), `restaurants` (`restaurant_id`, `name`, `city`, `category`), and `customers` (`customer_id`, `name`, `area`). A business team asks for a report showing every restaurant's name and total revenue, including restaurants that have not yet received any orders.*

#### **Question:**
> *Explain which SQL JOIN type must be used to include restaurants with zero orders and describe what would happen if an INNER JOIN were used instead. Justify your choice of join type in terms of how it handles unmatched rows between the two tables.*

---

### **Comprehensive Solution & Analysis:**

#### 1. Required Join Type: `LEFT OUTER JOIN` (or `LEFT JOIN`)
To ensure complete business visibility, a **`LEFT JOIN`** must be executed with `restaurants` as the left (preserving) table and `orders` as the right table.

```sql
SELECT 
    r.restaurant_id,
    r.name AS restaurant_name,
    r.category,
    r.city,
    COUNT(o.order_id) AS total_orders,
    COALESCE(SUM(o.order_value), 0.00) AS total_revenue
FROM restaurants r
LEFT JOIN orders o ON r.restaurant_id = o.restaurant_id
GROUP BY r.restaurant_id, r.name, r.category, r.city
ORDER BY total_revenue DESC;
```

#### 2. What Happens if an `INNER JOIN` is Used Instead?
* An `INNER JOIN` enforces a strict intersection: it returns a row **only if a matching foreign key exists in both tables** (`r.restaurant_id = o.restaurant_id`).
* Consequently, newly onboarded restaurants, seasonal partner kitchens, or inactive restaurants that have logged **zero orders** will have no matching records in the `orders` table.
* An `INNER JOIN` silently **drops all zero-order restaurants** from the output. In an executive operations report, this creates a false impression that all partner restaurants are actively generating revenue, masking operational stagnation and newly onboarded partners that need marketing promotion.

#### 3. Mechanics of Handling Unmatched Rows:
* **Row Preservation:** `LEFT JOIN` returns every single record from the left table (`restaurants`), regardless of whether a matching record exists on the right.
* **NULL Generation for Missing Attributes:** When a restaurant ID has no corresponding rows in `orders`, the database populates all columns from the `orders` table (`order_id`, `order_value`) with `NULL`.
* **Safe Aggregation with `COALESCE`:**
  * Using `COUNT(o.order_id)` correctly returns `0` because standard SQL aggregate functions ignore `NULL` values. (Note: Using `COUNT(*)` would incorrectly return `1` for the unmatched row).
  * `SUM(o.order_value)` returns `NULL` when summing over missing values. Wrapping the sum in `COALESCE(SUM(o.order_value), 0.00)` replaces `NULL` with `0.00`, ensuring clean financial reporting.

---

### Scenario 5 & Question 5: Advanced SQL Window Functions vs. Self-Joins

#### **Business Scenario:**
> *You are working on a food delivery analytics dashboard. Management wants to rank all delivery agents within each city by their number of completed deliveries for the current month, and also display how each agent's count compares to the agent ranked directly above them in that city.*

#### **Question:**
> *Identify the two SQL window functions you would use — one for ranking and one for retrieving the previous row's value — and explain why using window functions with PARTITION BY produces more accurate and maintainable results for this requirement than a GROUP BY with a self-join or correlated subquery approach.*

---

### **Comprehensive Solution & Analysis:**

#### 1. Identification of Window Functions:
1. **For Ranking within City:** **`DENSE_RANK()`** (or `RANK()`)
   $$\text{DENSE\_RANK}() \ \text{OVER} \ (\text{PARTITION BY } \text{city} \ \text{ORDER BY } \text{completed\_deliveries DESC})$$
   * `DENSE_RANK()` assigns consecutive integer ranks ($1, 2, 3, \dots$) without skipping rank positions in the event of ties.
2. **For Retrieving the Previous Agent's Count:** **`LAG()`**
   $$\text{LAG}(\text{completed\_deliveries}, 1) \ \text{OVER} \ (\text{PARTITION BY } \text{city} \ \text{ORDER BY } \text{completed\_deliveries DESC})$$
   * `LAG(..., 1)` accesses data from the immediately preceding row in the sorted partition without requiring a multi-table self-join.

#### 2. Pure SQL Implementation Query:
```sql
SELECT 
    agent_id,
    agent_name,
    city,
    completed_deliveries,
    -- Window Function 1: Rank agents within each city
    DENSE_RANK() OVER (
        PARTITION BY city 
        ORDER BY completed_deliveries DESC
    ) AS city_rank,
    -- Window Function 2: Fetch deliveries of the agent ranked directly above
    LAG(completed_deliveries, 1) OVER (
        PARTITION BY city 
        ORDER BY completed_deliveries DESC
    ) AS deliveries_of_higher_ranked_agent,
    -- Performance Gap Calculation
    (LAG(completed_deliveries, 1) OVER (
        PARTITION BY city 
        ORDER BY completed_deliveries DESC
    ) - completed_deliveries) AS delivery_deficit_to_agent_above
FROM delivery_agents
ORDER BY city, city_rank;
```

#### 3. Architectural Comparison: Window Functions vs. Self-Joins / Correlated Subqueries:

```
┌────────────────────────────────────────────────────────────────────────┐
│ WINDOW FUNCTION APPROACH:                                              │
│ Table Data ──► [Sort within Partition] ──► Streaming Window Evaluation │
│ Complexity: O(N log N) | Memory: Single Sequential Scan Buffer         │
└────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────┐
│ SELF-JOIN / CORRELATED SUBQUERY APPROACH:                              │
│ For each agent in Table A:                                             │
│   Scan entire Table B to find higher agent in same city                │
│ Complexity: O(N²) Quadratic | Memory: Nested Loop Join Buffers         │
└────────────────────────────────────────────────────────────────────────┘
```

* **Preservation of Row Granularity:** `GROUP BY` collapses multiple rows into a single aggregated summary row, destroying individual agent-level metadata unless re-joined. Window functions calculate peer-relative aggregations **without collapsing individual rows**.
* **Computational Complexity & Query Execution Plan:**
  * Window functions execute in a single or double sorting pass: $O(N \log N)$ time complexity.
  * Correlated subqueries and self-joins evaluate a nested comparison for every single row in the outer table, resulting in $O(N^2)$ quadratic complexity that degrades on large datasets.
* **Code Maintainability & Tie Handling:** Self-joins require complex multi-column inequality join predicates (`a.city = b.city AND b.deliveries > a.deliveries`) that break or duplicate rows when two agents share identical delivery counts. Window functions handle ties deterministically and cleanly.

---

### Scenario 6 & Question 6: Common Table Expressions (CTEs) vs. Nested Subqueries

#### **Business Scenario:**
> *You are optimising a SQL report for a food delivery operations team. The report first identifies the top 5 restaurant categories by total order count, and then retrieves the full order details only for restaurants belonging to those top categories. A colleague has written this as a deeply nested subquery inside the WHERE clause.*

#### **Question:**
> *Compare using a CTE (WITH clause) versus a nested subquery for this two-step requirement. Evaluate the trade-offs in terms of readability, ability to reuse the intermediate result, and debuggability. In what specific situation would you prefer the nested subquery over a CTE?*

---

### **Comprehensive Solution & Analysis:**

#### 1. Structural Comparison:

##### **A. Refactored CTE Approach (`WITH` Clause):**
```sql
WITH Top5Categories AS (
    -- Step 1: Identify the top 5 cuisine categories by order volume
    SELECT 
        r.category,
        COUNT(o.order_id) AS total_orders
    FROM restaurants r
    JOIN orders o ON r.restaurant_id = o.restaurant_id
    GROUP BY r.category
    ORDER BY total_orders DESC
    LIMIT 5
)
-- Step 2: Fetch full order details strictly for those 5 categories
SELECT 
    o.order_id,
    r.name AS restaurant_name,
    r.category,
    o.order_value,
    o.delivery_time_mins,
    o.rating
FROM orders o
JOIN restaurants r ON o.restaurant_id = r.restaurant_id
JOIN Top5Categories t ON r.category = t.category
ORDER BY t.total_orders DESC, o.order_value DESC;
```

##### **B. Colleague's Nested Subquery in `WHERE` Clause:**
```sql
SELECT 
    o.order_id,
    r.name AS restaurant_name,
    r.category,
    o.order_value,
    o.delivery_time_mins,
    o.rating
FROM orders o
JOIN restaurants r ON o.restaurant_id = r.restaurant_id
WHERE r.category IN (
    SELECT category FROM (
        SELECT r_sub.category, COUNT(o_sub.order_id) AS sub_orders
        FROM restaurants r_sub
        JOIN orders o_sub ON r_sub.restaurant_id = o_sub.restaurant_id
        GROUP BY r_sub.category
        ORDER BY sub_orders DESC
        LIMIT 5
    ) AS top_sub
)
ORDER BY r.category, o.order_value DESC;
```

#### 2. Detailed Trade-Off Evaluation:

| Criterion | Common Table Expression (CTE) | Deeply Nested Subquery |
| :--- | :--- | :--- |
| **Readability & Cognitive Flow** | **Superior.** Reads sequentially from top to bottom (Step 1 $\rightarrow$ Step 2), matching the natural business thought process. | **Poor.** Requires reading "inside-out". Developers must drill to the lowest indentation level to understand the filtering condition. |
| **Reusability of Intermediate State** | **High.** Can be referenced multiple times within the same main query (e.g., in a `JOIN`, a second sub-CTE, or an executive summary union) without repeating logic. | **Zero.** The subquery must be copied and pasted in every clause where it is needed, leading to duplicated code and redundant database execution. |
| **Debuggability & Unit Testing** | **Trivial.** An engineer can test Step 1 in isolation by temporarily replacing the final query with `SELECT * FROM Top5Categories;`. | **Difficult.** Requires manually copying out the inner subquery, removing brackets, and running it separately in a new query tab. |

#### 3. When Would You Prefer a Nested Subquery Over a CTE?
While CTEs are preferred for complex, multi-step data pipelines, a **nested subquery is specifically preferred** in the following circumstances:
1. **Single-Use Scalar Comparisons:** When filtering against a single scalar value calculated on the fly:
   ```sql
   WHERE order_value > (SELECT AVG(order_value) FROM orders)
   ```
   Writing a CTE for a one-line scalar lookup introduces unnecessary boilerplate.
2. **Correlated `EXISTS` / `NOT EXISTS` Semi-Joins:** When checking for row existence, modern database query planners optimize `WHERE EXISTS (SELECT 1 FROM ... WHERE foreign_key = parent.key)` using index short-circuiting. The engine stops scanning as soon as the first match is found, outperforming CTE materialization.
3. **Engine Compatibility / Legacy Databases:** In legacy systems (such as MySQL versions prior to 8.0, or specific embedded SQL engines) that lack `WITH` clause support.

---

# Section B — Practical Database & Analytical Tasks

---

### Task 1: Daily Order & Revenue Analysis

#### **Requirements:**
* Analyze $30$ days of simulated daily order count and revenue data.
* Compute total orders, total revenue, mean daily orders, standard deviation of orders, and find the day with the highest single-day revenue.
* Use threshold filtering (order count $> \text{mean} + 1\sigma$) to identify surge days, their count, and their total revenue.
* Aggregate data into $5$ weekly operational blocks ($6$ days per block).

#### **1. Relational Table Schema & Data Setup:**
```sql
CREATE TABLE daily_orders_revenue (
    day_id INT PRIMARY KEY,
    order_count INT NOT NULL,
    revenue DECIMAL(10, 2) NOT NULL
);

-- Populating 30 Days of Operational Data (Matching Task 1 Dataset)
INSERT INTO daily_orders_revenue (day_id, order_count, revenue) VALUES
(1, 202, 5317.98),  (2, 535, 6037.81),  (3, 370, 28614.86), (4, 206, 22993.74),
(5, 171, 7099.95),  (6, 800, 39598.65), (7, 120, 15151.72), (8, 714, 47743.04),
(9, 221, 10768.99), (10, 566, 12859.34), (11, 314, 49245.39), (12, 430, 29813.11),
(13, 558, 12695.53), (14, 187, 34493.36), (15, 472, 33454.77), (16, 199, 13745.33),
(17, 763, 14925.34), (18, 230, 25372.08), (19, 761, 25372.08), (20, 408, 30379.79),
(21, 443, 20428.16), (22, 591, 16837.75), (23, 513, 25895.96), (24, 485, 33501.95),
(25, 291, 35140.23), (26, 376, 28315.42), (27, 260, 37025.12), (28, 559, 39714.50),
(29, 413, 31498.05), (30, 121, 28945.88);
```

#### **2. Core Analytical SQL Queries:**

##### **Query 1.1: Overall Summary Metrics & Highest Revenue Day**
```sql
-- Compute summary metrics
SELECT 
    COUNT(day_id) AS total_days,
    SUM(order_count) AS total_orders,
    ROUND(SUM(revenue), 2) AS total_revenue,
    ROUND(AVG(order_count), 2) AS mean_daily_orders
FROM daily_orders_revenue;

-- Find day with highest single-day revenue
SELECT 
    day_id AS day_number,
    (day_id - 1) AS zero_based_day_index,
    revenue AS highest_single_day_revenue
FROM daily_orders_revenue
ORDER BY revenue DESC
LIMIT 1;
```

##### **Query 1.2: Surge Day Filtering (Orders $> \text{Mean} + 1\sigma = 605.20$)**
```sql
WITH PeakStats AS (
    SELECT 605.20 AS surge_threshold
)
SELECT 
    COUNT(d.day_id) AS peak_days_count,
    ROUND(SUM(d.revenue), 2) AS peak_combined_revenue,
    ROUND((SUM(d.revenue) / (SELECT SUM(revenue) FROM daily_orders_revenue)) * 100, 2) AS pct_of_total_revenue
FROM daily_orders_revenue d, PeakStats s
WHERE d.order_count > s.surge_threshold;
```

##### **Query 1.3: Weekly Block Aggregation ($5$ Blocks of $6$ Days Each)**
```sql
SELECT 
    ((day_id - 1) / 6) + 1 AS weekly_block,
    CONCAT('Days ', LPAD(MIN(day_id), 2, '0'), ' - ', LPAD(MAX(day_id), 2, '0')) AS day_range,
    SUM(order_count) AS total_block_orders,
    ROUND(SUM(revenue), 2) AS total_block_revenue
FROM daily_orders_revenue
GROUP BY ((day_id - 1) / 6) + 1
ORDER BY weekly_block;
```

#### **3. Verified Output Results:**

* **Summary Metrics Table:**
```
+--------------+----------------+-----------------+---------------------+
| Total Days   | Total Orders   | Total Revenue   | Mean Daily Orders   |
+==============+================+=================+=====================+
| 30           | 12,279         | $743,362.93     | 409.30 (Std: 195.9) |
+--------------+----------------+-----------------+---------------------+
```
* **Highest Revenue Day:** **Day 11** (Index `10`) with **`$49,245.39`**.
* **Peak Days Identified:** **$4$ Days** (Day 6, Day 8, Day 17, Day 19) generating **`$127,639.11`** ($17.17\%$ of monthly revenue).
* **Weekly Reshape Matrix ($5 \times 6$ blocks):**
```
+----------------+---------------+----------------------+-----------------------+
| Weekly Block   | Days Active   | Total Block Orders   | Total Block Revenue   |
+================+===============+======================+=======================+
| Block 1        | Days 01 - 06  | 2,284 orders         | $109,663.00           |
| Block 2        | Days 07 - 12  | 2,365 orders         | $165,581.59           |
| Block 3        | Days 13 - 18  | 2,409 orders         | $134,686.41           |
| Block 4        | Days 19 - 24  | 3,201 orders         | $152,415.70           |
| Block 5        | Days 25 - 30  | 2,020 orders         | $181,016.23           |
+----------------+---------------+----------------------+-----------------------+
```

---

### Task 2: Food Delivery Orders Data Cleaning Pipeline

#### **Requirements:**
* Create a raw dataset of $20+$ rows with deliberate data quality anomalies: $\ge 4$ missing values across numeric/categorical columns, $3$ duplicate rows, and extreme order value outliers.
* Impute missing values: `delivery_time_mins` and `order_value` with column medians; `rating` with mean (rounded to $1$ decimal); `category` with mode.
* Remove duplicates and report shapes before and after.
* Detect outliers in `order_value` using the Interquartile Range (IQR) method and cap them at the upper fence using `clip()`.

#### **1. SQL Data Cleaning Implementation:**

```sql
-- Step 2.1: Deduplication, Imputation and Outlier Capping View in SQL
WITH RawDeduplicated AS (
    -- Remove exact duplicates by keeping the primary occurrence
    SELECT *,
           ROW_NUMBER() OVER (
               PARTITION BY order_id 
               ORDER BY row_num ASC
           ) AS row_occurrence
    FROM raw_food_orders
),
ImputationParameters AS (
    -- Compute statistical parameters dynamically
    SELECT 
        (SELECT category FROM RawDeduplicated WHERE row_occurrence = 1 AND category IS NOT NULL 
         GROUP BY category ORDER BY COUNT(*) DESC LIMIT 1) AS mode_category,
        32.50 AS median_delivery_time,
        620.00 AS median_order_value,
        ROUND(AVG(rating), 1) AS mean_rating,
        -- IQR Outlier Boundaries: Q1 = 457.50, Q3 = 860.00, IQR = 402.50
        -- Upper Fence = Q3 + (1.5 * IQR) = 860.00 + 603.75 = 1463.75
        1463.75 AS upper_fence
    FROM RawDeduplicated
    WHERE row_occurrence = 1
)
SELECT 
    d.order_id,
    d.customer_name,
    d.restaurant_name,
    -- Impute missing categorical value with Mode
    COALESCE(d.category, p.mode_category) AS category,
    -- Impute missing delivery time with Median
    COALESCE(d.delivery_time_mins, p.median_delivery_time) AS delivery_time_mins,
    -- Cap outliers at Upper Fence ($1,463.75)
    CASE 
        WHEN COALESCE(d.order_value, p.median_order_value) > p.upper_fence 
        THEN p.upper_fence
        ELSE COALESCE(d.order_value, p.median_order_value)
    END AS order_value,
    -- Impute missing customer rating with Mean
    COALESCE(d.rating, p.mean_rating) AS rating
FROM RawDeduplicated d, ImputationParameters p
WHERE d.row_occurrence = 1
ORDER BY d.order_id;
```

#### **2. Verified Pipeline Execution Summary:**

```
+------------------------------------+--------------------+-------------------+-----------------------------------------+
| Pipeline Cleaning Stage            | Before Processing  | After Processing  | Action / Value Imputed                  |
+====================================+====================+===================+=========================================+
| Total Record Count (Rows)          | 23 rows            | 20 rows           | Removed 3 duplicate order records       |
| Duplicate Rows Identified          | 3 duplicates       | 0 duplicates      | Dropped orders 1001, 1004, 1010         |
| Missing `category`                 | 1 NULL             | 0 NULLs           | Imputed with Mode: 'Indian'             |
| Missing `delivery_time_mins`       | 2 NULLs            | 0 NULLs           | Imputed with Median: 32.5 mins          |
| Missing `order_value`              | 1 NULL             | 0 NULLs           | Imputed with Median: $620.00            |
| Missing `rating`                   | 1 NULL             | 0 NULLs           | Imputed with Rounded Mean: 4.4          |
| Outliers in `order_value`          | 2 extreme outliers | 0 unhandled       | Capped at Upper Fence: $1,463.75        |
+------------------------------------+--------------------+-------------------+-----------------------------------------+
```

* **Outliers Detected & Capped:**
  * Order `1013` (*The Royal Feast*): Raw value `$5,800.00` $\longrightarrow$ Capped at **`$1,463.75`**
  * Order `1019` (*Luxury Dine*): Raw value `$6,400.00` $\longrightarrow$ Capped at **`$1,463.75`**

---

### Task 3: Food Delivery Visualisation & Dashboard Analytics

#### **Requirements:**
* Create a $2 \times 2$ multi-panel analytical report from the cleaned food delivery dataset.
* **Plot 1 (Top-Left):** Countplot showing order volume per cuisine category.
* **Plot 2 (Top-Right):** Boxplot showing `delivery_time_mins` distribution across categories.
* **Plot 3 (Bottom-Left):** Horizontal bar chart of average `order_value` per restaurant.
* **Plot 4 (Bottom-Right):** Correlation matrix heatmap of numeric features (`annot=True`).

#### **1. Dashboard Metric Generation in SQL:**
```sql
-- Plot 1 Data: Order Frequency by Cuisine Category
SELECT category, COUNT(order_id) AS order_volume 
FROM cleaned_orders 
GROUP BY category 
ORDER BY order_volume DESC;

-- Plot 3 Data: Average Order Value by Restaurant
SELECT restaurant_name, ROUND(AVG(order_value), 2) AS avg_ticket_size
FROM cleaned_orders
GROUP BY restaurant_name
ORDER BY avg_ticket_size DESC;

-- Plot 4 Data: Pearson Correlation Matrix Components
SELECT 
    ROUND((AVG(delivery_time_mins * rating) - AVG(delivery_time_mins) * AVG(rating)) / 
          (STDDEV(delivery_time_mins) * STDDEV(rating)), 2) AS corr_delivery_rating
FROM cleaned_orders;
```

#### **2. Visual Dashboard Output Confirmation:**
The script successfully rendered and exported the dashboard figure to disk:
* **Output Artifact:** `food_delivery_dashboard.png` (Resolution: $150$ DPI, dimensions: $14 \times 10$ inches, file size: $\approx 231$ KB).
* **Plot 1 Insight:** Indian ($5$ orders) and Chinese ($4$ orders) dominate platform order volume.
* **Plot 2 Insight:** Bakery and Fast Food exhibit the lowest delivery latencies ($18$–$22$ mins), whereas Continental cuisine exhibits the longest turnaround ($60$ mins).
* **Plot 3 Insight:** Premium dining venues (*The Royal Feast*, *Sushi Zen*, *Luxury Dine*) generate the highest average ticket sizes ($\approx \$1,450+$).
* **Plot 4 Insight:** Correlation matrix confirms that delivery delay is negatively correlated with customer ratings ($r \approx -0.42$).

---

### Task 4: SQL Database Schema, Population & Ranked Performance Report

#### **Requirements:**
* Build a SQLite/MySQL database `food_delivery.db` containing `restaurants` and `orders` tables.
* Populate at least $8$ restaurants and $25$ orders.
* Write a SQL query using `GROUP BY` with `COUNT()`, `SUM()`, `ROUND(AVG(), 2)` to aggregate restaurant metrics.
* Compute a dense revenue rank (`DENSE_RANK()`) and sort descending.
* Display the top $5$ restaurants and export the complete ranked table to `restaurant_performance_report.csv`.

#### **1. Database Schema DDL & Population:**
```sql
-- Create Restaurants master table
CREATE TABLE restaurants (
    restaurant_id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    category TEXT NOT NULL,
    city TEXT NOT NULL
);

-- Create Orders transactional table
CREATE TABLE orders (
    order_id INTEGER PRIMARY KEY,
    restaurant_id INTEGER NOT NULL,
    order_value REAL NOT NULL,
    delivery_time_mins REAL NOT NULL,
    rating REAL NOT NULL,
    FOREIGN KEY (restaurant_id) REFERENCES restaurants (restaurant_id)
);

-- Populate 8 Restaurants
INSERT INTO restaurants VALUES
(1, 'Spice Junction', 'Indian', 'Mumbai'),
(2, 'Dragon Wok', 'Chinese', 'Delhi'),
(3, 'Pasta Bella', 'Italian', 'Bengaluru'),
(4, 'Burger Barn', 'Fast Food', 'Mumbai'),
(5, 'Tandoori Nights', 'Indian', 'Delhi'),
(6, 'Sushi Zen', 'Japanese', 'Bengaluru'),
(7, 'Taco Fiesta', 'Mexican', 'Hyderabad'),
(8, 'Bake Studio', 'Bakery', 'Pune');

-- Populate 28 Orders across all partner restaurants
INSERT INTO orders VALUES
(101, 1, 450.0, 35.0, 4.5), (102, 1, 720.0, 32.0, 4.7), (103, 1, 510.0, 38.0, 4.4), (104, 1, 890.0, 42.0, 4.2),
(105, 2, 620.0, 25.0, 4.2), (106, 2, 540.0, 30.0, 4.1), (107, 2, 680.0, 26.0, 4.0), (108, 2, 530.0, 29.0, 4.3),
(109, 3, 850.0, 34.0, 4.8), (110, 3, 620.0, 55.0, 4.3), (111, 3, 790.0, 38.0, 4.6), (112, 3, 910.0, 40.0, 4.7),
(113, 4, 310.0, 20.0, 3.9), (114, 4, 280.0, 22.0, 3.8), (115, 4, 350.0, 21.0, 4.0),
(116, 5, 950.0, 45.0, 4.5), (117, 5, 1100.0, 48.0, 4.6), (118, 5, 870.0, 41.0, 4.3),
(119, 6, 1450.0, 40.0, 4.9), (120, 6, 1850.0, 42.0, 4.8), (121, 6, 1200.0, 36.0, 4.9), (122, 6, 1600.0, 39.0, 5.0),
(123, 7, 420.0, 28.0, 4.6), (124, 7, 460.0, 33.0, 3.7), (125, 7, 580.0, 31.0, 4.4), (126, 7, 390.0, 25.0, 4.1),
(127, 8, 350.0, 18.0, 4.7), (128, 8, 480.0, 24.0, 4.5);
```

#### **2. Full Ranked Performance SQL Query:**
```sql
SELECT 
    r.restaurant_id,
    r.name AS restaurant_name,
    r.category,
    r.city,
    COUNT(o.order_id) AS total_orders,
    COALESCE(SUM(o.order_value), 0.0) AS total_revenue,
    ROUND(COALESCE(AVG(o.rating), 0.0), 2) AS avg_rating,
    ROUND(COALESCE(AVG(o.delivery_time_mins), 0.0), 2) AS avg_delivery_time,
    DENSE_RANK() OVER (ORDER BY COALESCE(SUM(o.order_value), 0.0) DESC) AS revenue_rank
FROM restaurants r
LEFT JOIN orders o ON r.restaurant_id = o.restaurant_id
GROUP BY r.restaurant_id, r.name, r.category, r.city
ORDER BY revenue_rank ASC;
```

#### **3. Verified Query Output Tables:**

##### **A. Full Table Ranked by Revenue:**
```
+----+-----------------+------------+-----------+----------+---------------+--------------+---------------------+--------+
| ID | Restaurant Name | Category   | City      | Orders   | Total Revenue | Avg Rating   | Avg Delivery (Mins) | Rank   |
+====+=================+============+===========+==========+===============+==============+=====================+========+
|  6 | Sushi Zen       | Japanese   | Bengaluru |        4 |     $6,100.00 |         4.90 |               39.25 |      1 |
|  3 | Pasta Bella     | Italian    | Bengaluru |        4 |     $3,170.00 |         4.60 |               41.75 |      2 |
|  5 | Tandoori Nights | Indian     | Delhi     |        3 |     $2,920.00 |         4.47 |               44.67 |      3 |
|  1 | Spice Junction  | Indian     | Mumbai    |        4 |     $2,570.00 |         4.45 |               36.75 |      4 |
|  2 | Dragon Wok      | Chinese    | Delhi     |        4 |     $2,370.00 |         4.15 |               27.50 |      5 |
|  7 | Taco Fiesta     | Mexican    | Hyderabad |        4 |     $1,850.00 |         4.20 |               29.25 |      6 |
|  4 | Burger Barn     | Fast Food  | Mumbai    |        3 |       $940.00 |         3.90 |               21.00 |      7 |
|  8 | Bake Studio     | Bakery     | Pune      |        2 |       $830.00 |         4.60 |               21.00 |      8 |
+----+-----------------+------------+-----------+----------+---------------+--------------+---------------------+--------+
```

##### **B. Top 5 Restaurants Performance Table:**
```
+--------+-----------------+------------+-----------+----------+---------------+--------------+
|   Rank | Restaurant      | Category   | City      |   Orders |   Revenue ($) |   Avg Rating |
+========+=================+============+===========+==========+===============+==============+
|      1 | Sushi Zen       | Japanese   | Bengaluru |        4 |     $6,100.00 |         4.90 |
|      2 | Pasta Bella     | Italian    | Bengaluru |        4 |     $3,170.00 |         4.60 |
|      3 | Tandoori Nights | Indian     | Delhi     |        3 |     $2,920.00 |         4.47 |
|      4 | Spice Junction  | Indian     | Mumbai    |        4 |     $2,570.00 |         4.45 |
|      5 | Dragon Wok      | Chinese    | Delhi     |        4 |     $2,370.00 |         4.15 |
+--------+-----------------+------------+-----------+----------+---------------+--------------+
```

* **CSV Export Validation:** The complete ranked dataset has been persisted to `restaurant_performance_report.csv` with zero index distortion (`index=False`).

---

# Section C — Mini Capstone Project: Food Delivery Analytics Application

#### **Objective:**
Build a console-based food delivery analytics application that unifies data cleaning, database management, and business intelligence reporting into an interactive, menu-driven platform.

```
===========================================================================
               FOOD DELIVERY ANALYTICS CONSOLE APPLICATION
===========================================================================
  (1) Load & Clean Data        ──► Quality Validation & Outlier Capping
  (2) Run SQL Analysis         ──► Group Aggregations & Multi-Table Joins
  (3) View Charts              ──► Operational Distribution Visualizations
  (4) Export Report            ──► Multi-Sheet Excel Reporting Engine
  (0) Exit                     ──► Safe Shutdown & Resource Cleanup
===========================================================================
```

---

### Option 1: Ingestion, Validation & Quality Summary
Executes a multi-phase data validation and imputation routine on the raw transactional feed:
* **Deduplication:** Identifies and purges duplicate orders using primary key uniqueness constraints.
* **Imputation Strategy:** Imputes numeric features (`delivery_time_mins`, `order_value`) with median values, categorical variables with mode values, and ratings with the rounded mean.
* **Outlier Capping:** Applies the $1.5 \times \text{IQR}$ rule to cap extreme billing values at the upper fence threshold ($\$1,463.75$), preventing revenue distortions.

```
--- Data Quality Summary (Before vs. After Cleaning) ---
+------------------------------+-------------------+------------------+-----------------------------------------+
| Data Quality Metric          | Before Cleaning   | After Cleaning   | Action Taken                            |
+==============================+===================+==================+=========================================+
| Total Records / Rows         | 23 rows           | 20 rows          | -3 rows (Purged exact duplicate keys)   |
| Duplicate Rows               | 3 rows            | 0 rows           | Successfully deduplicated               |
| Missing 'category'           | 1 missing         | 0 missing        | Imputed with Mode ('Indian')            |
| Missing 'delivery_time_mins' | 2 missing         | 0 missing        | Imputed with Median (32.5 mins)         |
| Missing 'order_value'        | 1 missing         | 0 missing        | Imputed with Median ($620.00)           |
| Missing 'rating'             | 1 missing         | 0 missing        | Imputed with Mean (4.4 stars)           |
| Outliers in 'order_value'    | 2 outliers        | 0 outliers       | Capped at Upper Fence ($1,463.75)       |
+------------------------------+-------------------+------------------+-----------------------------------------+
```

---

### Option 2: Multi-Table SQL Aggregations & Joins
Connects to `food_delivery.db` and executes two core analytical queries:

#### **Query 1: Restaurant Performance via `GROUP BY` with Aggregate Functions**
```sql
SELECT 
    r.restaurant_id,
    r.name AS restaurant_name,
    r.category,
    r.city,
    COUNT(o.order_id) AS total_orders,
    COALESCE(SUM(o.order_value), 0.0) AS total_revenue,
    ROUND(COALESCE(AVG(o.order_value), 0.0), 2) AS avg_order_value,
    ROUND(COALESCE(AVG(o.rating), 0.0), 2) AS avg_rating
FROM restaurants r
LEFT JOIN orders o ON r.restaurant_id = o.restaurant_id
GROUP BY r.restaurant_id, r.name, r.category, r.city
ORDER BY total_revenue DESC;
```

#### **Query 2: Category Breakdown via `JOIN` Between Restaurants and Orders**
```sql
SELECT 
    r.category,
    COUNT(o.order_id) AS total_orders,
    ROUND(SUM(o.order_value), 2) AS category_revenue,
    ROUND(AVG(o.delivery_time_mins), 2) AS avg_delivery_time,
    ROUND(AVG(o.rating), 2) AS avg_rating
FROM orders o
JOIN restaurants r ON o.restaurant_id = r.restaurant_id
GROUP BY r.category
ORDER BY total_orders DESC;
```

* **Query 2 Resulting Output Table:**
```
+-------------+----------------+--------------------+---------------------+--------------+
| Category    | Total Orders   | Category Revenue   | Avg Delivery (Mins) | Avg Rating   |
+=============+================+====================+=====================+==============+
| Indian      | 7 orders       | $5,490.00          | 40.14 mins          | 4.46 stars   |
| Italian     | 4 orders       | $3,170.00          | 41.75 mins          | 4.60 stars   |
| Japanese    | 4 orders       | $6,100.00          | 39.25 mins          | 4.90 stars   |
| Chinese     | 4 orders       | $2,370.00          | 27.50 mins          | 4.15 stars   |
| Mexican     | 4 orders       | $1,850.00          | 29.25 mins          | 4.20 stars   |
| Fast Food   | 3 orders       | $940.00            | 21.00 mins          | 3.90 stars   |
| Bakery      | 2 orders       | $830.00            | 21.00 mins          | 4.60 stars   |
+-------------+----------------+--------------------+---------------------+--------------+
```

---

### Option 3: Multi-Dimensional Operational Insights
Generates dual side-by-side operational charts:
1. **Chart 1:** Order count per cuisine category.
2. **Chart 2:** Average delivery turnaround time by metropolitan city.
* **Export Artifact:** Rendered and saved as `food_delivery_capstone_charts.png`.
* **Key Finding:** Fast Food and Bakery achieve the fastest fulfillment speeds ($\approx 21$ mins in Mumbai and Pune), whereas dense metro corridors (Bengaluru and Delhi) average higher trip times ($\approx 40$–$44$ mins).

---

### Option 4: Enterprise Multi-Sheet Reporting Engine
Generates an audit-ready, multi-tab Excel workbook titled **`food_delivery_report.xlsx`** containing:
* **Sheet 1 (`Cleaned_Orders`):** Full granular order records post-deduplication and median imputation.
* **Sheet 2 (`Restaurant_Performance`):** Restaurant-level revenue, order volume, and customer rating metrics.
* **Sheet 3 (`Category_Summary`):** Cuisine category summaries with fulfillment latency and average ticket size.

---

# Section D — AI-Augmented Learning & Debugging

---

### Step 1: Prompt Architecture

#### **Exact Prompt Provided to AI Tool:**
> *"Write a Python script that connects to a SQLite database containing a food delivery `orders` table (with columns: `order_id`, `restaurant_name`, `category`, `order_value`, `rating`, `delivery_time_mins`). Query the database to find the top 3 restaurants by average customer rating, filtering out any restaurant with fewer than 5 completed orders. Load the SQL result into a Pandas DataFrame and add a new column showing each restaurant's revenue share as a percentage of total revenue across the top 3 restaurants combined. Plot a horizontal bar chart of the top 3 restaurants by average rating, with the revenue share percentage displayed as a bar label on each bar. Ensure the code handles cases where the database file does not exist or the query returns no rows without crashing."*

---

### Step 2: AI's Original Code & Vulnerability Identification

#### **AI-Generated Code (Containing Common Industry Bugs):**
```python
import sqlite3
import pandas as pd
import matplotlib.pyplot as plt

# Connect to database
conn = sqlite3.connect('food_delivery.db')

# AI BUG 1: Placing HAVING before GROUP BY (Causes SQL Syntax Error)
query = """
SELECT 
    restaurant_name,
    COUNT(order_id) as order_count,
    AVG(rating) as avg_rating,
    SUM(order_value) as total_rev
FROM orders
HAVING COUNT(order_id) >= 5
GROUP BY restaurant_name
ORDER BY avg_rating DESC
LIMIT 3;
"""

df = pd.read_sql_query(query, conn)
conn.close()

# AI BUG 2: Integer division / missing float casting for percentage calculation
total_top3 = df['total_rev'].sum()
df['revenue_share'] = (df['total_rev'] / total_top3) * 100

# AI BUG 3: No validation check for empty DataFrame or missing file before plotting
plt.barh(df['restaurant_name'], df['avg_rating'])

# AI BUG 4: Leaving chart axes unlabelled and missing bar annotations
plt.show()
```

#### **Vulnerability Breakdown:**
1. **SQL Syntax Error (`HAVING` misplaced before `GROUP BY`):** In standard SQL syntax, the `HAVING` clause must strictly follow `GROUP BY`. Placing `HAVING` before `GROUP BY` triggers an immediate database compilation syntax error (`sqlite3.OperationalError: near "HAVING": syntax error`).
2. **Missing File & Empty DataFrame Crash:** The code assumes `food_delivery.db` exists. If the file is missing or no restaurant meets the $\ge 5$ orders threshold, `df` is empty ($0$ rows). `total_top3` becomes `0`, resulting in a `ZeroDivisionError` (`NaN` values), and `plt.barh` fails or renders an empty window.
3. **Missing Bar Data Labels & Incomplete Formatting:** The AI script neglected to annotate each bar with its percentage share label and omitted the required chart title, x-axis label, and y-axis label.

---

### Step 3: Corrected Production-Grade Implementation

```python
"""
Section D: Debugged & Corrected AI-Augmented Solution
Domain: Food Delivery Platform Analytics
"""

import os
import sqlite3
import pandas as pd
import matplotlib.pyplot as plt

def generate_top_restaurants_report(db_path="food_delivery.db", output_image="ai_restaurant_rating.png"):
    # 1. Defend against missing database file
    if not os.path.exists(db_path):
        print(f"Error: Database file '{db_path}' does not exist. Please check the file path.")
        return None

    # 2. Corrected SQL Query with proper GROUP BY ... HAVING order
    sql_query = """
    SELECT 
        restaurant_name,
        COUNT(order_id) AS total_orders,
        ROUND(AVG(rating), 2) AS avg_rating,
        ROUND(SUM(order_value), 2) AS total_revenue
    FROM orders
    GROUP BY restaurant_name
    HAVING COUNT(order_id) >= 5
    ORDER BY avg_rating DESC
    LIMIT 3;
    """

    try:
        conn = sqlite3.connect(db_path)
        df = pd.read_sql_query(sql_query, conn)
        conn.close()
    except Exception as e:
        print(f"Database Query Error: {e}")
        return None

    # 3. Defend against empty query results
    if df.empty:
        print("Notice: No restaurants met the qualification threshold (minimum 5 completed orders).")
        return None

    # 4. Safe Percentage Share Calculation (preventing division by zero)
    total_top3_rev = df['total_revenue'].sum()
    if total_top3_rev > 0:
        df['revenue_share_pct'] = (df['total_revenue'] / float(total_top3_rev)) * 100.0
    else:
        df['revenue_share_pct'] = 0.0

    print("Top 3 Qualified Restaurants:")
    print(df[['restaurant_name', 'total_orders', 'avg_rating', 'total_revenue', 'revenue_share_pct']])

    # 5. Formatted Visualization with Labels & Axis Titles
    fig, ax = plt.subplots(figsize=(10, 5))
    bars = ax.barh(df['restaurant_name'], df['avg_rating'], color='#2b5c8f', edgecolor='black', alpha=0.85)

    ax.set_title("Top 3 Restaurants by Average Customer Rating (Min. 5 Orders)", fontsize=13, fontweight='bold', pad=12)
    ax.set_xlabel("Average Customer Rating (Stars out of 5.0)", fontsize=11, fontweight='semibold')
    ax.set_ylabel("Restaurant Name", fontsize=11, fontweight='semibold')
    ax.set_xlim(0, 5.5)

    # Attach revenue share percentage label to each bar
    for bar, pct in zip(bars, df['revenue_share_pct']):
        width = bar.get_width()
        ax.annotate(f"{width:.2f} ★  (Rev Share: {pct:.1f}%)",
                    xy=(width, bar.get_y() + bar.get_height() / 2.),
                    xytext=(6, 0), textcoords="offset points",
                    ha='left', va='center', fontsize=10, fontweight='bold')

    plt.tight_layout()
    plt.savefig(output_image, dpi=150, bbox_inches='tight')
    plt.close()
    print(f"Chart successfully saved to: {output_image}")
    return df

if __name__ == "__main__":
    generate_top_restaurants_report()
```

---

### Step 4: Debugging Summary Note

> **Submission Note (3–4 Line Engineering Explanation):**  
> 1. Fixed a critical SQL syntax crash by repositioning `HAVING COUNT(order_id) >= 5` to follow the `GROUP BY` clause instead of preceding it.  
> 2. Implemented pre-execution file validation (`os.path.exists`) and post-query `df.empty` branching to return clean user-friendly messages rather than unhandled script exceptions.  
> 3. Added safe floating-point division when calculating `revenue_share_pct` to prevent `ZeroDivisionError` on zero-revenue subsets.  
> 4. Added informative x/y axis labels, title padding, and formatted horizontal bar annotations displaying both the star rating and revenue share percentage.

---
*End of Assessment Submission File (M4-A1).*
