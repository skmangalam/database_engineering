**PostgreSQL Query Planning, Scans, and Caching - Deep Notes**

---

### 1. **Primary Key & Indexing**

* Primary key is **not mandatory** in PostgreSQL but highly recommended.
* Creating a primary key **automatically creates a unique B-Tree index**.
* PostgreSQL primary key is **not clustered by default** (unlike MySQL).

    * Use `CLUSTER tablename USING indexname;` to physically order heap.
    * This is a one-time operation and doesn’t persist across inserts.

---

### 2. **ctid & Heap Pages**

* Every row has an internal `ctid` = (block number aka page number, tuple index).
* `tuple_index` **starts at 0 for each page**, it is not global.
* `ctid` is **not indexed** and is used internally to point to the exact location in heap.
* Index leaf nodes store `ctid` to find corresponding heap tuples.

---

### 3. **Caching Behavior**

* PostgreSQL uses **shared\_buffers** (global buffer cache per instance, not per table).
* If a page is cached and another row from same page is queried → **no disk I/O needed**.
* Table is considered "small" and always cached if size < shared\_buffers (\~100MB-200MB).
* You can monitor cache hit ratio using:

```sql
SELECT relname, blks_hit, blks_read, ROUND(100 * blks_hit / NULLIF(blks_hit + blks_read, 0), 2) AS hit_ratio
FROM pg_stat_user_tables ORDER BY hit_ratio DESC;
```

---

### 4. **Count(\*) Strategies**

* `SELECT COUNT(*) FROM table;` is accurate but slow (full table scan).
* `EXPLAIN SELECT * FROM table;` gives **estimates**, not actual row count.
* For fast approximate count:

```sql
SELECT reltuples::BIGINT FROM pg_class WHERE relname = 'your_table';
```

* For fast access in prod, use **Redis counter with periodic sync from DB**.

    * Increment/decrement Redis on write.
    * Sync Redis count with `SELECT COUNT(*)` hourly.
    * Fallback: If Redis is down, fetch from DB and repopulate Redis.

---

### 5. **Scan Types in PostgreSQL**

| Type             | Used When                                            | Notes                                            |
| ---------------- | ---------------------------------------------------- | ------------------------------------------------ |
| Seq Scan         | Table is small or >90% rows needed                   | Full table scan, no index used                   |
| Index Scan       | Few rows, selective filter                           | Heap access per index hit (random I/O)           |
| Index Only Scan  | All needed columns in index + visibility map bit = 1 | No heap access at all                            |
| Bitmap Heap Scan | Mid-selectivity (\~5-20%), rows from clustered pages | Bitmap built from index, then batched heap fetch |

**Bitmap Scan Process**:

1. PostgreSQL walks index to find matching `ctid`s.
2. Builds bitmap of needed heap pages.
3. Fetches heap pages in batch.
4. Filters out required rows.

---

### 6. **EXPLAIN vs EXPLAIN ANALYZE**

| Command         | Runs Query? | Shows Actual Timing? | Use Case                                   |
| --------------- | ----------- | -------------------- | ------------------------------------------ |
| EXPLAIN         | No          | No                   | To see planner's estimate                  |
| EXPLAIN ANALYZE | Yes         | Yes                  | To see real performance and tuning insight |

---

### 7. **Parallelism**

* Tune via:

```sql
SET max_parallel_workers = 8;
SET max_parallel_workers_per_gather = 4;
```

* Planner automatically uses parallelism if cost justifies it.
* Use for large analytical queries, not OLTP.

---

### 8. **BIGINT for Large PKs**

* `INT` (4 bytes) maxes out at \~2.1B.
* Use `BIGSERIAL` (8 bytes) for auto-increment on massive tables.
* UUID can also be used but is 16 bytes and slower for joins.

---