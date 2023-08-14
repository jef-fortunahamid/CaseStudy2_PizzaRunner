# Case Study 2 Pizza Runner

Note: All information and data related to the case study were obtained from [here](https://8weeksqlchallenge.com/case-study-2/).
![Screenshot 2023-08-13 at 8 37 22 pm](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/7f31ec7a-b438-469f-973b-ebd6ffa00477)

## Business Task

## Entity Relationship Diagram

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

###Table: runner_orders
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
  -If the value is 'null' (as a string), it's converted to an actual NULL value.
  -If the value has a 'mins' suffix, that suffix is removed, and the remaining value is converted to an INT data type.
  -If the value has a 'minute' or 'minutes' suffix, that suffix is removed, and the remaining value is converted to an INT data type.
  -Otherwise, it's directly converted to an INT data type.
- If the cancellation column is NULL, an empty string, or has a value of 'null' (as a string), it's converted to an actual NULL value. Otherwise, the original value is retained.

![image](https://github.com/jef-fortunahamid/CaseStudy2_PizzaRunner/assets/125134025/c781fda5-43a1-479e-a1e8-ae0929e04046)


