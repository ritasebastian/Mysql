
---

## ✅ **What You Can Do with `mysqlbinlog` (with Examples)**

---

### 1. 🧾 **Read the contents of a binlog**

```bash
mysqlbinlog /var/lib/mysql/mysql-bin.000001
```

✔️ Dumps the raw SQL events from the binlog file in human-readable form.

---

### 2. 🕒 **Filter by time: Start and Stop datetime**

```bash
mysqlbinlog --start-datetime="2025-04-18 10:00:00" \
            --stop-datetime="2025-04-18 11:00:00" \
            mysql-bin.000001
```

✔️ Only dumps events between 10:00 and 11:00 on that day.

---

### 3. 🔢 **Filter by log position**

```bash
mysqlbinlog --start-position=120 --stop-position=999 \
            mysql-bin.000001
```

✔️ Dumps events within specific byte positions.

---

### 4. 🗃️ **Filter by database**

```bash
mysqlbinlog --database=mydb mysql-bin.000001
```

✔️ Dumps events only related to `mydb` database.

---

### 5. 🔁 **Replay the binlog into a live MySQL server**

```bash
mysqlbinlog mysql-bin.000001 | mysql -u root -p
```

✔️ Replays all events from the binlog into the database (useful for PITR).

---

### 6. 📝 **Export binlog to a SQL file**

```bash
mysqlbinlog mysql-bin.000001 > restore.sql
```

✔️ Saves the contents of the binlog to a file for review or later execution.

---

### 7. 📌 **Recover after accidental `DELETE`/`DROP`**

```bash
mysqlbinlog --start-datetime="2025-04-18 10:00:00" \
            --stop-datetime="2025-04-18 10:45:00" \
            mysql-bin.000001 | grep -v "DROP TABLE" | mysql -u root -p
```

✔️ Recovers data by excluding the destructive statements.

---

### 8. 🛠️ **Verbose decode of row-based format**

```bash
mysqlbinlog --base64-output=DECODE-ROWS --verbose mysql-bin.000002
```

✔️ Decodes `ROW`-based events (e.g., `UPDATE` on specific fields) into readable SQL.

---

### 9. 📂 **Process multiple binlogs at once**

```bash
mysqlbinlog mysql-bin.000001 mysql-bin.000002 | mysql -u root -p
```

✔️ Replays multiple binlogs in a chain — helpful after restoring a backup.

---

### 10. 🔍 **Search for specific actions (DDL/DML)**

```bash
mysqlbinlog mysql-bin.000001 | grep -E "CREATE|ALTER|DROP|UPDATE|DELETE"
```

✔️ Useful for auditing what schema or data changes were made.

---

### 11. 🔐 **Use remote binlog dump (with `--read-from-remote-server`)**

```bash
mysqlbinlog --read-from-remote-server \
            --host=your_host \
            --user=repl_user \
            --password \
            --raw --stop-never \
            mysql-bin.000001
```

✔️ Continuously stream binlog entries from a remote server (like a replica does).

> 🛑 Requires enabling `binlog_row_image` and user with `REPLICATION SLAVE` privilege.

---

### 12. 🧪 **Dry run without applying**

```bash
mysqlbinlog mysql-bin.000001 > changes.sql
less changes.sql
```

✔️ Review changes before applying.

---

## 📋 Summary Table

| Task | Command |
|------|---------|
| Dump all events | `mysqlbinlog mysql-bin.000001` |
| Filter by time | `--start-datetime`, `--stop-datetime` |
| Filter by DB | `--database=mydb` |
| Replay events | `| mysql -u root -p` |
| Export to SQL | `> restore.sql` |
| Decode row format | `--base64-output=DECODE-ROWS --verbose` |
| Multi-binlog input | `mysqlbinlog *.0000*` |
| Remote read | `--read-from-remote-server` |

---
