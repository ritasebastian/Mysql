Let’s now use these methods (✅ **SSH Pull** or ✅ **Netcat Push**) to **build a MySQL replica server** — a **read-only replica** of your production MySQL — using **XtraBackup + GTID/binlog position**, suitable for **replication**.

---

## 🧱 Goal: Create a **replica MySQL EC2 instance** from a live production EC2 using **XtraBackup** and either:

- 🅰️ **SSH Pull (Recommended for security)**
- 🅱️ **Netcat Push (Recommended for speed in trusted networks)**

---

## ✅ HIGH-LEVEL OVERVIEW

1. 🔧 Configure production MySQL for GTID and binlogs  
2. 📦 Take a full backup using `xtrabackup` (with GTID/binlog info)  
3. 🚚 Transfer backup to replica (via SSH pull or Netcat push)  
4. 🛠 Prepare & restore backup on replica  
5. 🔄 Configure replica `my.cnf` and `CHANGE MASTER TO`  
6. ✅ Start replication!

---

## 🔁 PREREQUISITES ON PROD

### In `my.cnf` on **prod**:
```ini
[mysqld]
server-id = 1
gtid_mode = ON
enforce-gtid-consistency = ON
log-bin = mysql-bin
log_slave_updates = ON
binlog_format = ROW
```

Then:
```bash
sudo systemctl restart mysqld
```

---

## ✅ STEP-BY-STEP: Build a Replica Using SSH Pull or Netcat Push

---

### 🔸 1. **Create replication user on PROD**
```sql
CREATE USER 'repl'@'%' IDENTIFIED BY 'StrongPassword';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
FLUSH PRIVILEGES;
```

---

### 🔸 2. **On UAT/Replica — Take Backup from PROD**

### 🅰️ Option 1: SSH Pull Method ✅
```bash
ssh ec2-user@prod-db "sudo xtrabackup --backup --stream=xbstream --compress --compress-threads=4" \
  > /tmp/prod-backup.xbstream
```

Then:
```bash
cd /tmp
mkdir -p prod-restore
cd prod-restore
xbstream -x < ../prod-backup.xbstream
find . -name "*.qp" -exec qpress -d {} \;
xtrabackup --prepare --target-dir=/tmp/prod-restore
```

### 🅱️ Option 2: Netcat Push (from PROD)
> On **replica** (listener):
```bash
mkdir -p /tmp/prod-restore
cd /tmp/prod-restore
nc -l -p 9999 | xbstream -x
```

> On **prod**:
```bash
xtrabackup --backup --stream=xbstream --compress --compress-threads=4 \
  | nc uat-hostname 9999
```

Then extract and prepare as above.

---

### 🔸 3. **Get GTID Info (Required for Replication)**

From the backup directory:
```bash
cat /tmp/prod-restore/xtrabackup_binlog_info
cat /tmp/prod-restore/xtrabackup_info
```

You'll see something like:
```
binlog_pos = filename 'mysql-bin.000012', position 4561234
GTID = a1111b22-uuid:1-2049
```

✅ Save this info!

---

### 🔸 4. **Restore Backup to Replica MySQL Datadir**

```bash
sudo systemctl stop mysqld
mv /var/lib/mysql /var/lib/mysql.bak
xtrabackup --copy-back --target-dir=/tmp/prod-restore
chown -R mysql:mysql /var/lib/mysql
sudo systemctl start mysqld
```

---

### 🔸 5. **Configure Replica MySQL (in `my.cnf`)**

```ini
[mysqld]
server-id = 2
gtid_mode = ON
enforce-gtid-consistency = ON
log-bin = mysql-bin
log_slave_updates = ON
relay_log = relay-bin
read_only = ON
```

Restart:
```bash
sudo systemctl restart mysqld
```

---

### 🔸 6. **Point Replica to Production Master**

Run on the replica server:

```sql
RESET MASTER;

CHANGE MASTER TO
  MASTER_HOST='prod-db-private-ip',
  MASTER_USER='repl',
  MASTER_PASSWORD='StrongPassword',
  MASTER_AUTO_POSITION=1,
  MASTER_PORT=3306;

START REPLICA;
```

> ✅ If using MySQL 5.7 or 8.0+, `MASTER_AUTO_POSITION=1` uses GTID.

---

### 🔸 7. **Verify Replication**

```sql
SHOW REPLICA STATUS\G
```

You should see:
- `Slave_IO_Running: Yes`
- `Slave_SQL_Running: Yes`
- No errors

---

