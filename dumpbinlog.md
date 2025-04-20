Dumping and analyzing **MySQL binary logs (binlogs)** is a great way to **audit activity**, **replicate data**, or **recover from data loss**. Here's a complete guide with **real-world examples** and usage tips.

---

## 📦 What is a Binlog?

The **binary log** (`binlog`) is a set of log files that record all changes to MySQL data (e.g., `INSERT`, `UPDATE`, `DELETE`) in binary format. It’s used for:
- 🔁 Replication
- 💥 Point-in-time recovery
- 🔍 Auditing changes

---

## ✅ Step 1: Enable Binlog in MySQL

In your `my.cnf`:
```ini
[mysqld]
server-id=1
log-bin=mysql-bin
binlog_format=ROW
expire_logs_days=7
```
Restart MySQL:
```bash
sudo systemctl restart mysqld
```

---

## 🔍 Step 2: View Binary Log Files

```bash
mysql -uroot -p -e "SHOW BINARY LOGS;"
```

Example Output:
```
+------------------+-----------+
| Log_name         | File_size|
+------------------+-----------+
| mysql-bin.000001 | 345       |
| mysql-bin.000002 | 298       |
+------------------+-----------+
```

---

## 🔧 Step 3: Use `mysqlbinlog` to Dump a Binlog

### Example: Dump entire log
```bash
mysqlbinlog /var/lib/mysql/mysql-bin.000001
```

### Example: Dump with timestamps
```bash
mysqlbinlog --verbose --base64-output=DECODE-ROWS /var/lib/mysql/mysql-bin.000001
```

> Needed if `binlog_format=ROW` is used (which is default in newer MySQL versions)

---

### Example: Dump logs between dates
```bash
mysqlbinlog --start-datetime="2025-04-20 09:00:00" \
            --stop-datetime="2025-04-20 10:00:00" \
            /var/lib/mysql/mysql-bin.000001
```

---

### Example: Dump logs for a specific database
```bash
mysqlbinlog --database=mydb /var/lib/mysql/mysql-bin.000001
```

---

### Example: Save to file for auditing or recovery
```bash
mysqlbinlog /var/lib/mysql/mysql-bin.000001 > binlog_dump.sql
```

Then you can reapply changes:
```bash
mysql -u root -p < binlog_dump.sql
```

---

## 🧠 When to Use `mysqlbinlog`

| Use Case                             | How It Helps                              |
|--------------------------------------|-------------------------------------------|
| 🔁 Replication sync/diagnosis        | See what’s being sent to replica          |
| 💥 Data recovery (point-in-time)     | Roll forward from last full backup        |
| 🔍 Auditing activity                 | Investigate who changed what              |
| 📊 Debugging                         | See exact queries that ran                |

---

## ⚠️ Notes

- File path may vary: e.g., `/var/lib/mysql/mysql-bin.*` (check with `SHOW VARIABLES LIKE 'log_bin%'`)
- MySQL user must have **`RELOAD` or `SUPER`** privileges to read from binlog
- Use `mysqlbinlog` from same version as the MySQL server

---

Awesome! Let's walk through **how to extract queries from MySQL binlog** based on **timestamp** or **user activity** using `mysqlbinlog`.

---

## ✅ Scenario 1: Extract Queries Between Two Timestamps

### 🕓 Example: Between 9:00 AM and 10:00 AM on April 20, 2025
```bash
mysqlbinlog \
  --start-datetime="2025-04-20 09:00:00" \
  --stop-datetime="2025-04-20 10:00:00" \
  --verbose --base64-output=DECODE-ROWS \
  /var/lib/mysql/mysql-bin.000001 > queries_9to10.sql
```

> This command will decode row-based events and save the output to a readable SQL file (`queries_9to10.sql`).

---

## ✅ Scenario 2: Extract Queries by a Specific User (Indirect Method)

Unfortunately, **binlogs do not store the MySQL user directly**. However, you can:

### ✅ Combine with General Log or Audit Plugin:
1. Enable the **general query log**:
   ```sql
   SET GLOBAL general_log = 'ON';
   SET GLOBAL log_output = 'TABLE';
   ```

2. Query the general log for specific user:
   ```sql
   SELECT * FROM mysql.general_log
   WHERE user_host LIKE 'myuser@%'
     AND event_time BETWEEN '2025-04-20 09:00:00' AND '2025-04-20 10:00:00';
   ```

3. Cross-check this with binlog time window using `mysqlbinlog` as shown above.

---

## 🔁 Replaying the Extracted Queries (Recovery)

Once you've extracted the required part:
```bash
mysql -u root -p mydatabase < queries_9to10.sql
```

---

## 🛡️ Tip for Future Auditing (Optional)

If you want to **track per-user actions more clearly**, consider enabling:
- **Audit plugin** (e.g., `audit_log` plugin by Percona)
- **ProxySQL logging** for user activity
- **Binlog with COMMENT** tags (if added from application queries)

---

Perfect! Let’s go step-by-step to set up a **proper audit trail** in MySQL to **track user activity**, especially if you're dealing with production logs or sensitive changes.

---

## 🔍 3 Best Options to Track User Activity in MySQL

---

### ✅ Option 1: Enable and Query the **General Query Log**

Good for **development or small production windows**.

#### Step 1: Turn on General Log (writes to table)
```sql
SET GLOBAL general_log = 'ON';
SET GLOBAL log_output = 'TABLE';
```

#### Step 2: View user-specific queries
```sql
SELECT event_time, user_host, argument
FROM mysql.general_log
WHERE user_host LIKE 'myuser@%'
  AND event_time BETWEEN '2025-04-20 09:00:00' AND '2025-04-20 10:00:00';
```

#### Step 3: Turn it off after auditing
```sql
SET GLOBAL general_log = 'OFF';
```

> ⚠️ General log can slow down MySQL if left on too long.

---

### ✅ Option 2: Use **Percona Audit Log Plugin**

Better for **production**, supports filtering and structured logs.

#### Step 1: Install Plugin (Percona Server or compatible MySQL)
```sql
INSTALL PLUGIN audit_log SONAME 'audit_log.so';
```

#### Step 2: Enable the plugin
```sql
SET GLOBAL audit_log_policy = ALL;
```

#### Step 3: Check the audit log file (default in `/var/lib/mysql/audit.log`)
You’ll see entries like:
```json
{
  "timestamp": "2025-04-20T09:02:45",
  "user": "myuser[root] @ localhost []",
  "query": "UPDATE accounts SET balance=0 WHERE user_id=42",
  ...
}
```

> Supports filtering by user, time, command, etc.

---

### ✅ Option 3: Use MySQL Enterprise Audit Plugin (If Using Enterprise)

If you're using **MySQL Enterprise**, there's a built-in plugin:
```sql
INSTALL PLUGIN audit_log SONAME 'audit_log.so';
SET GLOBAL audit_log_policy = ALL;
```

---

## 🎯 Use Case: Combining with Binlog

Once you find suspicious or target activity using logs or audit:
1. Note the timestamp
2. Extract the **matching binlog window** using:
   ```bash
   mysqlbinlog \
     --start-datetime="2025-04-20 09:00:00" \
     --stop-datetime="2025-04-20 09:10:00" \
     --base64-output=DECODE-ROWS --verbose \
     /var/lib/mysql/mysql-bin.000001 > changes.sql
   ```

3. Now you can replay or investigate the actual DML changes.

---

