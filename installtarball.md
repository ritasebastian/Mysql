**clean and production-style MySQL installation using a tarball**, where:

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
| MySQL User     | `mysql` (non-login user)    |

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
sudo nano /etc/my.cnf
```

Paste:

```ini
[mysqld]
basedir=/opt/mysql
datadir=/data/mysql
socket=/tmp/mysql.sock
pid-file=/var/run/mysqld/mysqld.pid

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
