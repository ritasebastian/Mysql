Here's a **step-by-step guide** to **troubleshoot MySQL replication issues**, especially for **asynchronous (master-replica)** replication using `CHANGE MASTER TO` or GTID.

This checklist applies to **MySQL 5.7, 8.0**, and even **Percona or MariaDB** (with minor differences).

---

## ✅ Step-by-Step MySQL Replication Troubleshooting Guide

---

### 🔹 Step 1: Check Replication Status

On the **replica server**, run:

```sql
SHOW SLAVE STATUS\G    -- (MySQL 5.7)
SHOW REPLICA STATUS\G  -- (MySQL 8.0+)
```

📌 Focus on these key fields:

| Field                     | Meaning                        |
|---------------------------|--------------------------------|
| `Slave_IO_Running`        | Is the I/O thread (binlog fetcher) running? |
| `Slave_SQL_Running`       | Is the SQL thread (replayer) running? |
| `Last_Errno`, `Last_Error`| Shows error that stopped replication |
| `Seconds_Behind_Master`   | Delay between master & replica |
| `Retrieved_Gtid_Set`      | GTIDs received (GTID mode)     |
| `Executed_Gtid_Set`       | GTIDs applied (GTID mode)      |

---

### 🔹 Step 2: Common Replication Errors

| Symptom                          | Likely Cause                                | Fix |
|----------------------------------|---------------------------------------------|-----|
| `Slave_IO_Running: No`           | Cannot connect to master                    | Check host, port, firewall, credentials |
| `Slave_SQL_Running: No`          | Query error while replaying binlog         | Investigate `Last_Error`                |
| `Seconds_Behind_Master: NULL`    | SQL thread stopped or lagged               | Resume or fix broken event              |
| `Duplicate entry...` or `PK error` | Conflicting data between master/replica  | Use `SKIP` or re-sync clean             |
| GTID conflict / executed set mismatch | Replica out of sync with GTID state   | Reset GTID + restore from backup        |

---

### 🔹 Step 3: Fix Broken Replication

#### ✅ If replication has stopped (but no data conflict):

```sql
STOP SLAVE;  -- or STOP REPLICA;
START SLAVE; -- or START REPLICA;
```

Check again:
```sql
SHOW SLAVE STATUS\G
```

---

### 🔹 Step 4: Handle Duplicate Key or Missing Row Errors

```sql
SHOW SLAVE STATUS\G
```

If error is like:

```
Last_SQL_Error: Error 'Duplicate entry...' on query ...
```

🔧 Option 1: Skip the problematic transaction (not ideal long-term):
```sql
SET GLOBAL SQL_SLAVE_SKIP_COUNTER = 1;
START SLAVE;
```

🔧 Option 2: Rebuild replica (best if data drifted):
- Stop slave
- Take fresh backup from master (`mysqldump` or `xtrabackup`)
- Restore on replica
- Reconfigure `CHANGE MASTER TO ...`

---

### 🔹 Step 5: GTID-Based Replication Fix (if using GTID)

If the replica is out of sync:

```sql
STOP SLAVE;
RESET SLAVE ALL;
RESET MASTER;
```

Then reconfigure:

```sql
CHANGE MASTER TO
  MASTER_HOST='your.master.ip',
  MASTER_USER='repl',
  MASTER_PASSWORD='yourpass',
  MASTER_AUTO_POSITION=1;

START SLAVE;
```

---

### 🔹 Step 6: Confirm Replication is Running

```sql
SHOW SLAVE STATUS\G
```

Check for:
- `Slave_IO_Running: Yes`
- `Slave_SQL_Running: Yes`
- `Seconds_Behind_Master`: should decrease

---

### 🔹 Step 7: Check for Silent Lag

Replica may appear OK, but data is behind:

```sql
SHOW PROCESSLIST;
SHOW SLAVE STATUS\G
```

If `Seconds_Behind_Master` is high:
- Check slow queries
- Check disk I/O
- Check replication filters (`replicate_ignore_db`, etc.)

---

## 🧠 BONUS: Monitoring Checklist

| Checkpoint                | Recommended Tool / Command       |
|---------------------------|----------------------------------|
| Replication status        | `SHOW SLAVE STATUS\G`            |
| I/O/SQL thread alive      | `mysqladmin processlist`         |
| Logs                      | `/var/log/mysql/error.log`       |
| Network issues            | `telnet master_ip 3306`, `ping`  |
| GTID alignment            | `SHOW MASTER STATUS` vs `SHOW SLAVE STATUS` |

---

