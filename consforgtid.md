

---

## 💡 What Makes a DDL Non-Deterministic or Non-Transactional?

### 📘 DDL (Data Definition Language) = SQL statements that define or modify schema:
Examples: `CREATE`, `ALTER`, `DROP`, `RENAME`, etc.

---

### 🔄 1. **Non-Deterministic DDLs**
These are DDL statements whose results can **vary between servers**. This causes replication issues when using **GTID**, because GTID requires that **all statements behave identically** on all replicas.

#### 🔥 Examples of Non-Deterministic DDLs:

| Statement                         | Why it's non-deterministic?                      |
|----------------------------------|--------------------------------------------------|
| `CREATE TABLE ... SELECT`        | Data copied might differ between source & replica |
| `ALTER TABLE ... PARTITION`      | Partition hashing is internal, can vary          |
| `RENAME TABLE db1.tbl TO db2.tbl`| Behavior can differ due to permissions, locks    |

---

### 🔒 2. **Non-Transactional DDLs**
These are DDLs that **cannot be rolled back** once executed — they are immediately and permanently applied.

#### 🔥 Examples of Non-Transactional DDLs:

| Statement                | Why it's non-transactional?                    |
|-------------------------|------------------------------------------------|
| `ALTER TABLE`           | Most `ALTER` operations are auto-committed     |
| `DROP TABLE`            | Can't undo a dropped table                     |
| `CREATE INDEX`          | May be non-atomic, depending on MySQL version |
| `ANALYZE TABLE`, `OPTIMIZE TABLE` | Not transactional; affects stats directly  |

---

### 📋 Why This Matters in GTID Mode?

GTID requires **strict consistency and deterministic execution** so that **replication doesn't break**.

That’s why **MySQL blocks any DDL** that is:
- Not guaranteed to behave the same everywhere (non-deterministic),
- Or cannot be part of a transaction (non-transactional),
**when `enforce_gtid_consistency=ON`**.

---

### ✅ How to Check/Bypass This?

#### ✅ Safe DDL Examples:
```sql
CREATE TABLE employees (id INT PRIMARY KEY, name VARCHAR(100));
ALTER TABLE employees ADD COLUMN email VARCHAR(100);
DROP TABLE employees;
```

#### ❌ Unsafe DDL Examples (under GTID):
```sql
CREATE TABLE backup_employees SELECT * FROM employees;
ALTER TABLE employees PARTITION BY HASH(id);
RENAME TABLE db1.table TO db2.table;
```

---

### 🧠 Summary

| Type                | Explanation                                                 | GTID Compatible |
|---------------------|-------------------------------------------------------------|------------------|
| Non-deterministic   | Behavior varies on replicas (e.g., depends on runtime data) | ❌               |
| Non-transactional   | Cannot be rolled back, not atomic                           | ❌               |
| Safe DDL            | Deterministic + transactional or atomic enough              | ✅               |

