# Query Performance Benchmark Report

## 1. Benchmark Overview

- Database: PostgreSQL
- Database name: `cofounder`
- Benchmark date: 23 November 2026
- Total queries benchmarked: 26
- Runs per query: 3
- Measurement: PostgreSQL `Execution Time` from `EXPLAIN (ANALYZE, BUFFERS)`
- READ queries: R1–R11
- WRITE queries: W1–W15

All WRITE benchmarks were executed inside transactions and rolled back after each run. No permanent test records were retained.

## 2. Query Timing Results

| Query | Type | What it does | Run 1 (ms) | Run 2 (ms) | Run 3 (ms) | Average (ms) | Scan type |
|---|---|---|---:|---:|---:|---:|---|
| R1 | READ | Login by email | 0.191 | 0.016 | 0.015 | 0.074 | Index Scan |
| R2 | READ | Login with profile details | 0.292 | 0.025 | 0.025 | 0.114 | Nested Loop Left Join |
| R3 | READ | Get user by ID | 0.013 | 0.013 | 0.039 | 0.022 | Index Scan |
| R4 | READ | Get user profile | 0.014 | 0.013 | 0.012 | 0.013 | Index Scan |
| R5 | READ | Get all skills | 0.039 | 0.036 | 0.018 | 0.031 | Seq Scan + Sort |
| R6 | READ | Get skills of a user | 0.032 | 0.031 | 0.064 | 0.042 | Hash Join + Bitmap/Seq Scans |
| R7 | READ | Get all interests | 0.019 | 0.023 | 0.040 | 0.027 | Seq Scan + Sort |
| R8 | READ | Get interests of a user | 0.029 | 0.068 | 0.031 | 0.043 | Hash Join + Bitmap/Seq Scans |
| R9 | READ | Get all projects | 0.056 | 0.090 | 0.080 | 0.075 | Hash Joins + Seq Scans + Sort |
| R10 | READ | Get projects excluding own projects | 0.075 | 0.096 | 0.034 | 0.068 | Hash Joins + Seq Scans + Sort |
| R11 | READ | Get projects by a specific owner | 0.025 | 0.058 | 0.028 | 0.037 | Nested Loop Left Join + Index Scan |
| W1 | WRITE | Create user account | 1.228 | 0.015 | 0.015 | 0.419 | — |
| W2 | WRITE | Create initial profile | 0.552 | 0.029 | 0.024 | 0.202 | — |
| W3 | WRITE | Update profile | 0.525 | 0.014 | 0.016 | 0.185 | — |
| W4 | WRITE | Insert or update a skill | 0.858 | 0.018 | 0.016 | 0.297 | — |
| W5 | WRITE | Link skill to user | 2.093 | 0.045 | 0.040 | 0.726 | — |
| W6 | WRITE | Remove skill from user | 0.091 | 0.009 | 0.008 | 0.036 | — |
| W7 | WRITE | Insert or update an interest | 1.114 | 0.019 | 0.015 | 0.383 | — |
| W8 | WRITE | Link interest to user | 1.561 | 0.045 | 0.042 | 0.549 | — |
| W9 | WRITE | Remove interest from user | 0.044 | 0.006 | 0.009 | 0.020 | — |
| W10 | WRITE | Create a project | 2.060 | 0.040 | 0.036 | 0.712 | — |
| W11 | WRITE | Add project owner as Founder | 1.445 | 0.037 | 0.037 | 0.506 | — |
| W12 | WRITE | Insert or find a required skill | 0.169 | 0.018 | 0.014 | 0.067 | — |
| W13 | WRITE | Link required skill to project | 1.895 | 0.035 | 0.027 | 0.652 | — |
| W14 | WRITE | Add a required role to project | 1.130 | 0.030 | 0.019 | 0.393 | — |
| W15 | WRITE | Delete a project owned by the user | 0.860 | 0.050 | 0.040 | 0.317 | — |

## 3. Benchmark Methodology

Each query was executed three times using PostgreSQL's:

`EXPLAIN (ANALYZE, BUFFERS)`

The average execution time was calculated as:

`Average = (Run 1 + Run 2 + Run 3) / 3`

Execution Time was used rather than Planning Time.

For WRITE queries, temporary test records were created where necessary. Each benchmark was enclosed in a transaction and rolled back to prevent permanent changes to the database.

For queries requiring related records, temporary parent and child records were created inside the same transaction.

## 4. Database Conditions and Limitations

The benchmark was performed against the local `cofounder` database with a small amount of existing dummy user and profile data.

- The `users` and `profiles` tables contained three existing test records.
- The `skills`, `user_skills`, `interests`, and `user_interests` tables were empty before the temporary benchmark transactions.
- The project-related tables had no permanent project records before benchmarking.
- Temporary records used for WRITE benchmarks were rolled back.

Consequently, several READ queries returned zero rows. Their timings represent execution against the current small test database and should not be interpreted as production-scale performance.

Execution times can vary depending on PostgreSQL caching, system load, buffer state, and database size.

## 5. Observations

1. Login and profile retrieval queries used indexes on email, user ID, and profile user ID.
2. Skill and interest listing queries used sequential scans and sorting on the small tables.
3. Project listing queries involved joins and subqueries to retrieve skills, roles, and member counts.
4. Several WRITE queries involved foreign-key checks and unique-index conflict handling.
5. Project deletion used an index on `owner_id` and triggered checks on related tables through foreign-key cascade constraints.
6. The first execution of several WRITE queries was slower than subsequent executions, partly due to buffer reads and writes.

## 6. Detailed Execution Plans

Individual execution plans for the three runs of each query are stored in the `dbms/plans/` directory.
