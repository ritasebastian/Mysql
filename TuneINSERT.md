
---

### ✅ 1. **Use Bulk Inserts**
Instead of inserting rows one by one:
```sql
-- Slow:
INSERT INTO my_table (col1, col2) VALUES (1, 'a');
INSERT INTO my_table (col1, col2) VALUES (2, 'b');
```
Use:
```sql
-- Faster:
INSERT INTO my_table (col1, col2) VALUES (1, 'a'), (2, 'b'), (3, 'c');
```

---

### ✅ 2. **Disable Indexes Temporarily (for large batches only)**
If you're doing **massive inserts** (like from a CSV):
```sql
ALTER TABLE my_table DISABLE KEYS;
-- Perform inserts
ALTER TABLE my_table ENABLE KEYS;
```
> Only applies to **MyISAM**, not **InnoDB**.

---

### ✅ 3. **Use Transactions Wisely (InnoDB)**
Group many inserts into one transaction:
```sql
START TRANSACTION;
-- many inserts
COMMIT;
```
> Avoid committing each insert individually. This reduces disk I/O.

---

### ✅ 4. **Optimize InnoDB Settings (for insert-heavy workloads)**
In your `my.cnf` or `my.ini`:
```ini
innodb_flush_log_at_trx_commit = 2
innodb_buffer_pool_size = 70-80% of RAM
innodb_log_buffer_size = 64M
innodb_log_file_size = 1G (or larger)
```
> `innodb_flush_log_at_trx_commit = 2` gives better performance at the cost of minimal durability risk.

---

### ✅ 5. **Avoid Unnecessary Indexes**
Each insert must update all indexes. Keep only **necessary** indexes during bulk insert.

---

### ✅ 6. **Use `LOAD DATA INFILE` for Very Fast Bulk Insert**
```sql
LOAD DATA INFILE '/path/to/data.csv'
INTO TABLE my_table
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n';
```
> 10–20x faster than `INSERT`.

---

### ✅ 7. **Partition Large Tables**
Partitioning can reduce index size and improve write speed:
```sql
PARTITION BY RANGE (YEAR(created_at)) ...
```

---

### ✅ 8. **Avoid Triggers and Foreign Keys During Bulk Load**
Disable temporarily (for controlled environments only):
```sql
SET foreign_key_checks = 0;
-- insert
SET foreign_key_checks = 1;
```

---

### ✅ 9. **Batch from Application Side**
Avoid inserting millions of rows one-by-one from app. Send in **batches** (e.g., 500–1000 rows per batch).

---

### ✅ 10. **Use Prepared Statements**
Prepared statements reduce parsing and planning overhead.

---

