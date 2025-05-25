
## **Database Features to Remember (Just a Few Things to Keep in Mind)**

Been working with databases lately, and here are some features I’ve found useful. Nothing too complicated—just little tricks to make things smoother. Figured I’d write them down so I don’t forget.

---

### **1. Indexes – Speeding Up Searches**

Indexes are like shortcuts. They help the database find things quickly without having to check every row. It's super helpful when you're searching for things like user info or orders, and you don’t want to wait forever for the query to run.

**When to Use:**  
Anytime I’m doing searches or lookups based on a certain column (like `order_date`, `customer_id`, etc.), adding an index on those columns is a good idea. It’ll make the search way faster.

**Example:**  
If I’m frequently searching for orders based on the `order_date` and `customer_id`, I can create an index on those columns:

```sql
CREATE INDEX idx_orders_date_customer ON orders(order_date, customer_id);
```

Now, every time I search for orders based on these fields, the database can find them quicker, and my app will be faster.

---

### **2. Triggers – Doing Things Automatically**

Triggers are basically little helpers that run automatically when something happens in the database. It’s like setting up a rule that says, “Hey, whenever a new order is placed, update the stock!”

**When to Use:**  
If I need something to happen automatically in the background (like adjusting stock after an order, or updating a log every time a record is changed), triggers save me from doing it manually.

**Example:**  
Let’s say I have an `orders` table, and I want to automatically decrease the stock of a product every time an order is placed. I can create a trigger for that:

```sql
CREATE TRIGGER update_stock_after_order
AFTER INSERT ON orders
FOR EACH ROW
BEGIN
    UPDATE products
    SET stock = stock - NEW.quantity
    WHERE product_id = NEW.product_id;
END;
```

With this trigger in place, every time a new order is inserted into the `orders` table, the stock in the `products` table will automatically decrease by the quantity ordered.

---

### **3. Views – Simplifying Queries**

Views are like saved queries. If I have a complex query that I need to run over and over, I can just save it as a view. That way, I don’t have to keep writing the same thing.

**When to Use:**  
If I find myself repeating the same complex `JOIN` or `GROUP BY` queries multiple times, a view can simplify things a lot. Instead of writing the same query, I can just use the view like a table.

**Example:**  
Let’s say I often need to know how many orders each customer has placed. Instead of writing a long `JOIN` query every time, I could create a view that does the work for me:

```sql
CREATE VIEW customer_order_summary AS
SELECT customers.customer_id, customers.name, COUNT(orders.order_id) AS order_count
FROM customers
JOIN orders ON customers.customer_id = orders.customer_id
GROUP BY customers.customer_id;
```

Now, whenever I need this info, I can just do:

```sql
SELECT * FROM customer_order_summary;
```

I don’t have to worry about writing the `JOIN` and `GROUP BY` part again. Nice and simple.

---

### **4. Foreign Keys – Keeping Things Consistent**

Foreign keys are great for making sure everything in my database is linked correctly. They’re like saying, “Hey, if you’re going to add an order, make sure it’s linked to a valid customer.”

**When to Use:**  
Whenever I have multiple tables that are related to each other (like `customers` and `orders`), foreign keys help me keep the relationships intact. They make sure that, for example, every order has a valid customer attached.

**Example:**  
If I have a `customers` table and an `orders` table, I can add a foreign key to ensure that every order in the `orders` table is linked to a customer in the `customers` table:

```sql
ALTER TABLE orders
ADD CONSTRAINT fk_customer_id
FOREIGN KEY (customer_id) REFERENCES customers(id);
```

This way, I can’t accidentally add an order without a valid customer ID. The database will automatically check for that relationship.

---

### **5. Stored Procedures – Wrapping Up SQL Logic**

Stored procedures are handy when I need to run a set of SQL commands together, and I don’t want to repeat myself. They let me group a bunch of SQL into one reusable "package."

**When to Use:**  
If I often run the same set of queries (for example, updating a customer balance or processing a payment), I can wrap those queries into a stored procedure so I don’t have to type them out every time.

**Example:**  
Let’s say I need to update a customer’s balance every time they make a purchase. Instead of writing the same update command over and over, I can create a stored procedure:

```sql
CREATE PROCEDURE UpdateCustomerBalance(IN customer_id INT, IN amount DECIMAL)
BEGIN
    UPDATE customers
    SET balance = balance - amount
    WHERE id = customer_id;
END;
```

Then, whenever I need to update the balance, I just call the procedure:

```sql
CALL UpdateCustomerBalance(123, 50.00);
```

I don’t need to remember the exact SQL every time—just the procedure name and parameters.

---

### **6. Transactions – Making Sure Everything Works Together**

A **transaction** groups several SQL commands together. It’s like saying, “If everything in this group works, save it; if something goes wrong, don’t save any of it.”

**When to Use:**  
When I’m doing multiple updates to the database that depend on each other (like transferring money between two bank accounts), I need to make sure either all the changes happen, or none of them do. Transactions help with that.

**Example:**  
Let’s say I’m transferring money from one account to another. If I subtract money from Account A but fail to add it to Account B, that’s a big problem. So, I can use a transaction:

```sql
START TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

COMMIT;
```

If anything goes wrong, I can roll it back and neither account will be affected. It’s like a safety net for changes that depend on each other.

---

### **7. Partitioning – Breaking Big Tables Into Chunks**

Partitioning splits a large table into smaller, more manageable parts. It’s kind of like breaking up a massive document into chapters so it’s easier to navigate.

**When to Use:**  
If I’m working with a table that gets really big (millions of rows), partitioning it can help speed up queries and make things more manageable. It’s especially useful when I’m querying by date or a specific range of values.

**Example:**  
If I have a huge `orders` table, I could partition it by year. That way, the database only has to look at the relevant partitions when I query by year:

```sql
CREATE TABLE orders (
    order_id INT,
    order_date DATE,
    customer_id INT
)
PARTITION BY RANGE (YEAR(order_date)) (
    PARTITION p2020 VALUES LESS THAN (2021),
    PARTITION p2021 VALUES LESS THAN (2022)
);
```

Now, instead of querying through the whole table, I’m only dealing with the data for the relevant year, making it faster.

---

### **End Note:**

Just a few things to keep in mind. Nothing too fancy, but definitely useful when I need them. I’ll keep adding to this list as I go. The more I use these features, the more I realize how much easier they make my life.
