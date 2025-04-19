Sure! Here's a complete and clean **MySQL Binary Installation & Backup Automation Guide** tailored for **Amazon Linux 2023 on AWS EC2**, including MySQL setup, Percona XtraBackup installation, and automated backup with cron.

---

# 🐬 MySQL 8.0 Binary Installation and Backup Automation on Amazon Linux 2023

## 🔧 Prerequisites

- EC2 instance with Amazon Linux 2023
- Internet access
- Root or `sudo` privileges

---

## ✅ 1. Install MySQL 8.0 from Binary Tarball

### 📥 Download and Extract

```bash
wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.36-linux-glibc2.28-x86_64.tar.xz
tar -xf mysql-8.0.36-linux-glibc2.28-x86_64.tar.xz
sudo mv mysql-8.0.36-linux-glibc2.28-x86_64 /usr/local/mysql
```

---

### 👥 Create MySQL User and Set Permissions

```bash
sudo groupadd mysql
sudo useradd -r -g mysql -s /bin/false mysql

cd /usr/local/mysql
sudo mkdir mysql-files
sudo chown -R mysql:mysql /usr/local/mysql
sudo chmod 750 mysql-files
```

---

### 📁 Initialize the MySQL Data Directory

```bash
sudo bin/mysqld --initialize --user=mysql
```

Note the temporary root password shown.

---

### 🚀 Start MySQL Manually (Initial Start)

```bash
sudo bin/mysqld_safe --user=mysql &
```

---

### 🔑 Secure MySQL

```bash
mysql -u root -p
```

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'dbaonly123';
```

---

### 🛠️ Add MySQL to PATH

```bash
echo 'export PATH=/usr/local/mysql/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

---

## ✅ 2. Set Up MySQL as a Systemd Service

### 📄 Create systemd Unit File

```bash
sudo vi /etc/systemd/system/mysql.service
```

Paste:

```ini
[Unit]
Description=MySQL Server
After=network.target

[Service]
Type=simple
User=mysql
Group=mysql
ExecStart=/usr/local/mysql/bin/mysqld --datadir=/usr/local/mysql/data
ExecStop=/usr/local/mysql/bin/mysqladmin shutdown
Restart=on-failure
LimitNOFILE=5000

[Install]
WantedBy=multi-user.target
```

---

### 🔄 Reload and Enable the Service

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable mysql
sudo systemctl start mysql
sudo systemctl status mysql
```

---

## ✅ 3. Install Percona XtraBackup

### 📥 Download & Extract

```bash
wget https://downloads.percona.com/downloads/Percona-XtraBackup-8.0/Percona-XtraBackup-8.0.35-30/binary/tarball/percona-xtrabackup-8.0.35-30-Linux-x86_64.glibc2.17.tar.gz
tar -xzf percona-xtrabackup-8.0.35-30-Linux-x86_64.glibc2.17.tar.gz
sudo mv percona-xtrabackup-8.0.35-30-Linux-x86_64.glibc2.17 /opt/xtrabackup
```

### 🛠️ Add to PATH

```bash
echo 'export PATH=$PATH:/opt/xtrabackup/bin' >> ~/.bashrc
source ~/.bashrc
```

### 🧪 Verify Installation

```bash
xtrabackup --version
```

---

## ✅ 4. Create Automated Backup Script

### 📄 Backup Script: `/usr/local/mysql/scripts/mysql_backup.sh`
```bash
sudo mkdir -p /usr/local/mysql/scripts
sudo chown mysql:mysql /usr/local/mysql/scripts
sudo vi /usr/local/mysql/scripts/mysql_backup.sh
```
```bash
#!/bin/bash

DATE=$(date +%F_%H-%M)
BACKUP_DIR="/backup/mysql/$DATE"
DATA_DIR="/usr/local/mysql/data"
LOG_FILE="/var/log/mysql_backup.log"
MYSQL_USER="root"
MYSQL_PASS="dbaonly123"
XTRABACKUP_BIN="/opt/xtrabackup/bin/xtrabackup"
SOCKET_PATH="/tmp/mysql.sock"

mkdir -p "$BACKUP_DIR"
echo "[$(date)] Starting backup..." >> "$LOG_FILE"

# Cleanup old backups
find /backup/mysql/* -maxdepth 0 -type d -mtime +7 -exec rm -rf {} \; >> "$LOG_FILE" 2>&1

"$XTRABACKUP_BIN" --backup \
  --target-dir="$BACKUP_DIR" \
  --datadir="$DATA_DIR" \
  --user="$MYSQL_USER" \
  --password="$MYSQL_PASS" \
  --socket="$SOCKET_PATH" >> "$LOG_FILE" 2>&1

if [ $? -eq 0 ]; then
  echo "[$(date)] Backup successful: $BACKUP_DIR" >> "$LOG_FILE"
else
  echo "[$(date)] Backup failed!" >> "$LOG_FILE"
fi
```

### 🧾 Make Executable

```bash
sudo chmod +x /usr/local/mysql/scripts/mysql_backup.sh
```

---

## 📅 5. Automate with Cron

Edit cron:

```bash
crontab -e
```

Add this line to back up every day at 2:30 AM:

```bash
30 2 * * * /usr/local/mysql/scripts/mysql_backup.sh
```

---

## ✅ 6. Manually Test the Backup

```bash
sudo /usr/local/mysql/scripts/mysql_backup.sh
cat /var/log/mysql_backup.log
ls /backup/mysql
```

---

## ✅ 7. Optional: Install Perl English Module (to suppress warning)

```bash
sudo yum install perl-English -y
```

---

## 🏁 Done!

You now have:
- ✅ MySQL installed manually from binaries
- ✅ systemd-controlled service
- ✅ Hot backups using Percona XtraBackup
- ✅ Automatic daily backups with retention

---
Perfect! Let’s update your setup to use **hostnames `db1` and `db2`** instead of IP addresses, using the `/etc/hosts` file on both EC2 instances.

---

## 🖥️ Step 0: Set Static Hostnames via `/etc/hosts`

### ✅ On **both db1 and db2**, add the following to `/etc/hosts`:

Edit with:

```bash
sudo vi /etc/hosts
```

Add lines:

```ini
192.168.1.10   db1
192.168.1.20   db2
```

> 📝 Replace `192.168.1.10` and `192.168.1.20` with the **actual private IPs** of your EC2 instances.

Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).

---

## 🛠️ Step 1: Configure `my.cnf` on **db1 (Primary)**
```bash
sudo vi /etc/my.cnf
```
```ini
[mysqld]
server-id=1
log-bin=mysql-bin
gtid_mode=ON
enforce-gtid-consistency=ON
binlog-format=ROW
log_slave_updates=ON
bind-address=0.0.0.0
```

Restart MySQL:

```bash
sudo systemctl restart mysql
```

---

## 👤 Step 2: Create Replication User on **db1**

Log into MySQL:

```bash
mysql -u root -p
```

Run:

```sql
CREATE USER 'repl'@'db2'
IDENTIFIED WITH mysql_native_password BY 'replpass';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'db2';
FLUSH PRIVILEGES;
```

---

## 🛠️ Step 3: Configure `my.cnf` on **db2 (Replica)**

```ini
[mysqld]
server-id=2
gtid_mode=ON
enforce-gtid-consistency=ON
binlog-format=ROW
log_slave_updates=ON
read_only=ON
```

Restart MySQL:

```bash
sudo systemctl restart mysql
```

---

## 🔍 Step 4: Get GTID Info from **db1**

```sql
SHOW MASTER STATUS\G
```

Note the `Executed_Gtid_Set` value (you'll need it in Step 5).

---

## 🔁 Step 5: Configure Replication on **db2**

Log into MySQL:

```bash
mysql -u root -p
```

Then run:

```sql
STOP SLAVE;

CHANGE MASTER TO
  MASTER_HOST='db1',
  MASTER_USER='repl',
  MASTER_PASSWORD='replpass',
  MASTER_AUTO_POSITION = 1;

START SLAVE;
```

---

## ✅ Step 6: Verify Replication on db2

```sql
SHOW SLAVE STATUS\G
```

Ensure:
- `Slave_IO_Running: Yes`
- `Slave_SQL_Running: Yes`
- `Seconds_Behind_Master: 0`

---

## 🧪 Step 7: Test Replication

On **db1**:

```sql
CREATE DATABASE testdb;

USE testdb;

CREATE TABLE employees (
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100),
  department VARCHAR(100),
  hire_date DATE
);

INSERT INTO employees (name, department, hire_date) VALUES
('Alice', 'Engineering', '2022-05-01'),
('Bob', 'Sales', '2021-11-12'),
('Charlie', 'HR', '2020-03-15');

```

On **db2**:

```sql
SHOW DATABASES;
```

You should see `testdb`.

---

## 📝 Summary

| Server | IP | Hostname | Role |
|--------|----|----------|------|
| EC2-1  | `192.168.1.10` | db1 | MySQL Primary (Master) |
| EC2-2  | `192.168.1.20` | db2 | MySQL Replica (Slave)  |

- Replication uses GTID for consistency and crash recovery
- `/etc/hosts` lets you reference `db1`/`db2` instead of IPs

---

