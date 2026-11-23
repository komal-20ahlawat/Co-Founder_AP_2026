# Query Performance Benchmark Report

## 1. Benchmark Overview

- Database: PostgreSQL
- Database name: `cofounder`
- Total queries benchmarked: 26
- Runs per query: 3
- Measurement: PostgreSQL `Execution Time` from `EXPLAIN (ANALYZE, BUFFERS)`
- READ queries: R1–R11
- WRITE queries: W1–W15

# Query Execution Timings

All 26 database queries (R1–R11 and W1–W15) were benchmarked using PostgreSQL `EXPLAIN (ANALYZE, BUFFERS)`. Each query was executed three times, and the average execution time was calculated from those three runs.

Write queries were tested inside transactions and rolled back after each run to avoid permanently modifying the benchmark database.

## READ Queries

| Query | Type | What it does | Run 1 (ms) | Run 2 (ms) | Run 3 (ms) | Average (ms) | Scan type |
|---|---|---|---:|---:|---:|---:|---|
| R1 | READ | Find a user by email | 0.024 | 0.024 | 0.026 | 0.025 | Index Scan |
| R2 | READ | Retrieve login details with profile information | 0.057 | 0.031 | 0.038 | 0.042 | Nested Loop Join / Index Scan |
| R3 | READ | Retrieve user details by ID | 0.160 | 0.050 | 0.042 | 0.084 | Index Scan |
| R4 | READ | Retrieve a user's profile | 0.139 | 0.052 | 0.052 | 0.081 | Index Scan |
| R5 | READ | Retrieve all available skills | 0.055 | 0.029 | 0.031 | 0.038 | Sequential Scan / Sort |
| R6 | READ | Retrieve skills selected by a user | 0.093 | 0.057 | 0.039 | 0.063 | Hash Join / Sequential Scan / Bitmap Scan |
| R7 | READ | Retrieve all available interests | 0.073 | 0.035 | 0.024 | 0.044 | Sequential Scan / Sort |
| R8 | READ | Retrieve interests selected by a user | 0.093 | 0.070 | 0.057 | 0.073 | Hash Join / Sequential Scan / Bitmap Scan |
| R9 | READ | Retrieve all projects with founder, skills, roles and member count | 0.065 | 0.055 | 0.075 | 0.065 | Hash Left Join / Sequential Scan / SubPlans |
| R10 | READ | Retrieve projects created by other users | 0.143 | 0.097 | 0.080 | 0.107 | Sort / Hash Left Join / Sequential Scan / SubPlans |
| R11 | READ | Retrieve projects created by the logged-in user | 0.101 | 0.464 | 0.058 | 0.208 | Sort / Nested Loop Left Join / Index Scan / SubPlans |

## WRITE Queries

| Query | Type | What it does | Run 1 (ms) | Run 2 (ms) | Run 3 (ms) | Average (ms) | Scan type |
|---|---|---|---:|---:|---:|---:|---|
| W1 | WRITE | Create a new user account | 0.126 | 0.119 | 0.119 | 0.121 | N/A – INSERT |
| W2 | WRITE | Create the initial user profile | 0.344 | 0.272 | 0.278 | 0.298 | N/A – INSERT |
| W3 | WRITE | Update a user's profile information | 0.082 | 0.068 | 0.072 | 0.074 | Index Scan – UPDATE |
| W4 | WRITE | Insert or retrieve an existing skill | 0.267 | 0.215 | 0.156 | 0.213 | N/A – INSERT ON CONFLICT |
| W5 | WRITE | Link a skill to a user | 0.571 | 0.319 | 0.319 | 0.403 | N/A – INSERT ON CONFLICT |
| W6 | WRITE | Remove a skill from a user's profile | 0.091 | 0.069 | 0.053 | 0.071 | Index Scan – DELETE |
| W7 | WRITE | Insert or retrieve an existing interest | 0.165 | 0.139 | 0.241 | 0.182 | N/A – INSERT ON CONFLICT |
| W8 | WRITE | Link an interest to a user | 0.359 | 0.295 | 0.338 | 0.331 | N/A – INSERT ON CONFLICT |
| W9 | WRITE | Remove an interest from a user's profile | 0.040 | 0.036 | 0.037 | 0.038 | Index Scan – DELETE |
| W10 | WRITE | Create a new project | 0.288 | 0.414 | 0.292 | 0.331 | N/A – INSERT |
| W11 | WRITE | Add the project owner as Founder | 0.429 | 0.192 | 0.212 | 0.278 | N/A – INSERT ON CONFLICT |
| W12 | WRITE | Insert or retrieve a required project skill | 0.358 | 0.139 | 0.124 | 0.207 | N/A – INSERT ON CONFLICT |
| W13 | WRITE | Link a required skill to a project | 0.542 | 0.279 | 0.339 | 0.387 | N/A – INSERT ON CONFLICT |
| W14 | WRITE | Add a required role to a project | 0.463 | 0.185 | 0.169 | 0.272 | N/A – INSERT |
| W15 | WRITE | Delete a project owned by the logged-in user | 1.626 | 0.774 | 0.765 | 1.055 | Index Scan – DELETE |


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
