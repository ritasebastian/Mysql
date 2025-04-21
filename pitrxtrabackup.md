Great! Since you're taking:

- ✅ **Full backup every Sunday**
- ✅ **Incremental backups daily (Mon–Sat)**
- ✅ On a **production system**

You absolutely **can perform Point-In-Time Recovery (PITR)** using XtraBackup + binlogs — but **you must prepare the backups** before restoring.

---

## 🧠 Why You Still Need `--prepare`

- XtraBackup writes **InnoDB redo logs** during backup
- **`--prepare` applies those logs** to bring the data to a consistent state
- Even for incrementals, you must apply them **on top of the full backup**, then prepare the final result before restoring

---

## ✅ Step-by-Step: PITR with Full + Incremental XtraBackup

### 🗓 Assume:
- Sunday: full backup → `/backup/full`
- Monday: incremental → `/backup/inc-mon`
- Tuesday: incremental → `/backup/inc-tue`
- Accident happened: **Tuesday 3:00 PM**
- You want to **restore up to Tuesday 3:00 PM**

---

### ✅ Step 1: Prepare Full Backup (DO NOT apply logs yet)

```bash
xtrabackup --prepare --apply-log-only --target-dir=/backup/full
```

---

### ✅ Step 2: Apply Monday Incremental

```bash
xtrabackup --prepare --apply-log-only \
  --target-dir=/backup/full \
  --incremental-dir=/backup/inc-mon
```

---

### ✅ Step 3: Apply Tuesday Incremental (Last Incremental)

```bash
xtrabackup --prepare \
  --target-dir=/backup/full \
  --incremental-dir=/backup/inc-tue
```

✅ The `--prepare` (without `--apply-log-only`) finalizes the backup and makes it ready for restore.

---

### ✅ Step 4: Restore the Final Backup

```bash
systemctl stop mysqld
mv /var/lib/mysql /var/lib/mysql.bak
xtrabackup --copy-back --target-dir=/backup/full
chown -R mysql:mysql /var/lib/mysql
systemctl start mysqld
```

---

### ✅ Step 5: Find Binlog Position

Check:
```bash
cat /backup/full/xtrabackup_binlog_info
```

Example output:
```
mysql-bin.000123    12345678
```

---

### ✅ Step 6: Apply Binlogs Up to Crash Time (e.g., 3:00 PM)

```bash
mysqlbinlog \
  --start-position=12345678 \
  --stop-datetime="2025-04-22 15:00:00" \
  /var/log/mysql/mysql-bin.000123 \
  /var/log/mysql/mysql-bin.000124 \
| mysql -u root -p
```

---

## ✅ Summary of PITR Steps with Full + Incrementals

| Step | Description                                  |
|------|----------------------------------------------|
| 1️⃣   | Prepare full backup with `--apply-log-only` |
| 2️⃣   | Apply incrementals with `--apply-log-only`  |
| 3️⃣   | Final `--prepare` to make data consistent    |
| 4️⃣   | Restore data to `datadir`                    |
| 5️⃣   | Read binlog position from backup             |
| 6️⃣   | Replay binlogs to exact recovery time        |

---
