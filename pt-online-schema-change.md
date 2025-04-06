

## 🧭 First: What kind of GCP MySQL environment are you using?

| GCP Setup              | Supports pt-online-schema-change? | Notes |
|------------------------|-----------------------------------|-------|
| ✅ MySQL on GCE (Compute Engine) | ✅ Yes | Full control — best option |
| ✅ MySQL on GKE (Kubernetes)     | ✅ Yes | As long as MySQL allows triggers |
| ❌ Cloud SQL for MySQL (Managed) | ❌ No  | Restrictions: no SUPER, no custom triggers |

---

## 🛠️ If you're using **MySQL on GCE (VM)**

### ✅ Step-by-step to use `pt-online-schema-change`:

---

### 🔹 1. **Install Percona Toolkit**

SSH into your GCE VM:
```bash
sudo apt update
sudo apt install percona-toolkit
```

---

### 🔹 2. **Make sure MySQL user has permissions**

The user should have:
- `ALTER`
- `INSERT`
- `DELETE`
- `CREATE`
- `DROP`
- `TRIGGER`
- `SELECT`
- (SUPER is optional but not required if using `--set-vars`)

---

### 🔹 3. **Run a Dry Run First**

```bash
pt-online-schema-change \
  --alter "ADD COLUMN email_verified TINYINT(1) DEFAULT 0" \
  --user=root --password=yourpass --host=127.0.0.1 \
  D=mydb,t=users \
  --dry-run
```

---

### 🔹 4. **Execute the Change**

If dry-run is successful:

```bash
pt-online-schema-change \
  --alter "ADD COLUMN email_verified TINYINT(1) DEFAULT 0" \
  --user=root --password=yourpass --host=127.0.0.1 \
  D=mydb,t=users \
  --execute
```

---

### 🔹 5. **Monitor the process**

- Add flags like:
  - `--chunk-size=1000`
  - `--max-lag=2`
  - `--set-vars=innodb_lock_wait_timeout=60`
- Logs will tell you how many rows are copied, how many triggers applied.

---

## ⚠️ If you are using **Cloud SQL for MySQL (Managed)**

> **You cannot use `pt-online-schema-change` on Cloud SQL.**

### ❌ Why?
Cloud SQL restricts:
- `SUPER` privilege
- `TRIGGER` creation
- Low-level access to system resources

---

### ✅ Alternatives for Cloud SQL:

#### 1. **Use native online DDL (MySQL 8.0)**

MySQL 8.0 supports:
```sql
ALTER TABLE my_table ADD COLUMN new_col INT DEFAULT 0, ALGORITHM=INPLACE;
```

Some changes can even be **`INSTANT`**, depending on the column type.

---

#### 2. **Create a clone + swap strategy** (zero-downtime):

- Create a new table with updated schema.
- Backfill data via cron/script.
- Sync changes using triggers or app-layer dual writes.
- Rename new table when ready.

---

#### 3. **Use Database Migration Service (DMS)**

- Create a clone with schema change applied
- Replicate live data into the new instance
- Do a **controlled cutover**

---

## 📝 Summary

| GCP MySQL Environment | Can Use PCT? | Recommended Action |
|------------------------|--------------|--------------------|
| GCE VM or GKE          | ✅ Yes       | Install toolkit and run it directly |
| Cloud SQL for MySQL    | ❌ No        | Use native DDL, clone strategy, or DMS |

---

