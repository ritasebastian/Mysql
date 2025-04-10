---

### 🚀 What is DML in MySQL?

DML stands for **Data Manipulation Language**, which includes SQL statements that **modify data** in the database.

#### ✅ Common DML Commands:
- `INSERT`: Add new rows
- `UPDATE`: Modify existing rows
- `DELETE`: Remove rows
- `SELECT`: (technically read-only, but still often grouped here)

---

### 🔄 DML Execution Lifecycle in MySQL (with InnoDB engine)

Let’s walk through what happens internally when a DML command is run:

---

#### 1. **Client Sends Query**

Example:
```sql
UPDATE employees SET salary = salary * 1.1 WHERE department = 'Sales';
```

---

#### 2. **Parser and Optimizer**

- MySQL parses the query for syntax.
- Then the **optimizer** decides how to best execute the query (e.g., use indexes or not).

---

#### 3. **Locking (InnoDB)**

- **Row-level locks** are acquired to prevent other transactions from modifying the same rows.
- Uses **MVCC (Multi-Version Concurrency Control)** for consistency during concurrent transactions.

---

#### 4. **Undo Log Creation**

- Before modifying the row, MySQL stores the **old version** in the **Undo Log** (used for rollback and MVCC).

---

#### 5. **Change Happens in Buffer Pool (RAM)**

- The row is updated **in memory** inside the **InnoDB Buffer Pool**.
- Marked as "dirty" (modified) – not yet written to disk.

---

#### 6. **Redo Log Update (WAL - Write Ahead Log)**

- MySQL writes a **redo log entry** to ensure changes can be replayed if MySQL crashes before flushing to disk.

---

#### 7. **Binary Log (if enabled)**

- For replication, DML changes are written to the **binary log** so replicas can apply them.

---

#### 8. **Commit or Rollback**

- If autocommit is ON, MySQL commits immediately.
- If in a transaction:
  - `COMMIT` → changes are permanent.
  - `ROLLBACK` → revert changes using Undo Log.

---

#### 9. **Checkpoint & Flush to Disk**

- Periodically, dirty pages in buffer pool are flushed to disk.
- Controlled by background threads (flush thread, log writer, etc.).

---

### 🔐 Transaction Safe?

If using **InnoDB**, yes:
- `BEGIN ... COMMIT` ensures atomic DML operations.
- Follows **ACID** principles (Atomicity, Consistency, Isolation, Durability).

---

### 🧪 Example in Action

```sql
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

- Both updates succeed → committed.
- One fails → both rolled back using **undo logs**.

---
![image](https://github.com/user-attachments/assets/bcc988ef-f53e-428c-8bdf-e29c1bd672ef)

