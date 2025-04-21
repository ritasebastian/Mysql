If your **MySQL replication is stuck and never catches up**, here’s a **step-by-step deep-dive troubleshooting guide** to help you identify and resolve the root cause.

---

## 🚨 Problem: **MySQL replication never catches up**  
- `Seconds_Behind_Master` is always high or increasing  
- Replication threads show as `Running`, but the data is lagging  
- No obvious error in `SHOW SLAVE STATUS\G`

---

## ✅ Step-by-Step: Troubleshoot Replication Lag That Never Catches Up

---

### 🔍 Step 1: Check Actual Replication Status

On the **replica**, run:

```sql
SHOW SLAVE STATUS\G
```

Check:
- `Slave_IO_Running: Yes`
- `Slave_SQL_Running: Yes`
- `Seconds_Behind_Master: X`

> 🔸 If both threads are running, but `Seconds_Behind_Master` is constantly growing → SQL thread is **too slow** to apply events.

---

### 🔍 Step 2: Check for Long-Running Queries on Replica

```sql
SHOW PROCESSLIST;
```

Look for:
- Long `INSERT`, `UPDATE`, or `DELETE` queries
- Queries in "Waiting for table metadata lock"
- Any queries in `Copying to tmp table`, `Writing to net`, etc.

> 🔥 **Slow queries** can delay the SQL thread and block replication

---

### 🔍 Step 3: Check for Disk/IO Bottlenecks on Replica

Use Linux commands:

```bash
iostat -xz 1
vmstat 1
top -u mysql
df -h
```

Look for:
- High disk I/O wait times
- Low free space
- High CPU usage
- Disk throttling on EBS (AWS)

> 🛠️ Fix: Increase IOPS, optimize queries, or change volume type (`gp3`/`io2`)

---

### 🔍 Step 4: Check Slow Query Log on Replica

Enable and inspect:

```sql
SET GLOBAL slow_query_log = 1;
SET GLOBAL long_query_time = 1;

SHOW VARIABLES LIKE 'slow_query_log_file';
```

Tail the file:
```bash
tail -f /var/log/mysql/mysql-slow.log
```

Look for recurring slow queries that match replicated transactions.

---

### 🔍 Step 5: Monitor Relay Log Growth

Run:
```sql
SHOW SLAVE STATUS\G
```

Check:
- `Relay_Log_Space`: growing?
- `Exec_Master_Log_Pos`: not moving?

> If `Relay_Log_Space` is increasing but `Exec_Master_Log_Pos` is not → **SQL thread is stuck or too slow**

---

### 🔍 Step 6: Check Replica Filtering or Triggers

- Are you using:
  - `replicate_ignore_table`
  - `replicate_wild_do_table`
  - Triggers or foreign keys?

These can **slow down or silently block** replication.

---

### 🔍 Step 7: Schema Drift or Row Mismatch

If master and replica schema diverged:
- Replicated DMLs may apply slower or fail silently

Check:
```sql
CHECKSUM TABLE table_name;
```

Or better:
```bash
pt-table-checksum --host=replica --user=root --password=...
```

> 🧨 If tables don’t match, replication will slow or fail

---

### 🔍 Step 8: Monitor GTID / Binary Log Position

Compare master vs replica:

On master:
```sql
SHOW MASTER STATUS;
```

On replica:
```sql
SHOW SLAVE STATUS\G
```

Check:
- `Retrieved_Gtid_Set` vs `Executed_Gtid_Set`
- `Exec_Master_Log_Pos` vs `Read_Master_Log_Pos`

> If **gap is growing**, SQL thread is too slow

---

### 🔧 Step 9: Optimize the Replica

**Tuning suggestions:**

| Variable                        | Recommended Value            |
|--------------------------------|------------------------------|
| `innodb_flush_log_at_trx_commit` | 2 (replica only!)            |
| `sync_binlog`                  | 0 (replica only!)            |
| `innodb_io_capacity`           | Match EBS performance (e.g., 3000) |
| `innodb_read_io_threads`       | 4–8                          |
| `innodb_write_io_threads`      | 4–8                          |
| `slave_parallel_workers`       | 4–16 (MySQL 5.7+)            |
| `slave_parallel_type`          | LOGICAL_CLOCK                |

> 🔄 Restart MySQL after config changes or apply dynamically if allowed

---

## 🧼 Final Step: Restart Replication Cleanly (optional)

```sql
STOP SLAVE;
START SLAVE;
SHOW SLAVE STATUS\G
```

---

## ✅ Summary Checklist: Why Replica Never Catches Up

| Cause                         | Fix                                            |
|------------------------------|-------------------------------------------------|
| Slow queries on replica       | Optimize slow SQL or use multi-threaded replication |
| Disk I/O limits               | Upgrade EBS, tune IOPS                         |
| Schema drift or row conflict  | Use `pt-table-checksum` and fix schema         |
| Replica server too small      | Upgrade instance type or tune InnoDB           |
| Network lag (rare)            | Check latency, use internal IPs                |

---

Would you like a **replica health checker script** that outputs this info and alerts if the replica falls behind?
