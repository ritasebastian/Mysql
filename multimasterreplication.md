Setting up **MySQL Multi-Master Replication** (Active-Active) means two or more MySQL servers act as both master and slave to each other. Changes in one server are replicated to the other(s), allowing for high availability and write/read from multiple nodes. Here's a **step-by-step setup for 2-node multi-master replication** using **MySQL 8.0+**.

---

## 🛠️ Prerequisites

- Two MySQL servers: `mysql1` and `mysql2`
- Each must have a unique `server-id`
- Enable binary logging
- Network access between nodes (e.g., port 3306 open)

---

## ✅ Step-by-Step Setup

### **Step 1: Configure `mysql1`**

Edit `my.cnf` or `my.ini` (location varies: `/etc/mysql/my.cnf` or `/etc/my.cnf`):

```ini
[mysqld]
server-id=1
log_bin=mysql-bin
binlog_format=ROW
gtid_mode=ON
enforce_gtid_consistency=ON
log_slave_updates=ON
read_only=OFF
```

> Restart MySQL after editing:
```bash
sudo systemctl restart mysql
```

---

### **Step 2: Configure `mysql2`**

```ini
[mysqld]
server-id=2
log_bin=mysql-bin
binlog_format=ROW
gtid_mode=ON
enforce_gtid_consistency=ON
log_slave_updates=ON
read_only=OFF
```

> Restart MySQL:
```bash
sudo systemctl restart mysql
```

---

### **Step 3: Create Replication User on Both Servers**

**On `mysql1`:**
```sql
CREATE USER 'repl'@'%' IDENTIFIED WITH mysql_native_password BY 'repl_pass';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
FLUSH PRIVILEGES;
```

**On `mysql2`:** (do the same)
```sql
CREATE USER 'repl'@'%' IDENTIFIED WITH mysql_native_password BY 'repl_pass';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
FLUSH PRIVILEGES;
```

---

### **Step 4: Get GTID Info from Both Servers**

On each server:
```sql
SHOW MASTER STATUS\G
```
> Note down `File` and `Position`, or use `GTID_EXECUTED`

---

### **Step 5: Configure Replication**

**On `mysql1` to replicate from `mysql2`:**
```sql
CHANGE REPLICATION SOURCE TO
  SOURCE_HOST='mysql2-ip',
  SOURCE_USER='repl',
  SOURCE_PASSWORD='repl_pass',
  SOURCE_AUTO_POSITION=1;
START REPLICA;
```

**On `mysql2` to replicate from `mysql1`:**
```sql
CHANGE REPLICATION SOURCE TO
  SOURCE_HOST='mysql1-ip',
  SOURCE_USER='repl',
  SOURCE_PASSWORD='repl_pass',
  SOURCE_AUTO_POSITION=1;
START REPLICA;
```

---

### **Step 6: Verify Status**

On both servers:
```sql
SHOW REPLICA STATUS\G
```
Check:
- `Replica_IO_Running: Yes`
- `Replica_SQL_Running: Yes`

---

## ⚠️ Best Practices

- Avoid auto-increment conflicts (set offsets):
```ini
auto_increment_increment = 2
auto_increment_offset = 1  # On mysql1
auto_increment_offset = 2  # On mysql2
```
- Enable conflict resolution in app logic if needed.
- Consider tools like **MySQL Group Replication** or **Galera** for production.

---
