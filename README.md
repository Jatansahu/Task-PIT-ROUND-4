# Task-PIT-ROUND-4
This repository contains an analysis and implementation of Point-in-Time (PIT) joins to prevent data leakage in machine learning models. It includes a detailed examination of the MULTIPLE_FEATURE_VIEW_POINT_IN_TIME_JOIN function from Feast, covering edge cases, usage, and a custom Python implementation of PIT joins without SQL.

## TASK 

Understand the concept of Point in time Joins here [Point-in-time joins | Feast]([https://pages.github.com/](https://docs.feast.dev/getting-started/concepts/point-in-time-joins) 

Point in time joins
are very important while creating training data for ML models and are critical to ensuring that
there is no data/feature leakage.

## Problem Statement

Here is an open source implementation of a Point in Time join [query in Feast](https://github.com/feast-dev/feast/blob/b0dc6832ff446429390a916aa9e0e61066cbde1d/sdk/python/feast/infra/offline_stores/redshift.py#L633

1. Understand this query and what edge cases it considers

2. Figure out where and how MULTIPLE_FEATURE_VIEW_POINT_IN_TIME_JOIN is
being called/referenced throughout this [repo](https://github.com/feast-dev/feast).

3. Implement the PIT join yourself in Python (no usage of SQL).








Understanding Point-in-Time Joins
Point-in-time joins are a technique used to combine data from different sources in a way that ensures no "future" information is leaked into the past when creating training datasets for machine learning models. This is crucial because in real-world applications, you won't have access to future data at the time a prediction is made.

Feast (Feature Store) is an open-source feature store for machine learning, and the query you are examining is part of Feast’s implementation to support point-in-time joins.

Analyzing the Query
Here’s a breakdown of the key aspects of the query linked:

General Purpose:
The query is designed to perform a point-in-time join between the entity data (which likely contains timestamps) and the feature data stored in Redshift. It ensures that only the feature data available before or at the timestamp of the entity data is considered.

Handling Edge Cases:

Time Lag Consideration: The query likely includes logic to ensure that features are not joined from a future time relative to the entity's timestamp.

Duplicated or Missing Timestamps: The query may consider scenarios where there are multiple records with the same timestamp or missing timestamps and handle them appropriately to avoid incorrect feature assignment.

Data Overlap: Ensures that if there are multiple potential matches (e.g., if a feature exists at several points in time close to the entity timestamp), it correctly selects the most relevant one based on some logic (like closest time before the entity timestamp).

Data Consistency: The query needs to ensure consistency in the case of data being updated or backfilled after the fact. It prevents leaking this backfilled data into past predictions by only joining data that was genuinely available at the time.

Key Components of the Query:
Partitioning and Windowing: The query might use SQL window functions to correctly partition data by entity and order it by timestamp, ensuring that only the most relevant past data is joined.

Filters and Conditions: These help in eliminating rows that might cause data leakage, such as filtering out feature rows that occur after the entity timestamp.

Edge Case Handling: The query might include specific conditions to deal with missing data or ensure that in case of ties (multiple feature rows with the same timestamp), a consistent method of selection is used.

Conclusion
Understanding this query involves recognizing how it manages data in time-sensitive contexts, ensuring that the training data remains clean and unbiased by future information. This is crucial for maintaining the integrity and effectiveness of machine learning models, as it helps prevent data leakage that could artificially inflate model performance during training.
