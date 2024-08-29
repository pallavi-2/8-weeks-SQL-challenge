# Readme

![image](https://github.com/user-attachments/assets/7d37c1cc-568c-4af9-842d-fda1540e002f)

### Entity Relationship Diagram

![127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8](https://github.com/user-attachments/assets/33ab1bb3-337a-48cb-b567-1c911625c7a7)


### **Table 1: sales**

The `sales` table captures all `customer_id` level purchases with an corresponding `order_date` and `product_id` information for when and what menu items were ordered.

| **customer_id** | **order_date** | **product_id** |
| --- | --- | --- |
| A | 2021-01-01 | 1 |
| A | 2021-01-01 | 2 |
| A | 2021-01-07 | 2 |
| A | 2021-01-10 | 3 |
| A | 2021-01-11 | 3 |
| A | 2021-01-11 | 3 |
| B | 2021-01-01 | 2 |
| B | 2021-01-02 | 2 |
| B | 2021-01-04 | 1 |
| B | 2021-01-11 | 1 |
| B | 2021-01-16 | 3 |
| B | 2021-02-01 | 3 |
| C | 2021-01-01 | 3 |
| C | 2021-01-01 | 3 |
| C | 2021-01-07 | 3 |

### **Table 2: menu**

The `menu` table maps the `product_id` to the actual `product_name` and `price` of each menu item.

| **product_id** | **product_name** | **price** |
| --- | --- | --- |
| 1 | sushi | 10 |
| 2 | curry | 15 |
| 3 | ramen | 12 |

### **Table 3: members**

The final `members` table captures the `join_date` when a `customer_id` joined the beta version of the Danny’s Diner loyalty program.

| **customer_id** | **join_date** |
| --- | --- |
| A | 2021-01-07 |
| B | 2021-01-09 |