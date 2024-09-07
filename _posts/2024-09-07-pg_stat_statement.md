---
title: Using pg_stat_statements to Track SQL Planning and Execution in PostgreSQL
date: 2024-09-07
categories: [PostgreSQL, Monitoring]
tags: [pg_stat_statements, SQL, performance, monitoring]
toc: true
toc_label: "On this Page"
---

The `pg_stat_statements` module in PostgreSQL provides a way to track the planning and execution statistics of all SQL statements executed by the server. It collects data across all databases on the server and offers insights via the views `pg_stat_statements` and `pg_stat_statements_info`, along with utility functions `pg_stat_statements_reset` and `pg_stat_statements`.

### References:

- [PostgreSQL Official Documentation](https://www.postgresql.org/docs/current/pgstatstatements.html)
- [Citus Data Blog](https://www.citusdata.com/blog/2019/02/08/the-most-useful-postgres-extension-pg-stat-statements/)
- [Data Sentinel Blog](https://blog.datasentinel.io/post/pg_stats_statements/)
- [Timescale Blog](https://www.timescale.com/blog/using-pg-stat-statements-to-optimize-queries/)

### Step 1: Enable the Extension

To start using `pg_stat_statements`, you need to enable the extension and configure the PostgreSQL settings:

```bash
# Edit the postgresql.conf file
vim postgresql.conf

# Add the following settings
shared_preload_libraries = 'pgaudit,pgauditlogtofile,pglogical,pg_stat_statements'
pg_stat_statements.max = 1000              
pg_stat_statements.track = 'all'
log_min_duration_statement = 30000    # 30 seconds or higher
log_duration = off       # Set to off or on. 'On' creates huge log files based on traffic
```

### Step 2: Create the Extension

```sql
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
```

### Step 3: Monitor SQL Performance

#### Viewing Query Execution Statistics

```sql
SELECT * FROM pg_stat_statements;
``` 

#### Resetting Statistics

```sql
SELECT pg_stat_statements_reset();
``` 

#### Analyzing Query Execution

```sql
SELECT query, calls, total_time, rows, shared_blks_hit, shared_blks_read, shared_blks_dirtied, shared_blks_written, temp_blks_read, temp_blks_written
FROM pg_stat_statements
ORDER BY total_time DESC
LIMIT 10;
```         
### Usage Examples
1. Queries Based on Execution Time
```sql
SELECT queryid, query, total_exec_time, rows
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```

2. Queries Based on Number of Calls
```sql
SELECT queryid, query, calls
FROM pg_stat_statements
ORDER BY calls DESC
LIMIT 10;
```

3. Queries Based on Number of Rows Processed
```sql
SELECT queryid, query, rows
FROM pg_stat_statements
ORDER BY rows DESC
LIMIT 10;
```

4. Queries Based on Shared Blocks Hit
```sql
SELECT queryid, query, shared_blks_hit, shared_blks_read
FROM pg_stat_statements
ORDER BY shared_blks_hit DESC
LIMIT 10;
``` 

5. Queries Based on Temporary Blocks
```sql
SELECT queryid, query, temp_blks_read, temp_blks_written
FROM pg_stat_statements
ORDER BY temp_blks_read DESC
LIMIT 10;
```         

6. Queries Based on Disk Writes
```sql
SELECT queryid, query, blks_read, blks_written
FROM pg_stat_statements
ORDER BY blks_written DESC
LIMIT 10;
```         

7. Queries Based on Disk Reads
```sql
SELECT queryid, query, blks_read
FROM pg_stat_statements
ORDER BY blks_read DESC
LIMIT 10;
```                    

8. Queries Generating Most WAL Records (For Master Database)
```sql
SELECT queryid, query, wal_records
FROM pg_stat_statements
ORDER BY wal_records DESC
LIMIT 10;
```

9. Queries with High I/O Activity
```sql
SELECT queryid, query, rows, blks_read, blks_written
FROM pg_stat_statements
ORDER BY (blks_read + blks_written) DESC
LIMIT 10;
```

10. Queries Causing More Locks
```sql
SELECT queryid, query, locks_hit, locks_missed
FROM pg_stat_statements
ORDER BY locks_missed DESC
LIMIT 10;
```

### Reset pg_stat_statements

```sql
SELECT pg_stat_statements_reset();
```
- This will reset the statistics collected by `pg_stat_statements`.

### Conclusion

`pg_stat_statements` is a powerful tool for analyzing and optimizing SQL performance in PostgreSQL. By tracking query execution statistics, you can identify the most resource-intensive queries, understand their impact on the system, and make informed decisions to improve performance and resource usage.

Remember to regularly monitor these statistics and adjust settings as needed to ensure optimal performance and resource utilization.