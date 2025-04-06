
---

## 🔥 TOUGH MYSQL INTERVIEW QUESTIONS & ANSWERS (with context)

---

### 🔸 1. **What’s the difference between `REPEATABLE READ` and `READ COMMITTED` in InnoDB? How does it affect phantom reads?**

**Answer:**

- **READ COMMITTED**: Each SELECT reads fresh committed data. Allows **non-repeatable reads** and **phantom reads**.
- **REPEATABLE READ** (default in InnoDB): Ensures same result for the same query within a transaction. It prevents **non-repeatable reads** but not phantom reads.

🧠 However, **InnoDB uses next-key locking** in REPEATABLE READ to prevent **phantom reads**, making it very close to `SERIALIZABLE`.

---

### 🔸 2. **What is the difference between `binlog_format = ROW` vs `STATEMENT`? Which one is preferred in HA setups?**

**Answer:**

- `STATEMENT`: Logs the SQL itself. Lightweight but not deterministic (may cause replication issues).
- `ROW`: Logs **actual row changes**. Safer, consistent, but more storage-heavy.
- `MIXED`: Hybrid (MySQL decides).

✅ In HA/Replication/GTID setups, **ROW format** is recommended for consistency and data integrity.

---

### 🔸 3. **How do you perform zero-downtime DDL changes in MySQL?**

**Answer:**

Use **Percona’s `pt-online-schema-change`** or **`gh-ost`** to perform non-blocking schema migrations.

- These tools create a shadow copy of the table and apply changes live.
- Avoids table locking or downtime for large tables.
- Not supported in RDS by default unless hosted in EC2 or you use MySQL 8+ with `ALTER ONLINE`.

---

### 🔸 4. **What is GTID-based replication? Why is it better than traditional replication?**

**Answer:**

- **GTID** = Global Transaction ID
- Provides a **unique ID per transaction** → easier failover & monitoring.
- In contrast, traditional replication uses **log file + position**, which is harder to automate.

✅ GTID simplifies:
- Multi-source replication
- Semi-sync setups
- Failover tools like `MHA`, `Orchestrator`

---

### 🔸 5. **How would you troubleshoot a sudden replication lag?**

**Answer:**

1. Check replica I/O & SQL thread status:
   ```sql
   SHOW SLAVE STATUS\G
   ```
2. Look for:
   - `Seconds_Behind_Master`
   - `Slave_IO_Running` / `Slave_SQL_Running`
3. Investigate:
   - Long-running queries on replica
   - Disk I/O issues
   - Lock contention
   - Network latency
4. Check slow query log or `performance_schema`.

---

### 🔸 6. **What is the purpose of `mysql_upgrade`? Is it still needed in MySQL 8.0?**

**Answer:**

- In older versions, `mysql_upgrade` updates system tables after upgrades.
- In MySQL 8+, it runs **automatically during startup** → manual call is optional.
- Still useful to confirm system tables are up-to-date or to fix minor inconsistencies.

---

### 🔸 7. **How do you handle a table with 2TB of data that needs to be archived?**

**Answer:**

Options:
- Use **partitioning by date** or range, then `DROP PARTITION`
- Create an **archive table** and move data in chunks:
  ```sql
  INSERT INTO archive_table SELECT * FROM main_table WHERE date < '2023';
  DELETE FROM main_table WHERE date < '2023' LIMIT 10000;
  ```
- Use external tools like:
  - **pt-archiver**
  - **MySQL Shell + Dump Utilities**
- Ensure minimal locks and perform during off-peak hours.

---

### 🔸 8. **What are `UNDO` and `REDO` logs in InnoDB?**

**Answer:**

- **UNDO log**: Keeps pre-change state for rollback and MVCC
- **REDO log**: Ensures committed changes can be replayed after crash

Together, they guarantee **ACID** compliance in InnoDB.

---

### 🔸 9. **How do you monitor MySQL health in production?**

**Answer:**

Use a combination of:

- **Performance Schema**
- **Slow Query Log**
- **SHOW GLOBAL STATUS**
- Tools:
  - **Percona Monitoring and Management (PMM)**
  - **AppDynamics, Datadog, New Relic**
  - **Cloud-native tools** (CloudWatch, Stackdriver)

Monitor:
- QPS
- Connection usage
- Thread states
- Replication lag
- InnoDB buffer pool hit ratio

---

### 🔸 10. **What happens when InnoDB runs out of undo space?**

**Answer:**

- Long-running transactions may block purge operations
- You’ll see warnings like:
  ```
  "Undo log overflow"
  ```
- Solutions:
  - Break large transactions into smaller chunks
  - Monitor and increase `innodb_undo_tablespaces` and `innodb_max_undo_log_size`

---

## 🔥 Bonus Tip: Ask This Back

> “What tooling do you use for schema versioning and deployment (e.g., Flyway, Liquibase, gh-ost, pt-osc)?”

Shows you think **beyond MySQL** and care about **DevOps/Platform**.

---
