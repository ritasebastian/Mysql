## **Clean and production-style MySQL installation using a tarball**, where:

- ✅ MySQL **software is installed separately** (e.g., `/opt/mysql`)
- ✅ **Data**, **binlogs**, **slow logs**, **error logs**, and `my.cnf` are all placed in **custom directories**
- ✅ A dedicated **mysql UNIX user** is used
- ✅ The **PATH is updated**
- ✅ The install is clean and manageable for admin and backup purposes

---

# 🛠️ Step-by-Step: Install MySQL from Tarball with Custom Paths

---

## ✅ 1. Create Directory Layout

Choose your layout:

| Purpose        | Path                        |
|----------------|-----------------------------|
| Software       | `/opt/mysql`                |
| Data           | `/data/mysql`               |
| Logs (binlog, slow, error) | `/var/log/mysql`            |
| Config         | `/etc/my.cnf`               |
| MySQL User     | `mysql` (login user)    |

```bash
sudo mkdir -p /opt/mysql
sudo mkdir -p /data/mysql
sudo mkdir -p /var/log/mysql
sudo mkdir -p /var/run/mysqld
sudo mkdir -p /home/mysql
sudo groupadd mysql
sudo useradd -r -g mysql -s /bin/false -d /home/mysql mysql
```

---

## ✅ 2. Download and Extract Tarball

```bash
wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.36-linux-glibc2.28-x86_64.tar.xz
tar -xf mysql-8.0.36-linux-glibc2.28-x86_64.tar.xz
sudo mv mysql-8.0.36-linux-glibc2.28-x86_64/* /opt/mysql
```

---

## ✅ 3. Set Ownership and Permissions

```bash
sudo chown -R mysql:mysql /opt/mysql
sudo chown -R mysql:mysql /data/mysql
sudo chown -R mysql:mysql /var/log/mysql
sudo chown -R mysql:mysql /var/run/mysqld
sudo chown mysql:mysql /home/mysql
```

---

## ✅ 4. Create Custom `my.cnf`

```bash
sudo vi /etc/my.cnf
```

Paste:

```ini
[mysqld]
basedir=/opt/mysql
datadir=/data/mysql
socket=/tmp/mysql.sock
pid-file=/var/run/mysqld/mysqld.pid
innodb_dedicated_server=ON

log-error=/var/log/mysql/mysqld.err
slow_query_log=1
slow_query_log_file=/var/log/mysql/mysql-slow.log

server-id=1
log-bin=/var/log/mysql/mysql-bin
binlog_format=ROW
gtid_mode=ON
enforce-gtid-consistency=ON
log_slave_updates=ON

[client]
socket=/tmp/mysql.sock
```

---

## ✅ 5. Initialize MySQL Database

```bash
cd /opt/mysql
sudo bin/mysqld --defaults-file=/etc/my.cnf --initialize --user=mysql
```

This creates the initial database in `/data/mysql` and outputs a temporary root password.

---

## ✅ 6. Add to `PATH` (so you can run `mysql` from anywhere)

```bash
echo 'export PATH=$PATH:/opt/mysql/bin' >> ~/.bashrc
source ~/.bashrc
```

Or system-wide:

```bash
echo 'export PATH=$PATH:/opt/mysql/bin' | sudo tee /etc/profile.d/mysql.sh
source /etc/profile.d/mysql.sh
```

---

## ✅ 7. Create systemd Service (optional but recommended)

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
ExecStart=/opt/mysql/bin/mysqld --defaults-file=/etc/my.cnf
ExecStop=/opt/mysql/bin/mysqladmin shutdown
Restart=on-failure
LimitNOFILE=5000

[Install]
WantedBy=multi-user.target
```

Enable & start:

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable mysql
sudo systemctl start mysql
sudo systemctl status mysql
```

---

## ✅ 8. Secure MySQL

```bash
sudo su mysql
sudo cat /var/log/mysql/mysqld.err | grep 'temporary password'
/opt/mysql/bin/mysql_secure_installation
```

Or manually:

```bash
mysql -u root -p
ALTER USER 'root'@'localhost' IDENTIFIED BY 'YourStrongPass123!';
```

---

## ✅ 9. Verify Everything

```bash
mysql -u root -p -e "SHOW VARIABLES LIKE 'datadir';"
mysql -u root -p -e "SHOW VARIABLES LIKE 'log_bin%';"
ls /data/mysql
ls /var/log/mysql
```

---

## 📦 Done!

Now you have a **clean, custom-path MySQL** install with:

- Software in `/opt/mysql`
- Data in `/data/mysql`
- Logs in `/var/log/mysql`
- Config at `/etc/my.cnf`
- Controlled with systemd
- MySQL CLI available via `$PATH`

---


# 📦 Install and Automate Percona XtraBackup on Custom MySQL Install (Tarball)

---

## ✅ 1. Download & Install Percona XtraBackup

```bash
wget https://downloads.percona.com/downloads/Percona-XtraBackup-8.0/Percona-XtraBackup-8.0.35-30/binary/tarball/percona-xtrabackup-8.0.35-30-Linux-x86_64.glibc2.17.tar.gz
tar -xzf percona-xtrabackup-8.0.35-30-Linux-x86_64.glibc2.17.tar.gz
sudo mv percona-xtrabackup-8.0.35-30-Linux-x86_64.glibc2.17 /opt/xtrabackup
```

---

## 🛠️ 2. Add XtraBackup to PATH

```bash
echo 'export PATH=$PATH:/opt/xtrabackup/bin' >> ~/.bashrc
source ~/.bashrc
```

---

## 🧪 3. Verify Installation

```bash
xtrabackup --version
```

Expected output:

```bash
xtrabackup version 8.0.35-30 based on MySQL server 8.0.35
```

---

## 🛡️ 4. Create Backup Script

### 🔹 Create script directory & set ownership:

```bash
sudo mkdir -p /usr/local/mysql/scripts
sudo chown mysql:mysql /usr/local/mysql/scripts
```

### 🔹 Create script file:

```bash
sudo vi /usr/local/mysql/scripts/mysql_backup.sh
```

Paste:

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

# Cleanup old backups (older than 7 days)
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

---

## ✅ 5. Make Script Executable

```bash
sudo chmod +x /usr/local/mysql/scripts/mysql_backup.sh
```

---

## ⏰ 6. Automate Daily with Cron

Edit your crontab:

```bash
crontab -e
```

Add this line to schedule daily backups at **2:30 AM**:

```cron
30 2 * * * /usr/local/mysql/scripts/mysql_backup.sh
```

---

## 🧪 7. Manual Test

```bash
sudo /usr/local/mysql/scripts/mysql_backup.sh
cat /var/log/mysql_backup.log
ls /backup/mysql
```

You should see:
- A dated folder inside `/backup/mysql/`
- Log entries for "Backup successful"
- Dumped files like `xtrabackup_logfile`, `ibdata1`, etc.

---


