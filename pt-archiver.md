

## 🛠️ What is `pt-archiver`?

**`pt-archiver`** is a **Percona Toolkit command-line tool** used to:

> ✅ **Archive**, **purge**, or **move** large amounts of MySQL data from one table to another (or to a file) **without locking the database or slowing down production.**

---

## ✅ Key Use Cases

| Use Case | Description |
|----------|-------------|
| 🔁 Archive old data | Move records from `main_table` to `archive_table` |
| 🧹 Purge expired data | Delete rows based on time or flag condition |
| 💾 Export to file     | Save rows to a CSV or TSV for backup/logging |
| 🚀 Avoid large DELETEs | Prevent full table locks on big deletions |

---

## 💡 Why Not Just Use `DELETE FROM table WHERE created_at < '2023'`?

Because:
- It can **lock the table** during execution 😱
- May **slow down replication**
- Causes **undo log bloat**
- Affects **performance for other users**

🧠 `pt-archiver` solves this by:
- Deleting/archiving in **small chunks**
- Respecting **replication lag**
- Supporting **throttling**
- Adding **pause/resume and dry-run modes**

---

## 🔧 Example Commands

### ✅ 1. Archive old rows to another table:

```bash
pt-archiver \
  --source h=127.0.0.1,D=mydb,t=orders,u=root,p=pass \
  --dest h=127.0.0.1,D=mydb,t=orders_archive,u=root,p=pass \
  --where "created_at < '2023-01-01'" \
  --limit=1000 --commit-each --no-check-charset
```

### ✅ 2. Purge (DELETE) old rows in chunks:

```bash
pt-archiver \
  --source h=127.0.0.1,D=mydb,t=logs,u=root,p=pass \
  --where "log_date < '2023-01-01'" \
  --purge --limit=500 --commit-each
```

### ✅ 3. Export rows to a file:

```bash
pt-archiver \
  --source h=127.0.0.1,D=mydb,t=users,u=root,p=pass \
  --where "status='inactive'" \
  --file /tmp/inactive_users.csv --fields-terminated-by=',' \
  --limit=1000
```

---

## 🧰 Popular Options

| Option | Purpose |
|--------|---------|
| `--source` / `--dest` | Define source and destination DB/table |
| `--where` | Filtering rows |
| `--purge` | Delete rows after archiving |
| `--file` | Output to file instead of DB |
| `--limit` | Number of rows per batch |
| `--commit-each` | Commit after every batch |
| `--sleep` | Throttle to reduce DB load |
| `--progress` | Show progress bar |

---

## ✅ Best Practices

| Tip | Why |
|-----|-----|
| Use `--limit` (500–1000) | Prevent table lock |
| Use `--sleep` (0.1s+) | Avoid performance spikes |
| Run during off-peak hours | Reduce user impact |
| Always test with `--dry-run` first | Safe validation |
| Use on **indexed** columns | Prevent full table scans |

---

## 🚧 Caveats

| Limitation | Notes |
|------------|-------|
| Requires Perl | Installed with Percona Toolkit |
| Won’t work on Cloud SQL (directly) | Run from GCE VM or with proxy |
| Must have access to binlogs if lag-sensitive | For production safety |

---

## 🔥 When to Use `pt-archiver`

- Cleaning up logs, sessions, metrics, notifications, etc.
- Retaining last N days/months of data
- Automating purge/archive tasks via cron or pipeline
- Avoiding `DELETE FROM ... LIMIT 1000` loops

---

## 🧪 Sample Use Case

**Purge audit logs older than 6 months:**

```bash
pt-archiver \
  --source h=localhost,D=app,t=audit_logs,u=root,p=pass \
  --where "event_time < NOW() - INTERVAL 6 MONTH" \
  --purge --limit=1000 --commit-each --sleep=0.5
```

---
