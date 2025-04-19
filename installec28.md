To install **MySQL using a binary tarball** on an **AWS EC2 instance** (like Amazon Linux or Ubuntu), follow the steps below. This is useful when you don’t want to use package managers like `yum` or `apt`, and want **more control** over the installation (e.g., for MySQL 5.7, 8.0, etc.).

---

### ✅ Step-by-Step: Install MySQL Binary on AWS EC2

#### 📌 1. **Download the MySQL Binary (Generic Linux tarball)**

Go to the official MySQL download site:
- https://dev.mysql.com/downloads/mysql/

Or use `wget` directly for MySQL 8.0 (example):

```bash
wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.36-linux-glibc2.28-x86_64.tar.xz
```

> 📝 Adjust version if needed.

---

#### 📌 2. **Extract and Move the Binary**

```bash
tar -xf mysql-8.0.36-linux-glibc2.28-x86_64.tar.xz
sudo mv mysql-8.0.36-linux-glibc2.28-x86_64 /usr/local/mysql
```

---

#### 📌 3. **Create MySQL User and Group**

```bash
sudo groupadd mysql
sudo useradd -r -g mysql -s /bin/false mysql
```

---

#### 📌 4. **Create Data Directory and Set Permissions**

```bash
cd /usr/local/mysql
sudo mkdir mysql-files
sudo chown mysql:mysql mysql-files
sudo chmod 750 mysql-files
```

---

#### 📌 5. **Initialize MySQL Database (Data Dir)**

```bash
sudo bin/mysqld --initialize --user=mysql
```

> It will output a temporary root password — note it down.

---

#### 📌 6. **Start MySQL Server**

```bash
sudo bin/mysqld_safe --user=mysql &
```

---

#### 📌 7. **Add MySQL to PATH**

```bash
echo 'export PATH=/usr/local/mysql/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

---

#### 📌 8. **Login and Secure**

```bash
mysql -u root -p
```

Then run:

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'YourNewSecurePass123!';
```

---

Perfect, let’s get you fully set up with:

1. ✅ A `mysql.service` file for `systemd`  
2. ✅ Commands to **start**, **stop**, and **enable** the MySQL service  
3. ✅ How to **install Percona XtraBackup** for hot backups

Let’s go step-by-step. This is tailored for a **binary MySQL installation** in `/usr/local/mysql`.

---

## ✅ 1. Create `mysql.service` for systemd

Create the service file:

```bash
sudo nano /etc/systemd/system/mysql.service
```

Paste the following content (edit paths if different):

```ini
[Unit]
Description=MySQL Server
After=network.target

[Service]
Type=forking
User=mysql
Group=mysql
ExecStart=/usr/local/mysql/bin/mysqld_safe --datadir=/usr/local/mysql/data
ExecStop=/usr/local/mysql/bin/mysqladmin shutdown
Restart=on-failure
LimitNOFILE = 5000

[Install]
WantedBy=multi-user.target
```

> ✅ Replace `/usr/local/mysql/data` if your datadir is different.

Save and close (`Ctrl+O`, `Enter`, then `Ctrl+X`).

---

## ✅ 2. Reload systemd & Start MySQL Service

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable mysql
sudo systemctl start mysql
```

You can check status with:

```bash
sudo systemctl status mysql
```

And to stop it:

```bash
sudo systemctl stop mysql
```

---

## ✅ 3. Install Percona XtraBackup

Percona provides XtraBackup binaries for CentOS, Amazon Linux, and Ubuntu.

### ➤ For Amazon Linux 2 / CentOS (most EC2 AMIs):

```bash
sudo yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm -y
sudo percona-release enable-only tools release
sudo yum install percona-xtrabackup-80 -y
```

### ➤ For Ubuntu:

```bash
wget https://repo.percona.com/apt/percona-release_latest.generic_all.deb
sudo dpkg -i percona-release_latest.generic_all.deb
sudo percona-release setup tools
sudo apt update
sudo apt install percona-xtrabackup-80 -y
```

---

## ✅ 4. Test XtraBackup

Example command to take a full backup:

```bash
sudo mkdir -p /backup/mysql
sudo xtrabackup --backup --target-dir=/backup/mysql --datadir=/usr/local/mysql/data --user=root --password=YourPassword
```

> Change `--datadir` and `--target-dir` paths if needed.

---
Awesome! Here's how to automate **Percona XtraBackup** using a cron job and a shell script.

---

## ✅ Step-by-Step XtraBackup Automation Setup

### 📁 1. **Create a Backup Script**

Create a new script:

```bash
sudo nano /usr/local/mysql/scripts/mysql_backup.sh
```

Paste this backup script:

```bash
#!/bin/bash

DATE=$(date +%F_%H-%M)
BACKUP_DIR="/backup/mysql/$DATE"
DATA_DIR="/usr/local/mysql/data"
LOG_FILE="/var/log/mysql_backup.log"
MYSQL_USER="root"
MYSQL_PASS="YourPassword"

mkdir -p "$BACKUP_DIR"

echo "[$(date)] Starting backup..." >> "$LOG_FILE"

/usr/bin/xtrabackup --backup \
  --target-dir="$BACKUP_DIR" \
  --datadir="$DATA_DIR" \
  --user="$MYSQL_USER" \
  --password="$MYSQL_PASS" >> "$LOG_FILE" 2>&1

if [ $? -eq 0 ]; then
  echo "[$(date)] Backup successful: $BACKUP_DIR" >> "$LOG_FILE"
else
  echo "[$(date)] Backup failed!" >> "$LOG_FILE"
fi
```

> Replace `YourPassword` with your actual MySQL root password.  
> You can also use `--defaults-file` if you prefer not to expose the password in the script.

Make it executable:

```bash
sudo chmod +x /usr/local/mysql/scripts/mysql_backup.sh
```

---

### 📅 2. **Schedule Cron Job**

Open your crontab:

```bash
crontab -e
```

Add this line to run the backup daily at 2:30 AM:

```bash
30 2 * * * /usr/local/mysql/scripts/mysql_backup.sh
```

> You can adjust the timing as needed (cron format is `min hour day month weekday`).

---

### 📂 3. **Optional: Clean Up Old Backups Automatically**

To delete backups older than 7 days, add this line to the script **before the `mkdir -p "$BACKUP_DIR"`** line:

```bash
find /backup/mysql/* -maxdepth 0 -type d -mtime +7 -exec rm -rf {} \;
```

---

### 📌 Bonus: Test Script Manually

You can manually test the backup:

```bash
sudo /usr/local/mysql/scripts/mysql_backup.sh
```

Then check logs:

```bash
cat /var/log/mysql_backup.log
```

---

Do you want to compress the backups (e.g., using `tar.gz`) or push them to S3/GCS for remote storage? I can help with that too!
