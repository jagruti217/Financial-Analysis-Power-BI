# Financial-Analysis-Power-BI
This project focuses on analyzing credit card usage and financial metrics for a banking institution using Power BI and DAX queries. The goal is to provide key insights into customer behavior, credit utilization, and financial performance by applying advanced DAX functions.

# Store and Product Sales Analysis

- **Total Products Sold by Store:**
  ```sql
  SELECT 
    stores.store_name, 
    SUM(order_items.quantity) AS total_products_sold
  FROM 
    order_items
  JOIN orders ON order_items.order_id = orders.order_id
  JOIN stores ON stores.store_id = orders.store_id
  GROUP BY stores.store_name;

