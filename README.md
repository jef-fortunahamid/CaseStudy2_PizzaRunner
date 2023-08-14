# Case Study 2 Pizza Runner

*Note: All information and data related to the case study were obtained from [here](https://8weeksqlchallenge.com/case-study-2/).*
![Screenshot 2023-08-13 at 8 37 22 pm](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/7f31ec7a-b438-469f-973b-ebd6ffa00477)

## Business Task
Danny, inspired by an 80's Retro Styling ans Pizza trend on Instagram, decided to start a pizza delivery business, He name dit "Pizza Runner" and combined the pizza trend with an Uber-like delivery method. He hired "runners" for delivery and developed a mobile app for order placements.
With his background in data science, Danny knows the importance of data. He's created a database design for Pizza Runner but needs help cleaning the data and making calculations. This will help him manage his runners more efficiently and optimise the business operations.

## Entity Relationship Diagram
<img width="639" alt="image" src="https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/09d8efef-f7a4-4516-bd5e-73a0e6561044">

## Genaral Insights
- *Customer and Runner Experience:* The queries delved into understanding the efficiency and experience of runners. This included insights into the number of runners signing up each week, the average time taken for order pickups, and the relationship between the number of pizzas ordered and preparation time.
- *Ingredient Optimization:* Several questions aimed to understand the standard ingredients for each pizza, the most commonly added extras, and the most common exclusions. This provides insights into customer preferences and potential areas for menu optimization.
- *Pricing and Ratings:* This section focused on understanding the revenue generated by Pizza Runner. It considered fixed pizza prices, additional charges for extras, and delivery fees. Additionally, a new ratings system was proposed to capture customer feedback on their delivery experience.
- *Menu Expansion:* The bonus question explored the implications of adding a new pizza type to the menu, demonstrating how the current data design can accommodate such changes.

## Key SQL Syntax and Functions
- Temporary Tables (CREATE TEMP TABLE)
- Joins (LEFT JOIN, INNER JOIN, LEFT SEMI JOIN)
- Aggregation Functions (SUM, COUNT, AVG, MAX, MIN)
- Window Functions(RANK)
- Common Table Expressions (CTE)
- Conditional Logic (CASE WHEN)
- Date Functions (DATE_TRUNC, DATE_PART, AGE)
- String Functions (STRING_AGG, REGEXP_SPLIT_TO_TABLE)

## Data Cleaning and Transformation
### Table: customer_orders
If we are going to look closely at the table, there are some inconsistencies.
- There are missing/blanks and null values in both `exclusions` and `extras` columns
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/95ce827d-cf61-4041-b1fc-f2ac11e0e030)

We must clean up the `customer_orders` table. This cleaned-up table will be easier to work with for subsequent analyses and operations. We will proceed with the following:
- We need to create a temporary table with cleaned `exlcusions` and `extras` columns.
- We will replace the missing/blanks and 'null' values with NULL value.
```sql
DROP TABLE IF EXISTS customer_orders_temp;
CREATE TEMP TABLE customer_orders_temp AS 
SELECT
    order_id
  , customer_id
  , pizza_id
  , CASE 
      WHEN exclusions IN ('', 'null') THEN NULL
      ELSE exclusions
      END AS exclusions
  , CASE 
      WHEN extras IN ('', 'null') THEN NULL
      ELSE extras
      END AS extras
  , order_time
FROM pizza_runner.customer_orders;

SELECT *
FROM customer_orders_temp;
```
In this query, the `CASE WHEN` statement checks if the `exclusions` and `extras` values are blank ('') and 'null', and if so, replaces them with the `NULL` value.
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/40b6bbb6-aa14-4da9-9343-7b62d8f7798f)

### Table: runner_orders
Again, if we are going to look closely at this table, there are some issues.
- In the `pickup_time` column, we don't know if the null values correspond to the desired 'NULL' values.
- In the `distance` column, there are a few issues here. We have values where the number is directly followed by 'km' without a space. We also have instances where there's a space between the number and 'km'. Additionally, it's unclear if the null values are the desired 'NULL' values.
- In the `duration` column, quite a few issues here as well. We've got mins, minute and minutes, both with and without spaces. Additionally, it's uncertain whether the null values correspond to the desired 'NULL' values.
- In the `cancellation` column, we've got missing/blank values and different 'null' values.
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/b32fe23b-363a-4a4c-a41f-85f3b8684fce)

Just like from the previous table, we need to clean up the `runner_orders` table. Once tidied, it will be more manageable for subsequent analyses and tasks. 
```sql
DROP TABLE IF EXISTS runner_orders_temp;
CREATE TEMP TABLE runner_orders_temp AS
SELECT
    order_id
  , runner_id
  , CASE
      WHEN pickup_time LIKE 'null' THEN NULL
      ELSE pickup_time::TIMESTAMP
      END AS pickup_time
  , CASE 
      WHEN distance LIKE 'null' THEN NULL
      ELSE TRIM(REPLACE(distance, 'km', ''))::FLOAT
      END AS distance
  , CASE
      WHEN duration LIKE 'null' THEN NULL
      WHEN duration LIKE '%mins' THEN TRIM(REPLACE(duration, 'mins',''))::INT
      WHEN duration LIKE '%minute' THEN TRIM(REPLACE(duration, 'minute',''))::INT
      WHEN duration LIKE '%minutes' THEN TRIM(REPLACE(duration, 'minutes',''))::INT
      ELSE duration::INT
      END AS duration
  , CASE
      WHEN cancellation IS NULL OR cancellation LIKE 'null' OR cancellation LIKE '' THEN NULL
      ELSE cancellation
      END AS cancellation
FROM pizza_runner.runner_orders;

SELECT *
FROM runner_orders_temp;
```
This SQL statement is creating a cleaned-up version of the `runner_orders` table. It's making sure:
-  If the `pickup_time` column has a value of 'null' (as a string), it's converted to an actual NULL value. Otherwise, it's converted to a TIMESTAMP data type.
-  If the `distance` column has a value of 'null' (as a string), it's converted to an actual NULL value. If it has a 'km' suffix, that suffix is removed, and the remaining value is converted to a FLOAT data type.
- The `duration` column undergoes multiple transformations:
  - If the value is 'null' (as a string), it's converted to an actual NULL value.
  - If the value has a 'mins' suffix, that suffix is removed, and the remaining value is converted to an INT data type.
  - If the value has a 'minute' or 'minutes' suffix, that suffix is removed, and the remaining value is converted to an INT data type.
  - Otherwise, it's directly converted to an INT data type.
- If the cancellation column is NULL, an empty string, or has a value of 'null' (as a string), it's converted to an actual NULL value. Otherwise, the original value is retained.

![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/c781fda5-43a1-479e-a1e8-ae0929e04046)

### Part A: Pizza Metrics
> 1. How many pizzas were ordered?
```sql
SELECT 
  COUNT(*) AS pizza_count_order
FROM customer_orders_temp;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/109c9d11-f6be-4a7d-b441-a034432f5eff)

> 2. How many unique customer orders were made?
```sql
SELECT 
  COUNT(DISTINCT order_id) AS unique_customer_order
FROM customer_orders_temp;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/944cb3c6-c22d-43b8-b4b7-471b7325173c)

> 3. How many successful orders were delivered by each runner?
```sql
SELECT 
    runner_id
  , COUNT(DISTINCT order_id) AS successful_order_count
FROM runner_orders_temp
WHERE cancellation IS NULL
GROUP BY runner_id;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/25c0b488-8dfd-47c5-a2e9-79c3207e7788)

> 4. How many of each type of pizza was delivered?
```sql
SELECT 
    t2.pizza_name
  , COUNT(t1.pizza_id) AS delivered_pizza_count
FROM customer_orders_temp AS t1
LEFT JOIN pizza_runner.pizza_names AS t2
  ON t1.pizza_id = t2.pizza_id
WHERE EXISTS (
  SELECT 1
  FROM runner_orders_temp AS t3
  WHERE t1.order_id = t3.order_id
    AND cancellation IS NULL
)
GROUP BY t2.pizza_name
ORDER BY t2.pizza_name;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/13a8a75e-f417-471a-8549-5f60bf2788ef)

> 5. How many Vegetarian and Meatlovers were ordered by each customer?
```sql
SELECT
    customer_id
  , SUM(CASE
          WHEN pizza_id = 1 THEN 1
          ELSE 0
          END) AS meatlovers
  , SUM(CASE
          WHEN pizza_id = 2 THEN 1
          ELSE 0
          END) AS vegetarian
FROM customer_orders_temp AS t1 
GROUP BY
    t1.customer_id
ORDER BY
    t1.customer_id;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/7f113b00-8215-46bb-a154-3f6af563613f)

> 6. What was the maximum number of pizzas delivered in a single order?
```sql
WITH pizza_count AS (
  SELECT
      t1.order_id
    , COUNT(t1.pizza_id) AS pizza_count_per_order
    , RANK() OVER (ORDER BY COUNT(t1.pizza_id) DESC) AS count_rank
  FROM customer_orders_temp AS t1
  WHERE EXISTS (
      SELECT 1
      FROM runner_orders_temp AS t2
      WHERE t1.order_id = t2.order_id
        AND cancellation IS NULL
  )
  GROUP BY t1.order_id
)
SELECT
  pizza_count_per_order AS max_pizza_count
FROM pizza_count
WHERE count_rank = 1;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/1f1c1a05-bd5a-4c11-93e8-50b0004881a7)

> 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```sql 
SELECT
    t1.customer_id
  , SUM(CASE
            WHEN t1.exclusions IS NOT NULL OR t1.extras IS NOT NULL THEN 1 
            ELSE 0
            END) AS with_changes
  , SUM(CASE
            WHEN t1.exclusions IS NULL AND t1.extras IS NULL THEN 1
            ELSE 0
            END) AS no_changes
FROM customer_orders_temp AS t1
WHERE EXISTS (
      SELECT 1
      FROM runner_orders_temp AS t2
      WHERE t1.order_id = t2.order_id
        AND cancellation IS NULL
)
GROUP BY t1.customer_id
ORDER BY t1.customer_id;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/90639ed2-b7a3-4bcd-92e9-9a061ca1539e)

> 8. How many pizzas were delivered that had both exclusions and extras?
```sql
SELECT
  COUNT(t1.pizza_id) AS pizza_with_exclusions_and_extras
FROM customer_orders_temp AS t1
INNER JOIN runner_orders_temp AS t2
  ON t1.order_id = t2.order_id
        AND cancellation IS NULL
WHERE t1.exclusions IS NOT NULL AND t1.extras IS NOT NULL;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/37752362-ad91-4662-97ed-5fcff24cd218)

> 9. What was the total volume of pizzas ordered for each hour of the day?
```sql
SELECT
    DATE_PART('hour', order_time) AS hour_of_day
  , COUNT(*) AS pizza_count
FROM customer_orders_temp
GROUP BY hour_of_day
ORDER BY hour_of_day;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/fd31f825-da92-4f6a-a998-dc75d13f6f79)

> 10. What was the volume of orders for each day of the week?
```sql
SELECT
    TO_CHAR(order_time, 'Day') AS day_of_week
  , COUNT(*) AS pizza_count
FROM customer_orders_temp
GROUP BY day_of_week, DATE_PART('DOW', order_time)
ORDER BY DATE_PART('DOW', order_time);
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/f7a67337-315f-44a6-8893-061c88339145)

### Part B: Runner and Customer Experience
> 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
```sql
SELECT
    DATE_TRUNC('WEEK', registration_date) AS registration_week
  , COUNT(*) AS runners_count
FROM pizza_runner.runners
GROUP BY registration_week
ORDER BY registration_week;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/93046375-88a8-429c-8d56-7bba2601aab5)

> 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
```sql
WITH pickup_minutes_cte AS (
  SELECT DISTINCT
      t1.order_id
    , DATE_PART('minutes', AGE(t1.pickup_time, t2.order_time))::INT AS pickup_minutes
  FROM runner_orders_temp AS t1 
  INNER JOIN customer_orders_temp AS t2 
    ON t1.order_id = t2.order_id
  WHERE t1.pickup_time IS NOT NULL
)
SELECT
  ROUND(AVG(pickup_minutes), 3) AS average_pikcup_minutes
FROM pickup_minutes_cte;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/3d77176c-d217-4399-ad0e-d77a347534f5)

> 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
```sql
SELECT DISTINCT
    t1.order_id
  , DATE_PART('minutes', AGE(t1.pickup_time, t2.order_time))::INT AS pickup_minutes
  , COUNT(pizza_id) AS pizza_count
FROM runner_orders_temp AS t1 
INNER JOIN customer_orders_temp AS t2 
  ON t1.order_id = t2.order_id
WHERE t1.pickup_time IS NOT NULL
GROUP BY t1.order_id, pickup_minutes
ORDER BY pizza_count;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/2eda9dd8-63e5-4a4c-bd94-d98176190ee9)

> 4. What was the average distance travelled for each customer?
```sql
WITH customer_distance AS (
  SELECT DISTINCT
      t1.customer_id
    , t1.order_id
    , t2.distance
  FROM customer_orders_temp AS t1
  INNER JOIN runner_orders_temp AS t2
    ON t1.order_id = t2.order_id
  WHERE t2.pickup_time IS NOT NULL
)
SELECT
    customer_id
  , ROUND(AVG(distance)::NUMERIC, 1) AS average_distance
FROM customer_distance
GROUP BY customer_id
ORDER BY customer_id;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/700355b5-f60e-4243-adc5-81bde6f4db75)

> 5. What was the difference between the longest and shortest delivery times for all orders?
```sql
WITH customer_distance AS (
  SELECT DISTINCT
      t1.customer_id
    , t1.order_id
    , t2.duration
  FROM customer_orders_temp AS t1
  INNER JOIN runner_orders_temp AS t2
    ON t1.order_id = t2.order_id
  WHERE t2.pickup_time IS NOT NULL
)
SELECT
    MAX(duration) - MIN(duration) AS max_difference
FROM customer_distance;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/dfeedfff-7900-42b7-bb22-7c820c963bd9)

> 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
```sql
SELECT
    runner_id
  , order_id
  , EXTRACT(HOUR FROM pickup_time) AS hour_of_day
  , distance AS distance_km
  , duration AS duration_minutes
  , ROUND(distance::NUMERIC / (duration::NUMERIC / 60), 1) AS speed_km_hr
FROM runner_orders_temp
WHERE cancellation IS NULL;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/e2443468-145e-4ddf-b3a8-f8147bd79d76)

> 7. What is the successful delivery percentage for each runner?
```sql
SELECT
    runner_id
  , ROUND(
        100 * SUM(CASE WHEN pickup_time IS NOT NULL THEN 1 ELSE 0 END)::NUMERIC / 
        COUNT(*)
        ) AS success_percentage
FROM runner_orders_temp
GROUP BY runner_id
ORDER BY runner_id;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/1e7338dc-3416-406c-bbcd-63ebfba1bced)

### Part C: Ingredient Optimisation
> 1. What are the standard ingredients for each pizza?
```sql
WITH split_pizza_name AS(
  SELECT
      pizza_id
    , REGEXP_SPLIT_TO_TABLE(toppings, '[,\s]+')::INT AS topping_id
  FROM pizza_runner.pizza_recipes
)
SELECT
    pizza_id
  , STRING_AGG(t2.topping_name, ', ') AS standard_ingredients
FROM split_pizza_name AS t1
INNER JOIN pizza_runner.pizza_toppings AS t2 
  ON t1.topping_id = t2.topping_id
GROUP BY pizza_id
ORDER BY pizza_id;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/a91ba87d-0d15-4f8d-8c3c-8148036e09c6)

> 2. What was the most commonly added extra?
```sql
WITH extras_topping AS (
  SELECT
      pizza_id
    , REGEXP_SPLIT_TO_TABLE(extras, '[,\s]+')::INT AS topping_id
  FROM customer_orders_temp
  WHERE extras IS NOT NULL
)
SELECT
    t2.topping_name
  , COUNT(t1.topping_id) AS extras_topping_count
FROM extras_topping AS t1 
INNER JOIN pizza_runner.pizza_toppings AS t2
  ON t1.topping_id = t2.topping_id
GROUP BY t2.topping_name
ORDER BY extras_topping_count DESC;
```
![Screenshot 2023-08-14 at 10 34 35 pm](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/aab4ff3b-8d08-431a-bd2d-b715b3a54135)

> 3. What was the most common exclusion?
```sql
WITH exclusions_topping AS (
  SELECT
      pizza_id
    , REGEXP_SPLIT_TO_TABLE(exclusions, '[,\s]+')::INT AS topping_id
  FROM customer_orders_temp
  WHERE exclusions IS NOT NULL
)
SELECT
    t2.topping_name
  , COUNT(t1.topping_id) AS exclusions_topping_count
FROM exclusions_topping AS t1 
INNER JOIN pizza_runner.pizza_toppings AS t2
  ON t1.topping_id = t2.topping_id
GROUP BY t2.topping_name
ORDER BY exclusions_topping_count DESC;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/69e31f2c-9061-4de4-a0c7-8b7f5d343456)

> 4. Generate an order item for each record in the customers_orders table in the format of one of the following: + Meat Lovers + Meat Lovers - Exclude Beef + Meat Lovers - Extra Bacon + Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers
```sql
WITH customer_orders_row_numbered AS (
  SELECT
      order_id
    , customer_id
    , pizza_id
    , exclusions
    , extras
    , order_time
    , ROW_NUMBER() OVER() AS original_row_number
  FROM customer_orders_temp
),
extras_exclusions AS (
    SELECT
        order_id
      , customer_id
      , pizza_id
      , REGEXP_SPLIT_TO_TABLE(exclusions, '[,\s]+')::INT AS exclusions_topping_id
      , REGEXP_SPLIT_TO_TABLE(extras, '[,\s]+')::INT AS extras_topping_id
      , order_time
      , original_row_number
    FROM customer_orders_row_numbered
  UNION
    SELECT
        order_id
      , customer_id
      , pizza_id
      , NULL AS exclusions_topping_id
      , NULL AS extras_topping_id
      , order_time
      , original_row_number
    FROM customer_orders_row_numbered
    WHERE exclusions IS NULL AND extras IS NULL
),
complete_dataset AS (
SELECT
      base.order_id
    , base.customer_id
    , base.pizza_id
    , names.pizza_name
    , base.order_time
    , base.original_row_number
    , STRING_AGG(exclusions.topping_name, ', ') AS exclusions
    , STRING_AGG(extras.topping_name, ', ') AS extras
  FROM extras_exclusions AS base
  
  INNER JOIN pizza_runner.pizza_names AS names
    ON base.pizza_id = names.pizza_id
    
  LEFT JOIN pizza_runner.pizza_toppings AS exclusions
    ON base.exclusions_topping_id = exclusions.topping_id
    
  LEFT JOIN pizza_runner.pizza_toppings AS extras
    ON base.extras_topping_id = extras.topping_id
  GROUP BY
      base.order_id
    , base.customer_id
    , base.pizza_id
    , names.pizza_name
    , base.order_time
    , base.original_row_number
),
parsed_string_outputs AS (
  SELECT
      order_id
    , customer_id
    , pizza_id
    , order_time
    , original_row_number
    , pizza_name
    , CASE
        WHEN exclusions IS NULL THEN ''
        ELSE ' - Exclude ' || exclusions 
        END AS exclusions
    , CASE
        WHEN extras IS NULL THEN ''
        ELSE ' - Extra ' || extras
        END AS extras
  FROM complete_dataset
),
final_output AS (
  SELECT
      order_id
    , customer_id
    , pizza_id
    , order_time
    , pizza_name || exclusions || extras AS order_item
		, original_row_number
  FROM parsed_string_outputs
)
SELECT
    order_id
  , customer_id
  , pizza_id
  , order_time
  , order_item
FROM final_output
ORDER BY original_row_number;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/f040d8af-690a-45f9-9bcc-088035a46385)

> 5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients + For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"
```sql
WITH customer_orders_row_numbered AS (
  SELECT
      order_id
    , customer_id
    , pizza_id
    , exclusions
    , extras
    , order_time
    , ROW_NUMBER() OVER() AS original_row_number
  FROM customer_orders_temp
),
regular_toppings AS (
  SELECT
      pizza_id
    , REGEXP_SPLIT_TO_TABLE(toppings, '[,\s]+')::INT AS topping_id
  FROM pizza_runner.pizza_recipes
),
base_toppings AS (
  SELECT
      t1.order_id
    , t1.customer_id
    , t1.pizza_id
    , t1.order_time
    , t1.original_row_number
    , t2.topping_id
  FROM customer_orders_row_numbered AS t1 
  LEFT JOIN regular_toppings AS t2 
    ON t1.pizza_id = t2.pizza_id
),
with_exclusions AS (
  SELECT
      order_id
    , customer_id
    , pizza_id
    , order_time
    , original_row_number
    , REGEXP_SPLIT_TO_TABLE(exclusions, '[,\s]+')::INT AS topping_id
  FROM customer_orders_row_numbered
  WHERE exclusions IS NOT NULL
),
with_extras AS (
  SELECT
      order_id
    , customer_id
    , pizza_id
    , order_time
    , original_row_number
    , REGEXP_SPLIT_TO_TABLE(extras, '[,\s]+')::INT AS topping_id
  FROM customer_orders_row_numbered
  WHERE extras IS NOT NULL
),
combined_orders AS (
  SELECT * FROM base_toppings
  EXCEPT
  SELECT * FROM with_exclusions
  UNION ALL 
  SELECT * FROM with_extras
),
joined_toppings AS (
  SELECT
      t1.order_id
    , t1.customer_id
    , t1.pizza_id
    , t1.order_time
    , t1.original_row_number
    , t1.topping_id
    , t2.pizza_name
    , t3.topping_name
    , COUNT(t1.*) AS topping_count
  FROM combined_orders AS t1 
  INNER JOIN pizza_runner.pizza_names AS t2 
    ON t1.pizza_id = t2.pizza_id
  INNER JOIN pizza_runner.pizza_toppings AS t3 
    ON t1.topping_id = t3.topping_id
  GROUP BY
      t1.order_id
    , t1.customer_id
    , t1.pizza_id
    , t1.order_time
    , t1.original_row_number
    , t1.topping_id
    , t2.pizza_name
    , t3.topping_name
)
SELECT
    order_id
  , customer_id
  , pizza_id
  , order_time
  , original_row_number
  , pizza_name || ': ' || STRING_AGG(
            CASE 
              WHEN topping_count > 1 THEN topping_count || 'x ' || topping_name
              ELSE topping_name
              END,
            ', '
          ) AS ingredients_list
FROM joined_toppings
GROUP BY
    order_id
  , customer_id
  , pizza_id
  , order_time
  , original_row_number
  , pizza_name;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/0e9a21ba-34ab-4f79-976d-1d9d18e1c34c)

> 6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?
```sql
WITH customer_orders_row_numbered AS (
  SELECT
      order_id
    , customer_id
    , pizza_id
    , exclusions
    , extras
    , order_time
    , ROW_NUMBER() OVER() AS original_row_number
  FROM customer_orders_temp
),
regular_toppings AS (
  SELECT
      pizza_id
    , REGEXP_SPLIT_TO_TABLE(toppings, '[,\s]+')::INT AS topping_id
  FROM pizza_runner.pizza_recipes
),
base_toppings AS (
  SELECT
      t1.order_id
    , t1.customer_id
    , t1.pizza_id
    , t1.order_time
    , t1.original_row_number
    , t2.topping_id
  FROM customer_orders_row_numbered AS t1 
  LEFT JOIN regular_toppings AS t2 
    ON t1.pizza_id = t2.pizza_id
),
with_exclusions AS (
  SELECT
      order_id
    , customer_id
    , pizza_id
    , order_time
    , original_row_number
    , REGEXP_SPLIT_TO_TABLE(exclusions, '[,\s]+')::INT AS topping_id
  FROM customer_orders_row_numbered
  WHERE exclusions IS NOT NULL
),
with_extras AS (
  SELECT
      order_id
    , customer_id
    , pizza_id
    , order_time
    , original_row_number
    , REGEXP_SPLIT_TO_TABLE(extras, '[,\s]+')::INT AS topping_id
  FROM customer_orders_row_numbered
  WHERE extras IS NOT NULL
),
combined_orders AS (
  SELECT * FROM base_toppings
EXCEPT
  SELECT * FROM with_exclusions
UNION ALL 
  SELECT * FROM with_extras
)
SELECT
    t2.topping_name
  , COUNT(*) AS topping_count
FROM combined_orders AS t1 
INNER JOIN pizza_runner.pizza_toppings AS t2 
  ON t1.topping_id = t2.topping_id
INNER JOIN runner_orders_temp AS t3 
  ON t1.order_id = t3.order_id
    AND cancellation IS NULL
GROUP BY topping_name
ORDER BY topping_count DESC;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/8e542807-37ca-498e-8e06-30f7c7800ec4)

### Part D: Pricing and Ratings
> 1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
```sql
SELECT
  SUM(CASE
        WHEN pizza_id = 1 THEN 12
        WHEN pizza_id = 2 THEN 10
        ELSE 0
        END) AS revenue
FROM customer_orders_temp AS t1 
INNER JOIN runner_orders_temp AS t2 
  ON t1.order_id = t2.order_id
    AND cancellation IS NULL;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/848c73f3-0019-4f7f-8c69-8a5ab3d21967)

> 2. What if there was an additional $1 charge for any pizza extras? + Add cheese is $1 extra
```sql
WITH customer_runner_joined AS (
  SELECT
      order_id
    , customer_id
    , pizza_id
    , exclusions
    , extras
    , order_time
    , ROW_NUMBER() OVER() AS original_row_number
  FROM customer_orders_temp
  WHERE EXISTS (
      SELECT 1
      FROM runner_orders_temp
      WHERE customer_orders_temp.order_id = runner_orders_temp.order_id
        AND runner_orders_temp.cancellation IS NULL
  )
)
SELECT
  SUM(
    CASE
      WHEN pizza_id = 1 THEN 12
      WHEN pizza_id = 2 THEN 10
      END +
    COALESCE(
        CARDINALITY(REGEXP_SPLIT_TO_ARRAY(extras, '[,\s]+')),
        0
    )
  ) AS cost
FROM customer_runner_joined;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/9e96409a-a746-4ff8-ab85-d34235a0fd59)

> 3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
```sql
DROP TABLE IF EXISTS pizza_runner.ratings;
CREATE TABLE pizza_runner.ratings (
    rating_id   SERIAL  PRIMARY KEY -- Unique identifier for each rating
  , order_id    INT4    -- The order that the rating is associated with
  , rating      INT4    NOT NULL CHECK(rating>=1 AND rating<=5) -- The rating, which must be between 1 and 5
  , comments    TEXT  -- Any additional comments that the customer left about their experience
);
  
INSERT INTO pizza_runner.ratings (order_id, rating, comments)
SELECT 
    order_id
  , CASE 
      WHEN order_id = 1 THEN 5
      WHEN order_id = 2 THEN 4
      WHEN order_id = 3 THEN 5
      WHEN order_id = 4 THEN 3
      WHEN order_id = 5 THEN 4
      WHEN order_id = 6 THEN 5
      WHEN order_id = 7 THEN 2
      WHEN order_id = 8 THEN 4
      WHEN order_id = 9 THEN 3
      WHEN order_id = 10 THEN 5
    END AS rating
  , CASE 
      WHEN order_id = 1 THEN 'Great service! The pizza was delicious and the delivery was fast.'
      WHEN order_id = 2 THEN 'Good pizza, but the delivery took a bit longer than expected.'
      WHEN order_id = 3 THEN NULL
      WHEN order_id = 4 THEN 'Excellent service! The pizza was hot and the delivery was on time.'
      WHEN order_id = 5 THEN 'Good service overall. The pizza was tasty and the delivery was quick.'
      WHEN order_id = 6 THEN NULL
      WHEN order_id = 7 THEN 'The pizza was not as good as usual, and the delivery was late.'
      WHEN order_id = 8 THEN NULL
      WHEN order_id = 9 THEN NULL
      WHEN order_id = 10 THEN 'Excellent! The pizza was delicious and the delivery was fast.'
    END AS comments
FROM runner_orders_temp
WHERE pickup_time IS NOT NULL;

SELECT * FROM pizza_runner.ratings ORDER BY order_id;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/3f9c7e2f-e30b-4e27-b25f-f6eadfa77bc1)

> 4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries? + customer_id + order_id + runner_id + rating + order_time + pickup_time + Time between order and pickup + Delivery duration + Average speed + Total number of pizzas
```sql
SELECT
    t1.customer_id
  , t1.order_id
  , t2.runner_id
  , t3.rating
  , t1.order_time
  , t2.pickup_time
  , EXTRACT(MINUTE FROM (t2.pickup_time - t1.order_time)) AS pickup_minutes
  , t2.duration
  , ROUND(t2.distance::NUMERIC / (t2.duration::NUMERIC / 60), 1) AS avg_speed
  , COUNT(t1.pizza_id) AS pizza_count
FROM customer_orders_temp AS t1 

INNER JOIN runner_orders_temp AS t2 
  ON  t1.order_id = t2.order_id

INNER JOIN pizza_runner.ratings AS t3 
  ON t1.order_id = t3.order_id
  
GROUP BY
    t1.customer_id
  , t1.order_id
  , t2.runner_id
  , t3.rating
  , t1.order_time
  , t2.pickup_time
  , pickup_minutes
  , t2.duration
  , avg_speed
ORDER BY order_id;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/d694d673-3b60-41eb-8e54-13cfc791f66e)

> 5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
```sql
WITH cost_per_order AS (
  SELECT
      t1.order_id
    , SUM(CASE 
        WHEN t2.pizza_id = 1 THEN 12
        WHEN t2.pizza_id = 2 THEN 10
        END) AS pizza_cost
    , t1.distance * 0.3 AS delivery_cost
  FROM runner_orders_temp AS t1 
  INNER JOIN customer_orders_temp AS t2 
    ON t1.order_id = t2.order_id
  WHERE pickup_time IS NOT NULL
  GROUP BY 
      t1.order_id
    , t1.distance
)
SELECT
  ROUND(SUM(pizza_cost - delivery_cost)::NUMERIC, 2) AS total_revenue
FROM cost_per_order;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/6218f080-c020-4b53-bac1-99355b018fe7)

### Part E: Bonus Question
> If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?
```sql
DROP TABLE IF EXISTS pizza_names_temp;
CREATE TEMP TABLE pizza_names_temp AS
  SELECT * 
  FROM pizza_runner.pizza_names;

INSERT INTO pizza_names_temp (pizza_id, pizza_name)
VALUES
  (3, 'Supreme');

SELECT * FROM pizza_names_temp;

DROP TABLE IF EXISTS pizza_recipes_temp;
CREATE TEMP TABLE pizza_recipes_temp AS
  SELECT * 
  FROM pizza_runner.pizza_recipes;

INSERT INTO pizza_recipes_temp (pizza_id, toppings)
SELECT
  3,
  STRING_AGG(topping_id::TEXT, ', ')
FROM pizza_runner.pizza_toppings;

SELECT * FROM pizza_recipes_temp;
```
![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/e349acb8-8805-4bbc-a1da-9c70a7118b33)
