
---

### ✅ Sample Function: `get_discounted_price`

#### 🛠️ 1. **Create Function**
```sql
DELIMITER $$

CREATE FUNCTION get_discounted_price(original_price DECIMAL(10,2), discount_percent INT)
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
  DECLARE discounted_price DECIMAL(10,2);
  SET discounted_price = original_price - (original_price * discount_percent / 100);
  RETURN discounted_price;
END$$

DELIMITER ;
```

---

### ✅ Sample Procedure: `insert_product`

#### 🛠️ 2. **Create Table for Testing**
```sql
CREATE TABLE products (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100),
  original_price DECIMAL(10,2),
  discounted_price DECIMAL(10,2)
);
```

#### 🛠️ 3. **Create Stored Procedure**
```sql
DELIMITER $$

CREATE PROCEDURE insert_product(
  IN prod_name VARCHAR(100),
  IN prod_price DECIMAL(10,2),
  IN discount INT
)
BEGIN
  DECLARE final_price DECIMAL(10,2);
  SET final_price = get_discounted_price(prod_price, discount);
  INSERT INTO products (name, original_price, discounted_price)
  VALUES (prod_name, prod_price, final_price);
END$$

DELIMITER ;
```

---

### 🧪 Testing the Function & Procedure

#### ✅ Call the Procedure:
```sql
CALL insert_product('Laptop', 1200.00, 10);
CALL insert_product('Tablet', 800.00, 15);
```

#### ✅ Check the Results:
```sql
SELECT * FROM products;
```

Expected output:
```
+----+---------+----------------+------------------+
| id | name    | original_price | discounted_price |
+----+---------+----------------+------------------+
|  1 | Laptop  |        1200.00 |          1080.00 |
|  2 | Tablet  |         800.00 |           680.00 |
+----+---------+----------------+------------------+
```

---


## ✅ 1. Calling a **Function**

### 📌 Syntax (used inside a `SELECT`)
```sql
SELECT function_name(param1, param2, ...);
```

### 🧪 Example
```sql
-- Call a function to get discounted price
SELECT get_discounted_price(1000, 10);
-- Output: 900.00
```

### 🔍 Note:
You can also use functions inside queries:
```sql
SELECT name, get_discounted_price(price, 10) AS discounted_price FROM products;
```

---

## ✅ 2. Calling a **Stored Procedure**

### 📌 Syntax
```sql
CALL procedure_name(param1, param2, ...);
```

### 🧪 Example
```sql
-- Call a procedure to insert a product
CALL insert_product('Smartphone', 600, 15);
```

It will:
- Calculate discount using your `get_discounted_price()` function
- Insert the product into the `products` table

---

### 🧠 Summary Table

| Type         | Keyword | Usage                              | Can return value directly? |
|--------------|---------|-------------------------------------|-----------------------------|
| **Function** | `SELECT`| `SELECT func(a, b);`               | ✅ Yes                      |
| **Procedure**| `CALL`  | `CALL proc(a, b);`                 | ❌ No (but can use `OUT` vars or `SELECT` inside) |

---


