

# 🧾 Cloud SQL for MySQL – Parameter Cheat Sheet

> 🧠 Use `gcloud sql instances patch` or **Cloud Console → Edit Flags** to modify these flags.

---

## 🔧 **Memory & Buffer Pool Tuning**

| Parameter                     | Recommended Value                  | Purpose                                             |
|-------------------------------|------------------------------------|-----------------------------------------------------|
| `innodb_buffer_pool_size`     | 60–70% of total RAM                | Caches table and index data (crucial for performance) |
| `innodb_log_buffer_size`      | 8M–64M                             | Buffer size for transactions before disk write     |
| `query_cache_type`            | 0 (disabled)                       | Disabled in MySQL 5.7+ (use app caching)           |
| `query_cache_size`            | 0                                  | Deprecated – set to 0 to avoid overhead             |

---

## 🧵 **Concurrency & Connections**

| Parameter                     | Recommended Value                  | Purpose                                             |
|-------------------------------|------------------------------------|-----------------------------------------------------|
| `max_connections`             | 250–1000 (based on app needs)      | Max client connections                              |
| `wait_timeout`                | 60–600                             | Idle connection timeout (shorter = better for pool) |
| `interactive_timeout`         | Same as `wait_timeout`             | Same timeout for interactive sessions               |
| `thread_cache_size`           | 8–100                              | Cache re-usable threads                             |

---

## ⚡ **Performance Tuning**

| Parameter                     | Recommended Value                  | Purpose                                             |
|-------------------------------|------------------------------------|-----------------------------------------------------|
| `slow_query_log`              | `ON`                               | Enable slow query logging                           |
| `long_query_time`             | `1–5`                              | Queries slower than this are logged (in seconds)   |
| `log_output`                  | `TABLE` or `FILE`                  | Store logs in table (default) or log file          |
| `log_queries_not_using_indexes` | `ON`                           | Detect unoptimized queries                         |

---

## 🔐 **Security & Logging**

| Parameter                     | Recommended Value                  | Purpose                                             |
|-------------------------------|------------------------------------|-----------------------------------------------------|
| `log_bin_trust_function_creators` | `ON`                         | Allow non-super users to create functions          |
| `log_error_verbosity`         | `2` or `3`                         | Verbosity level for error logs                     |

---

## 🔄 **Replication / PITR Settings**

| Parameter                     | Recommended Value                  | Purpose                                             |
|-------------------------------|------------------------------------|-----------------------------------------------------|
| `binlog_format`               | `ROW`                              | Required for replication and PITR                  |
| `binlog_expire_logs_seconds` | `86400` (1 day) to `604800` (7 days)| How long to keep binary logs                       |
| `gtid_mode`                   | `ON`                               | Required for managed replication                   |
| `enforce_gtid_consistency`   | `ON`                               | GTID-compatible transactions only                  |

---

## 🗃️ **Storage and Temp Tables**

| Parameter                     | Recommended Value                  | Purpose                                             |
|-------------------------------|------------------------------------|-----------------------------------------------------|
| `tmp_table_size`              | `64M–256M`                         | Memory used for internal temp tables               |
| `max_heap_table_size`         | Same as `tmp_table_size`           | Max size for user-created memory tables            |

---

## 🧪 **Monitoring & Debugging**

| Parameter                     | Recommended Value                  | Purpose                                             |
|-------------------------------|------------------------------------|-----------------------------------------------------|
| `performance_schema`          | `ON`                               | Enable built-in query and performance monitoring   |
| `general_log`                 | `OFF` (ON for debugging only)      | Logs every query – use with caution (slow + large) |

---

## 🚫 Parameters NOT Supported in Cloud SQL

Cloud SQL restricts access to certain **dangerous or OS-level flags**, including:
- `innodb_file_per_table` (enabled by default)
- `super_read_only`, `skip-grant-tables`, etc.
- `audit_log` plugin (not allowed in managed Cloud SQL)

---

### 🔍 How to View/Edit Parameters

- **Console**:  
  SQL → Instance → Edit → Flags  
- **CLI**:
```bash
gcloud sql instances patch INSTANCE_NAME \
  --database-flags=innodb_buffer_pool_size=1073741824,max_connections=250
```

---

## 💡 Pro Tips

- Always **test flag changes** on a replica or dev instance before applying to production.
- Changing some flags **requires a restart** of the Cloud SQL instance.
- Monitor the effects via **Query Insights** and **Cloud Monitoring**.

---
