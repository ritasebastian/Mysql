To **perform DDL or DML in MySQL replication** without it being replicated to the slave(s), you have a few options depending on your replication setup. Here's how you can **safely execute statements that are ignored by replication** 👇

---

## ✅ Option 1: Use `sql_log_bin = 0` (Session Level)

This is the **most common** and safest way:

### 🔧 Syntax:
```sql
SET sql_log_bin = 0;
-- Your DDL or DML here
ALTER TABLE my_table ADD COLUMN debug_info TEXT;
-- Or
UPDATE my_table SET debug_info = 'test' WHERE id = 1;
SET sql_log_bin = 1;
```

### ✅ What it does:
- Temporarily **disables binary logging** for that session.
- **Replicas will not see these changes**.
- Only works on the current connection/session.

---

## ⚠️ Important Notes

- You must **have SUPER privilege** (or `SESSION_VARIABLES_ADMIN` in MySQL 8+).
- Don’t use this for data that should be kept in sync.
- Useful for **debug-only columns**, local dev tweaks, temp fixes, etc.

---

## 🚫 Option 2: Use Replication Filters (Less Recommended)

You can configure replication to **ignore specific databases or tables** in `my.cnf`:

```ini
replicate_ignore_db = debug_db
replicate_ignore_table = mydb.temp_log
```

> Only works **from master to slave**, and not for GTID-based auto-positioning.

---

## 🧪 Example: Ignore a column add

```sql
SET sql_log_bin = 0;
ALTER TABLE orders ADD COLUMN temp_flag BOOLEAN DEFAULT FALSE;
SET sql_log_bin = 1;
```

➡️ This column will **only exist on the master**, and won’t replicate.

---
