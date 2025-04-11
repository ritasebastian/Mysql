
---

## 📘 **MySQL Files & Their Usage Cookbook**

| File Type            | Filename/Extension             | Description / Usage                                                                 |
|----------------------|--------------------------------|--------------------------------------------------------------------------------------|
| 🔧 Config File        | `/etc/my.cnf`, `/etc/mysql/my.cnf` | Main MySQL configuration file (ports, buffers, engine settings, etc.)              |
| 🗃️ Data Directory     | `/var/lib/mysql/`              | Default directory for databases and table storage (.ibd, .frm, etc.)               |
| 🧾 Error Log          | `hostname.err`                 | Logs MySQL startup/shutdown events and errors                                      |
| 📝 General Query Log  | `hostname.log`                 | Logs every query (disabled by default; for debugging)                              |
| 📊 Slow Query Log     | `hostname-slow.log`            | Logs queries that exceed `long_query_time` threshold                               |
| 🔁 Binary Logs        | `mysql-bin.000001`, etc.       | Logs all DML changes for replication and point-in-time recovery (PITR)             |
| 🧩 Relay Logs         | `relay-log.000001`, etc.       | Logs received from the primary during replication                                  |
| 🧪 Log Index Files    | `mysql-bin.index`, `relay-log.index` | Index of binlog or relay log files (used by MySQL to know file order)             |
| 💽 InnoDB Log Files   | `ib_logfile0`, `ib_logfile1`   | InnoDB redo log files for crash recovery                                           |
| 📦 InnoDB System Table | `ibdata1`                      | InnoDB system tablespace (may contain undo/rollback, dictionary info)              |
| 📁 Table Files        | `*.frm`, `*.ibd`, `*.MYD`, `*.MYI` | `.frm`: Table structure<br>`.ibd`: InnoDB table data<br>`.MYD`: MyISAM data<br>`.MYI`: MyISAM index |
| 🔐 SSL Certificates   | `server-cert.pem`, `ca-cert.pem` | Used for enabling SSL connections                                                  |

---

### 🔍 Typical Locations (Linux-based MySQL):

| File                  | Default Path                  |
|-----------------------|-------------------------------|
| `my.cnf`              | `/etc/mysql/my.cnf` or `/etc/my.cnf` |
| Data Directory        | `/var/lib/mysql/`             |
| Logs (if enabled)     | `/var/log/mysql/` or within data dir |
| SSL Certs             | `/var/lib/mysql/private/`     |

---

### 🛠️ Key `my.cnf` Settings to Control These Files

```ini
[mysqld]
datadir = /var/lib/mysql
log_error = /var/log/mysql/error.log
slow_query_log = 1
slow_query_log_file = /var/log/mysql/mysql-slow.log
general_log = 0
general_log_file = /var/log/mysql/mysql.log
log_bin = /var/log/mysql/mysql-bin
relay_log = /var/log/mysql/relay-log
```

---

## ✅ Use Cases for Each File Type

| Use Case                         | File(s) Involved                           |
|----------------------------------|--------------------------------------------|
| Diagnose server crash            | `*.err`, `ib_logfile*`                     |
| Enable replication               | `mysql-bin.*`, `relay-log.*`, `*.index`    |
| Audit SQL activity               | `general log`, `slow query log`            |
| Point-in-time recovery (PITR)    | `mysql-bin.*` + binary log index           |
| Tuning InnoDB crash recovery     | `ib_logfile*`, `ibdata1`, `*.ibd`          |
| Table migration (transportable)  | `*.ibd`, `*.cfg`, `*.frm`                  |

---

## ⚠️ Tips & Best Practices

- 🔐 **Restrict access** to `.cnf`, binary logs, and error logs (`chmod 600`)
- 🧼 Rotate logs using `logrotate`
- 🔁 Enable `log_bin` for replication + PITR
- 🧵 Monitor file sizes (especially `ibdata1` if `innodb_file_per_table = OFF`)
- 📦 Keep backups of both **data files** and **binlogs**

---


## 📂 **MySQL File Structure Tree (InnoDB)**

```
/var/lib/mysql/
│
├── ibdata1                    # InnoDB system tablespace (dictionary, undo, etc.)
├── ib_logfile0                # InnoDB redo log file
├── ib_logfile1
├── mysql/                    # System schema (user accounts, privileges, etc.)
│   ├── user.ibd
│   ├── db.ibd
│   └── ...
├── performance_schema/       # Performance monitoring schema
├── sys/                      # Views built on performance_schema
├── your_db/                  # Your user-created database (e.g., myapp)
│   ├── sales.frm             # Table definition (MySQL 5.x)
│   ├── sales.ibd             # InnoDB table data (if file-per-table is ON)
│   ├── sales.cfg             # InnoDB table metadata (transportable tablespace)
│   ├── customers.frm
│   ├── customers.ibd
│   └── ...
├── hostname.err              # Error log
├── mysql-bin.000001          # Binary log (for replication, PITR)
├── mysql-bin.000002
├── mysql-bin.index           # Index of all binary log files
├── relay-log.000001          # Relay log (if acting as a replica)
├── relay-log.index
└── auto.cnf                  # Server UUID (used in replication)
```

---

## 🧠 Key Concepts

| File/Folder            | Purpose                                         |
|------------------------|-------------------------------------------------|
| `ibdata1`              | Shared tablespace for system/undo info          |
| `ib_logfile*`          | Redo logs used for crash recovery               |
| `*.ibd`                | Table data files (1 per table if `innodb_file_per_table=ON`) |
| `*.frm` (MySQL 5.x)    | Table definition (structure)                    |
| `*.cfg`                | Table metadata (optional, for transportable tablespaces) |
| `mysql/`               | Internal user/privilege tables                  |
| `mysql-bin.*`          | Binary logs used in replication and PITR       |
| `relay-log.*`          | Logs from primary if the server is a replica   |
| `auto.cnf`             | Server’s unique ID (important for replication) |

---

### 🛡️ Quick Best Practices

- Backup: `*.ibd + *.cfg + *.frm` for transportable tables
- Monitor: `mysql-bin.*` and `ib_logfile*` sizes regularly
- Replication: Make sure `mysql-bin` is enabled and readable

---
