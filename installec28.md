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
sudo chown -R mysql:mysql /usr/local/mysql
sudo chown -R mysql:mysql /usr/local/mysql/data

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
ALTER USER 'root'@'localhost' IDENTIFIED BY 'dbaonly123';
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
sudo vi /etc/systemd/system/mysql.service
```

Paste the following content (edit paths if different):

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

Percona provides XtraBackup binaries for CentOS, Amazon Linux.



### ✅ Recommended Solution: Use the Latest Available Binary Tarball

As of now, the latest available version of Percona XtraBackup is **8.0.35-30**, which supports MySQL versions up to 8.0.36. You can download the binary tarball compatible with your system (glibc 2.17) using the following command:

```bash
wget https://downloads.percona.com/downloads/Percona-XtraBackup-8.0/Percona-XtraBackup-8.0.35-30/binary/tarball/percona-xtrabackup-8.0.35-30-Linux-x86_64.glibc2.17.tar.gz
```


This tarball is suitable for most modern Linux distributions, including Amazon Linux 2023.

### 📦 Installation Steps

1. **Extract the Tarball**:

   ```bash
   tar -xzf percona-xtrabackup-8.0.35-30-Linux-x86_64.glibc2.17.tar.gz
   ```


2. **Move to a Desired Location** (e.g., `/opt`):

   ```bash
   sudo mv percona-xtrabackup-8.0.35-30-Linux-x86_64.glibc2.17 /opt/xtrabackup
   ```


3. **Add to PATH**:

   Temporarily:

   ```bash
   export PATH=$PATH:/opt/xtrabackup/bin
   ```


   Permanently (add to `.bashrc` or `.bash_profile`):

   ```bash
   echo 'export PATH=$PATH:/opt/xtrabackup/bin' >> ~/.bashrc
   source ~/.bashr
   ```


4. **Verify Installation**:

   ```bash
   xtrabackup --version
   ```


   You should see output indicating the installed version, confirming a successful setup.


---

## ✅ 4. Test XtraBackup

Example command to take a full backup:

```bash
sudo mkdir -p /backup/mysql
sudo chown mysql:mysql /backup/mysql
sudo /opt/xtrabackup/bin/xtrabackup --backup \
  --target-dir=/backup/mysql \
  --datadir=/usr/local/mysql/data \
  --user=root \
  --password=dbaonly123 \
  --socket=/tmp/mysql.sock

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
MYSQL_PASS="dbaonly123"

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


