Let's walk through how to **refresh a non-production MySQL EC2 instance from a running production EC2 instance** using:

- ✅ **XtraBackup**
- ✅ **`nc` (netcat)** for fast network transfer
- ✅ **No downtime on production**
- ✅ **Compressed backup streaming**
- ✅ Fully AWS EC2-friendly

---

## ✅ Architecture

| Role        | EC2 Hostname       | MySQL Role        |
|-------------|--------------------|-------------------|
| Source      | `prod-db`          | Production MySQL  |
| Destination | `uat-db`           | Non-Production    |

---

## 🔐 Requirements

1. **XtraBackup** and `nc` installed on both instances
2. **Port 9999** open on `uat-db` (via security group)
3. `mysqld` stopped on `uat-db` during restore
4. MySQL configured identically (same version preferred)

---

## ✅ Step-by-Step Guide: XtraBackup + Netcat + Refresh

---

### 🔹 **Step 1: Open netcat listener on UAT (receiver)**

SSH into `uat-db` and run:

```bash
sudo systemctl stop mysqld
mkdir -p /tmp/prod-restore
cd /tmp/prod-restore

nc -l -p 9999 | xbstream -x
```

This:
- Listens on port 9999
- Extracts the incoming backup stream into `/tmp/prod-restore`

Leave this running. ✅

---

### 🔹 **Step 2: Run XtraBackup stream from PROD (sender)**

SSH into `prod-db` and run:

```bash
xtrabackup --backup --stream=xbstream \
  --target-dir=/tmp \
  | nc uat-db 9999
```

> Replace `uat-db` with the **private IP or hostname** of your non-prod EC2.

This:
- Starts XtraBackup (non-locking)
- Streams raw backup via `xbstream`
- Sends it to `uat-db:9999`

✅ Production stays up the entire time.

---

### 🔹 **Step 3: On UAT — decompress (if compressed used)**

If you had used `--compress`, now decompress:

```bash
find . -name "*.qp" -exec qpress -d {} \;
```

Skip this step if not using `--compress`.

---

### 🔹 **Step 4: Prepare the backup**

```bash
xtrabackup --prepare --target-dir=/tmp/prod-restore
```

---

### 🔹 **Step 5: Copy restored files to datadir**

```bash
mv /var/lib/mysql /var/lib/mysql.bak
xtrabackup --copy-back --target-dir=/tmp/prod-restore
chown -R mysql:mysql /var/lib/mysql
```

---

### 🔹 **Step 6: Start MySQL on UAT**

```bash
sudo systemctl start mysqld
```

---

### 🔹 **Step 7: Cleanup (optional)**

```bash
rm -rf /tmp/prod-restore
```

---

## ✅ AWS-Specific Notes

- ✅ Use **private IPs** between EC2s for faster and secure transfer
- ✅ Security group on `uat-db` must allow **port 9999 TCP**
- ✅ Run everything under a user with `sudo` access
- ✅ `nc` is often pre-installed, or install via:
  ```bash
  sudo yum install nc
  ```

---

## 🧾 Summary of Commands (Prod → UAT via netcat)

| Host      | Command                                                                 |
|-----------|-------------------------------------------------------------------------|
| **UAT**   | `nc -l -p 9999 | xbstream -x -C /tmp/prod-restore`                     |
| **Prod**  | `xtrabackup --backup --stream=xbstream | nc uat-db-private-ip 9999`     |
| **UAT**   | `xtrabackup --prepare`, `--copy-back`, `chown`, `start mysqld`         |

---


To run PITR or daily refresh jobs easily?
