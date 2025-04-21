`pt-query-digest` is a powerful command-line tool from **Percona Toolkit** used to **analyze MySQL slow query logs**, general logs, or binary logs. It helps you **identify slow, frequent, or resource-intensive queries** that you should optimize.

---

### 🧠 What It Does
- Groups similar queries together (normalized)
- Shows metrics like:
  - Count
  - Exec time
  - Rows examined
  - Lock time
- Helps find the **top slowest queries** and **where to index or optimize**

---

### ✅ When to Use It
- After enabling **`slow_query_log`**
- Before performance tuning or audits
- For identifying worst queries on production

---

### 📦 Install `pt-query-digest`

Absolutely! Here's a clean, tested set of **right steps to install `pt-query-digest`** (from Percona Toolkit) on **Amazon Linux 2023** — including all dependencies.

---

## ✅ How to Install `pt-query-digest` on Amazon Linux 2023 (Step-by-Step)

---

### 🔧 Step 1: Enable Percona Repository

```bash
sudo yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
sudo percona-release enable tools release
sudo yum clean all
```

---

### 📦 Step 2: Download Percona Toolkit Manually (since it's not in repo yet)

```bash
wget https://repo.percona.com/yum/release/8/RPMS/x86_64/percona-toolkit-3.6.0-1.el8.x86_64.rpm
```

> If this fails with 404, search latest version here:  
> [https://repo.percona.com/yum/release/8/RPMS/x86_64/](https://repo.percona.com/yum/release/8/RPMS/x86_64/)

---

### 🧩 Step 3: Install Required Perl Modules

```bash
sudo yum install \
  perl-English \
  perl-Sys-Hostname \
  perl-List-MoreUtils \
  perl-Time-HiRes \
  perl-IO-Socket-SSL \
  perl-Digest-MD5 \
  perl-TermReadKey \
  perl-Module-Load \
  perl-DBI \
  perl-DBD-MySQL
```

---

### 🛠 Step 4: Install the Toolkit RPM

```bash
sudo yum install ./percona-toolkit-3.6.0-1.el8.x86_64.rpm
```

---

### ✅ Step 5: Verify Installation

```bash
pt-query-digest --version
```

Expected output:
```bash
pt-query-digest 3.6.0
```

---

### 🧪 Bonus: Test With mysqlbinlog

Example:

```bash
sudo /opt/mysql/bin/mysqlbinlog /var/log/mysql/mysql-bin.000003 \
| pt-query-digest --type=binlog
```

---


### 📝 Sample Usage

#### Step 1: Enable Slow Query Logging in MySQL

```sql
SET GLOBAL slow_query_log = 1;
SET GLOBAL long_query_time = 1;  -- log queries taking more than 1 second
SHOW VARIABLES LIKE 'slow_query_log_file';
```

> MySQL will now write slow queries to the slow log file.

---

#### Step 2: Run pt-query-digest

```bash
pt-query-digest /var/log/mysql/mysql-slow.log
```

#### 🔍 Sample Output:
```
# 12.3s user time, 40ms system time, 12.34M rss, 32.00M vsz
# Profile
# Rank Query ID           Response time Calls R/Call V/M   Item
# ==== ================== ============= ===== ====== ===== ============
#    1 0xA1B2C3D4E5F67890 5.2100 71.23%     8 0.6512  0.98 SELECT users
#    2 0xBBBBBBBBBBBBBBBB 1.4210 19.42%    10 0.1421  1.00 SELECT orders
#    3 0xCCCCCCCCCCCCCCCC 0.6882  9.35%     6 0.1147  1.00 UPDATE cart
```

It will also show the **normalized query**, for example:

```sql
# Query 1: 0xA1B2C3D4E5F67890
# SELECT * FROM users WHERE email = ?;
```

---

### 🧪 Advanced Options

- Only queries over 5 seconds:
  ```bash
  pt-query-digest --filter '$event->{Query_time} > 5' /var/log/mysql/mysql-slow.log
  ```

- Output to HTML:
  ```bash
  pt-query-digest /var/log/mysql/mysql-slow.log --output=html > report.html
  ```

---

### 🚀 Summary

| Tool              | Use For                              |
|-------------------|---------------------------------------|
| `pt-query-digest` | Analyzing slow queries and tuning     |
| Source            | Slow logs / general logs / binlogs    |
| Output            | Ranked query summary with metrics     |

---

