# Task-PIT-ROUND-4
This repository contains an analysis and implementation of Point-in-Time (PIT) joins to prevent data leakage in machine learning models. It includes a detailed examination of the MULTIPLE_FEATURE_VIEW_POINT_IN_TIME_JOIN function from Feast, covering edge cases, usage, and a custom Python implementation of PIT joins without SQL.

## TASK 
Understand the concept of Point in time Joins here [Point-in-time joins | Feast](https://docs.feast.dev/getting-started/concepts/point-in-time-joins) 

Point in time joins
are very important while creating training data for ML models and are critical to ensuring that
there is no data/feature leakage.

## Problem Statement
Here is an open source implementation of a Point in Time join [query in Feast](https://github.com/feast-dev/feast/blob/b0dc6832ff446429390a916aa9e0e61066cbde1d/sdk/python/feast/infra/offline_stores/redshift.py#L633)

##### 1. Understand this query and what edge cases it considers

##### 2. Figure out where and how MULTIPLE_FEATURE_VIEW_POINT_IN_TIME_JOIN is being called/referenced throughout this [repo](https://github.com/feast-dev/feast).

##### 3. Implement the PIT join yourself in Python (no usage of SQL).


--------------------------------------------------------------------------------------------------------------------------------------------------

## TASK 01 : Understand this query and what edge cases it considers


#### Understanding Point-in-Time Joins
Point-in-time joins are a technique used to combine data from different sources in a way that ensures no "future" information is leaked into the past when creating training datasets for machine learning models. This is crucial because in real-world applications, we won't have access to future data at the time a prediction is made.

Feast (Feature Store) is an open-source feature store for machine learning, and the query we are examining is part of Feast’s implementation to support point-in-time joins.

#### Analyzing the Query
Here’s a breakdown of the key aspects of the query linked:

##### General Purpose:
The query is designed to perform a PIT join between the entity data (which contains timestamps) and the feature data stored in Redshift. It ensures that only the feature data available before or at the timestamp of the entity data is considered.

##### Handling Edge Cases:
- Time Lag Consideration: The query likely includes logic to ensure that features are not joined from a future time relative to the entity's timestamp.

- Duplicated or Missing Timestamps: The query may consider scenarios where there are multiple records with the same timestamp or missing timestamps and handle them appropriately to avoid incorrect feature assignment.

- Data Overlap: Ensures that if there are multiple potential matches (e.g., if a feature exists at several points in time close to the entity timestamp), it correctly selects the most relevant one based on some logic (like closest time before the entity timestamp).

- Data Consistency: The query needs to ensure consistency in the case of data being updated or backfilled after the fact. It prevents leaking this backfilled data into past predictions by only joining data that was genuinely available at the time.

##### Key Components of the Query:
- Partitioning and Windowing: The query used SQL window functions to correctly partition data by entity and order it by timestamp, ensuring that only the most relevant past data is joined.

- Filters and Conditions: These help in eliminating rows that might cause data leakage, such as filtering out feature rows that occur after the entity timestamp.

- Edge Case Handling: The query include specific conditions to deal with missing data or ensure that in case of ties (multiple feature rows with the same timestamp), a consistent method of selection is used.

##### Observation
Understanding this query involves recognizing how it manages data in time-sensitive contexts, ensuring that the training data remains clean and unbiased by future information. This is crucial for maintaining the integrity and effectiveness of machine learning models, as it helps prevent data leakage that could artificially inflate model performance during training.



--------------------------------------------------------------------------------------------------------------------------------------------------

## Task 2. Figure out where and how MULTIPLE_FEATURE_VIEW_POINT_IN_TIME_JOIN is being called/referenced throughout this [repo](https://github.com/feast-dev/feast).

### Answer for WHERE? 
—> MULTIPLE_FEATURE_VIEW_POINT_IN_TIME_JOIN is called/referenced in 7 files:

a) [bigquery.py](https://github.com/feast-dev/feast/blob/419ca5e9523ff38f27141b79ae12ebb0646c6617/sdk/python/feast/infra/offline_stores/bigquery.py#L293)
b) [postgres.py](https://github.com/feast-dev/feast/blob/419ca5e9523ff38f27141b79ae12ebb0646c6617/sdk/python/feast/infra/offline_stores/contrib/postgres_offline_store/postgres.py#L432)
c) [spark.py](https://github.com/feast-dev/feast/blob/419ca5e9523ff38f27141b79ae12ebb0646c6617/sdk/python/feast/infra/offline_stores/contrib/spark_offline_store/spark.py#L571)
d) [trino.py](https://github.com/feast-dev/feast/blob/419ca5e9523ff38f27141b79ae12ebb0646c6617/sdk/python/feast/infra/offline_stores/contrib/trino_offline_store/trino.py#L527)
e) [redshift.py](https://github.com/feast-dev/feast/blob/419ca5e9523ff38f27141b79ae12ebb0646c6617/sdk/python/feast/infra/offline_stores/redshift.py#L633)
f) [snowflake.py](https://github.com/feast-dev/feast/blob/419ca5e9523ff38f27141b79ae12ebb0646c6617/sdk/python/feast/infra/offline_stores/snowflake.py#L712)
g) [athena.py](https://github.com/feast-dev/feast/blob/419ca5e9523ff38f27141b79ae12ebb0646c6617/sdk/python/feast/infra/offline_stores/contrib/athena_offline_store/athena.py#L540)

### Answer for HOW?

SCREENSHOT


##### For line 293:
The MULTIPLE_FEATURE_VIEW_POINT_IN_TIME_JOIN is referenced in the bigquery.py fileand  is used as a SQL query template within the get_historical_features method of the BigQueryOfflineStore class. Here's a breakdown of how it fits into the process:

1. Template Usage:
  - The MULTIPLE_FEATURE_VIEW_POINT_IN_TIME_JOIN template is passed as an argument to the build_point_in_time_query function, which is imported from the offline_utils.py file. This function uses the template to generate the final SQL query that performs a point-in-time join between the feature view tables and the entity dataframe in BigQuery.

2. Query Generation:
  - Inside the get_historical_features method, a query_generator function is defined that:

    - Uploads the entity dataframe (entity_df) to BigQuery.

    - Validates the columns in the entity dataframe.

    - Builds a query context using get_feature_view_query_context.

    - Uses this context along with the MULTIPLE_FEATURE_VIEW_POINT_IN_TIME_JOIN template to 
    generate a SQL query.

3. RetrievalJob:
- The generated SQL query is then used to create a BigQueryRetrievalJob, which handles 
    executing the query and retrieving the historical feature data.

### Key Points:
- The MULTIPLE_FEATURE_VIEW_POINT_IN_TIME_JOIN template is crucial in creating the SQL query 
  needed to join feature views with the entity dataframe based on timestamps.
- The query generation process is abstracted within the query_generator function, which is a 
  context manager ensuring that the temporary BigQuery table (used to store the uploaded entity 
  dataframe) is cleaned up after use.

#### For line 825:

#### Create Entity Dataframe:
Constructs a temporary table from left_table_query_string with entity identifiers and timestamps.

#### Process Each Feature View:
- Filter Data: Retrieves feature data where timestamps are ≤ entity timestamps.

- Handle TTL: Applies Time-to-Live (TTL) filter if defined, keeping data within a time range.

- Deduplicate: If a created_timestamp_column is present, deduplicates by keeping the most recent entry for each timestamp.

- Select Latest Features: Chooses the latest feature values based on event timestamps and optionally created timestamps.

#### Join Feature Views to Entity Dataframe:
- Joins cleaned feature data with the entity dataframe using unique identifiers.

#### Final Selection:
- Produces a consolidated table with entity data and the relevant feature data from all feature views.

#### Note:There is some difference in all files. These differences ensure that each template is optimized for the respective data warehouse environment while maintaining similar overall logic for performing point-in-time joins.

How MULTIPLE_FEATURE_VIEW_POINT_IN_TIME_JOIN is called in all files:

→Constructs a temporary entity_dataframe view with unique IDs.

→Performs point-in-time joins on feature views, filtering by event timestamp and TTL.

→Deduplicates data if needed, selects the latest features, and assembles the final output query.

#### Difference I found in bigquery.py file and redshift.py file :

#### 1. SQL Syntax Differences:
- Data Type:
  - Redshift Version: Uses VARCHAR data type for casting entities and timestamps.
  - BigQuery Version: Uses STRING data type for casting.
- Timestamp Arithmetic:
  - Redshift Version: Uses "t - x * interval '1' second" for timestamp arithmetic to compute TTL-based filtering.
  - BigQuery Version: Uses Timestamp_sub(...) for similar arithmetic.
- Column Handling:
  - Redshift Version: Replaces SELECT * EXCEPT (...) with SELECT * and later drops unnecessary columns, as Redshift doesn't support the EXCEPT clause.
  - BigQuery Version: Uses SELECT * EXCEPT (...), which is supported in BigQuery.
#### 2. Temporary Table Creation:
  - Redshift Version: Uses CTEs (WITH clauses) throughout to define intermediate steps but does not explicitly create temporary tables.
  - BigQuery Version: Explicitly creates temporary tables (CREATE TEMP TABLE) for intermediate results, making the process more memory-efficient in BigQuery.
#### 3. Handling of Entity Identifiers:
  - Redshift Version: Uses CONCAT and CAST to generate a unique identifier for each entity row.
  - BigQuery Version: Similar logic, but with CREATE TEMP TABLE for intermediate results and using BigQuery's specific syntax for generating unique IDs.
#### 4. Optimization Considerations:
  - Redshift Version: The code has TODO comments suggesting potential optimizations, such as using GENERATE_UUID() instead of ROW_NUMBER() and precomputing ROW_NUMBER() to avoid recomputation.
  - BigQuery Version: While not explicitly mentioned, the use of temporary tables and efficient subqueries suggests it is optimized for BigQuery's architecture.
#### 5. Deduplication Logic:
  - Redshift Version: Handles deduplication by calculating the MAX(created_timestamp) and joining back to filter out duplicates.
  - BigQuery Version: Similar approach but executed within the context of BigQuery's processing capabilities.
#### 6. Final Output Construction:
  - Both Versions: The final output joins the processed feature views back to the entity dataframe. The logic is similar, but the handling of intermediate steps differs due to the platform-specific optimizations.
#### Summary:
- Redshift Version: Tailored for Redshift, with modifications for data type handling, timestamp arithmetic, and the absence of the EXCEPT clause. It uses CTEs extensively.
- BigQuery Version: Optimized for BigQuery, with explicit temporary table creation and BigQuery-specific syntax. It takes advantage of BigQuery's features like EXCEPT and Timestamp_sub().
