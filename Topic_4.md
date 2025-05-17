# Integrity and Security

---

## ✅ 1. Domain Constraints

**Definition:**  
Domain constraints specify that all values in a column must be of the same data type and within a specific valid range or format.

**Real-World Example:**  
In a hotel management system, the `room_price` should always be a positive value and not a string.

**SQL Implementation:**
```sql
CREATE TABLE Rooms (
    room_id INT PRIMARY KEY,
    room_type VARCHAR(20),
    room_price DECIMAL(8,2) CHECK (room_price > 0)
);
```

**Explanation:**  
- `DECIMAL(8,2)`: Ensures the data type  
- `CHECK (room_price > 0)`: Ensures the value is within the domain

---

## ✅ 2. Referential Integrity

**Definition:**  
Referential integrity ensures that a foreign key in one table matches a primary key in another, maintaining consistency between related data.

**Real-World Example:**  
In a restaurant ERP system, each order must refer to a valid customer in the Customers table.

**SQL Implementation:**
```sql
CREATE TABLE Customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE Orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    FOREIGN KEY (customer_id) REFERENCES Customers(customer_id)
);
```

**Explanation:**  
- The `Orders` table can only reference `customer_id`s that exist in the `Customers` table.
- Prevents inserting orders for non-existent customers.

---

## ✅ Assertions

**Definition:**  
An assertion is a rule that applies to the whole database — not just one table.
It makes sure a condition is always true in the database.

**Real-World Example:**  
In a **hotel restaurant**, the total number of available waiters should never exceed 50.

---

### 🔹 SQL Implementation (Standard SQL – not supported in MySQL):

```sql
CREATE ASSERTION MaxWaiters
CHECK (
    (SELECT COUNT(*) FROM Waiters) <= 50
);
```
> ⚠️ **Note:** Most RDBMS like MySQL do not support `CREATE ASSERTION`. In such cases, you should use a trigger instead.

---

### 🔹 Alternative Implementation using Trigger (for MySQL/PostgreSQL):

```sql
DELIMITER //

CREATE TRIGGER check_waiter_limit
BEFORE INSERT ON Waiters
FOR EACH ROW
BEGIN
  IF (SELECT COUNT(*) FROM Waiters) >= 50 THEN
    SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Max 50 waiters allowed.';
  END IF;
END;
//

DELIMITER ;
```

---

## 📋 Summary Table

| Concept                   | Purpose                                  | Real-World Example/Use Case              | SQL Implementation                  |
|---------------------------|------------------------------------------|------------------------------------------|-------------------------------------|
| **Domain Constraints**    | Restrict values in a column              | Room price must be positive              | `CHECK (room_price > 0)`            |
| **Referential Integrity** | Maintain data consistency between tables | Orders linked to valid customers         | `FOREIGN KEY REFERENCES`            |
| **Assertions**            | Enforce complex conditions across tables | Max number of waiters should be ≤ 50     | `ASSERTION` or `TRIGGER` workaround |

---
---
# 🟢 Normalization Example

---

## 🔹 Before Normalization

```sql
CREATE TABLE staff (
    staff_id INT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    roles VARCHAR(100) NOT NULL,
    branch_name VARCHAR(100) NOT NULL,
    branch_city VARCHAR(50) NOT NULL
);

INSERT INTO staff (staff_id, name, roles, branch_name, branch_city) VALUES
(1, 'Alice', 'Manager, Cashier', 'Gourmet Bistro', 'Mumbai'),
(2, 'Bob', 'Waiter', 'Gourmet Bistro', 'Mumbai'),
(3, 'Charlie', 'Waiter, Bartender', 'Ocean View Café', 'Goa');
```

❌ **Issue:**  
The `roles` column has multiple values (e.g., "Manager, Cashier") — this violates **First Normal Form (1NF)**.

---

## ✅ Step 1: First Normal Form (1NF)

**Rule:**  
No multivalued attributes — each cell must hold a single value.

**Solution:**  
Split each role into a separate row.

```sql
DROP TABLE IF EXISTS staff_1nf;

CREATE TABLE staff_1nf (
    staff_id INT,
    name VARCHAR(50) NOT NULL,
    role VARCHAR(50) NOT NULL,
    branch_name VARCHAR(100) NOT NULL,
    branch_city VARCHAR(50) NOT NULL
);

INSERT INTO staff_1nf (staff_id, name, role, branch_name, branch_city) VALUES
(1, 'Alice', 'Manager', 'Gourmet Bistro', 'Mumbai'),
(1, 'Alice', 'Cashier', 'Gourmet Bistro', 'Mumbai'),
(2, 'Bob', 'Waiter', 'Gourmet Bistro', 'Mumbai'),
(3, 'Charlie', 'Waiter', 'Ocean View Café', 'Goa'),
(3, 'Charlie', 'Bartender', 'Ocean View Café', 'Goa');
```

---

## 🔵 Step 2: Second Normal Form (2NF)

**Definition:**  
A table is in 2NF if it is in 1NF and every non-key attribute is fully functionally dependent on the entire primary key (no partial dependency).

**Problem:**  
In `staff_1nf`, `branch_name` and `branch_city` depend only on `staff_id`, not on the full composite key (`staff_id`, `role`).

**Solution:**  
Split into two tables:  
- **Staff**: stores staff info  
- **Staff_Roles**: stores roles per staff

### Staff Table

```sql
DROP TABLE IF EXISTS staff;
CREATE TABLE staff (
    staff_id INT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    branch_name VARCHAR(100) NOT NULL,
    branch_city VARCHAR(50) NOT NULL
);

INSERT INTO staff (staff_id, name, branch_name, branch_city) VALUES
(1, 'Alice', 'Gourmet Bistro', 'Mumbai'),
(2, 'Bob', 'Gourmet Bistro', 'Mumbai'),
(3, 'Charlie', 'Ocean View Café', 'Goa');
```

### Staff_Roles Table

```sql
DROP TABLE IF EXISTS staff_roles;
CREATE TABLE staff_roles (
    staff_id INT,
    role VARCHAR(50),
    PRIMARY KEY (staff_id, role),
    FOREIGN KEY (staff_id) REFERENCES staff(staff_id)
);

INSERT INTO staff_roles (staff_id, role) VALUES
(1, 'Manager'),
(1, 'Cashier'),
(2, 'Waiter'),
(3, 'Waiter'),
(3, 'Bartender');
```

| Before (1NF)                              | After (2NF)                                 |
|--------------------------------------------|---------------------------------------------|
| One table mixing staff info and roles      | Two tables: staff info and roles separated  |
| Partial dependency exists                  | Each non-key attribute fully depends on PK  |

---

## 🟣 Step 3: Third Normal Form (3NF)

**Goal:**  
Remove transitive dependencies.

### 1. Create the Branch Table

```sql
CREATE TABLE Branch (
    branch_name VARCHAR(100) PRIMARY KEY,
    branch_city VARCHAR(50) NOT NULL
);
```

### 2. Create the Staff Table

```sql
CREATE TABLE Staff (
    staff_id INT PRIMARY KEY,
    staff_name VARCHAR(50) NOT NULL,
    branch_name VARCHAR(100),
    FOREIGN KEY (branch_name) REFERENCES Branch(branch_name)
);
```

### 3. Insert Sample Data

```sql
INSERT INTO Branch (branch_name, branch_city) VALUES
('Gourmet Bistro', 'Mumbai'),
('Ocean View Café', 'Goa');

INSERT INTO Staff (staff_id, staff_name, branch_name) VALUES
(1, 'Alice', 'Gourmet Bistro'),
(2, 'Bob', 'Gourmet Bistro'),
(3, 'Charlie', 'Ocean View Café');
```

---

**Summary:**  
- The **Branch** table stores branch info (`branch_name`, `branch_city`).
- The **Staff** table stores staff info, referencing branch by `branch_name`.
- This avoids repeating city info for each staff member.
- It removes transitive dependency → Your database is in **3NF**!

---

# 🟡 Candidate Key Example

## Example Table: Employee

| Employee_ID | Email               | Phone_Number | Name    |
|-------------|---------------------|--------------|---------|
| 101         | alice@example.com   | 1234567890   | Alice   |
| 102         | bob@example.com     | 0987654321   | Bob     |
| 103         | charlie@example.com | 1231231234   | Charlie |

---

## Candidate Keys here:

- **Employee_ID** uniquely identifies each employee.
- **Email** also uniquely identifies each employee.
- **Phone_Number** could also uniquely identify each employee.

---

## So in this case:

- **Employee_ID**, **Email**, and **Phone_Number** are all **candidate keys** because each can uniquely identify a record.
- You can pick **one** as the **primary key** (usually **Employee_ID**).

---

# 🟣 Boyce-Codd Normal Form (BCNF)

---

## Summary Table

| Candidate Keys      |
|---------------------|
| Employee_ID         |
| Email               |
| Phone_Number        |

---

### Confirmation

**Q:** Here, the Employee_ID is a Candidate Key, right?  
**A:** Yes, exactly! Employee_ID can be chosen as the candidate key — and usually, it’s the primary key because it uniquely identifies each employee.

But remember, **Email** and **Phone_Number** can also be candidate keys if they uniquely identify employees too.

- **Candidate Keys:** Employee_ID, Email, Phone_Number  
- **Primary Key:** Usually Employee_ID (chosen from candidate keys)
