

---

## 🛠️ STEP-BY-STEP: Cloud SQL MySQL Master → Multiple Replicas with Table Filtering

---

### 🧾 Prerequisites

- ✅ A Google Cloud Project
- ✅ Cloud SQL Admin permissions
- ✅ Billing enabled
- ✅ API enabled: Cloud SQL Admin API

---

### 1️⃣ **Create the Primary (Master) Cloud SQL MySQL Instance**

1. Go to [Google Cloud Console](https://console.cloud.google.com/sql)
2. Click **"Create Instance" > "MySQL"**
3. Set:
   - Name: `mysql-master`
   - Version: MySQL 8.0
   - Region/Zone: your choice
   - Root password: set and store securely
4. Enable **automated backups** and **binary logging** (required for replication):
   - **Flag to add**:  
     `log_bin = on`
5. Create the instance

---

### 2️⃣ **Create a Test Database and Tables**

Connect to master:
```bash
gcloud sql connect mysql-master --user=root
```

Then run:
```sql
CREATE DATABASE shop;
USE shop;

CREATE TABLE customers (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100)
);

CREATE TABLE orders (
  id INT AUTO_INCREMENT PRIMARY KEY,
  customer_id INT,
  amount DECIMAL(10,2)
);
```

---

### 3️⃣ **Insert Sample Data**
```sql
INSERT INTO customers (name) VALUES ('Alice'), ('Bob');
INSERT INTO orders (customer_id, amount) VALUES (1, 99.99), (2, 199.99);
```

---

### 4️⃣ **Create Replica A (replicates only `orders`)**

1. Go to the master instance page → Click **“Create Replica”**
2. Name: `replica-orders`
3. Set region/zone
4. Go to **"Customize Configuration" → Add Flags:**
   ```txt
   replicate-do-table=shop.orders
   ```
5. Create the replica

---

### 5️⃣ **Create Replica B (replicates only `customers`)**

Repeat the steps above, but set:

- Name: `replica-customers`
- Flag:
  ```txt
  replicate-do-table=shop.customers
  ```

---

### 6️⃣ **Test Replication Filtering**

Login to master:
```bash
gcloud sql connect mysql-master --user=root
```

Insert more data:
```sql
INSERT INTO customers (name) VALUES ('Charlie');
INSERT INTO orders (customer_id, amount) VALUES (1, 299.99);
```

---

### 7️⃣ **Connect to Replicas & Validate**

Replica A (orders only):
```bash
gcloud sql connect replica-orders --user=root
USE shop;
SELECT * FROM orders;       -- ✅ Should show all orders
SELECT * FROM customers;    -- ❌ Should be missing or empty
```

Replica B (customers only):
```bash
gcloud sql connect replica-customers --user=root
USE shop;
SELECT * FROM customers;    -- ✅ Should show all customers
SELECT * FROM orders;       -- ❌ Should be missing or empty
```

---

## ✅ DONE!

You now have:
- One primary MySQL instance
- Two filtered replicas:
  - `replica-orders`: syncs only the `orders` table
  - `replica-customers`: syncs only the `customers` table

---

