GlobalCart E-commerce Analytics Platform
Business Scenario
GlobalCart is a rapidly expanding online retailer specializing in consumer electronics. To sustain its growth and make data-driven decisions, the company needs to build a centralized data warehouse on AWS Redshift. The primary goal is to gain deep insights into sales performance, customer purchasing habits, product popularity, and inventory levels.
The business wants to answer questions like:
What are our top 10 best-selling products by revenue and quantity in the last quarter?
Who are our most valuable customers (by total spending) and what are their geographical locations?
How have product prices changed over time, and how did that affect sales?
What is the average order value by customer segment?
How does sales performance vary across different regions?
Technical Context
Data Sources: Transactional data comes from a PostgreSQL database. Product and customer updates are provided as daily CSV files.
Data Warehouse: AWS Redshift.
Key Business Entities: Customers, Products, Orders, Order Items, Payments, and Shipping Addresses.
Your Task
You are tasked with designing a dimensional data model for GlobalCart's new analytics platform. Your design should be optimized for OLAP queries and scalable for future growth.
Expected Outcomes
1. Conceptual Data Model:
Create a high-level diagram (e.g., an ERD) that illustrates the main business entities and the relationships between them. This should be a business-focused view, not a technical one.
2. Logical Data Model (Dimensional Model):
Design a Star Schema for the data warehouse.
Clearly define one central Fact Table for sales transactions. Specify the grain of the fact table. List all degenerate dimensions, foreign keys, and numeric measures (e.g., order_total, quantity_sold, discount_amount).
Define all necessary Dimension Tables. For each dimension, list its attributes.
dim_customer: This dimension must track customer history. If a customer changes their address or name, the old information should be preserved for historical reporting.
dim_product: This dimension must track product history. If a product's price or category changes, you need to be able to report sales against the values that were valid at the time of the transaction.
dim_date: A standard date dimension with attributes like year, quarter, month, day, day of the week, etc.
dim_location: A dimension for customer shipping locations (Country, State, City, Zip Code).
Provide a diagram of your logical star schema, showing the relationships between the fact and dimension tables.
3. Physical Data Model:
Write the complete DDL (Data Definition Language) SQL scripts to create your fact and dimension tables in AWS Redshift.
Your DDL must include:
Primary keys and foreign key relationships.
Appropriate data types for all columns.
Redshift-specific optimizations:
Choose and justify a Distribution Style (DISTSTYLE) for each table (e.g., KEY, ALL, EVEN). Explain why your choice is optimal for query performance.
Choose and justify Sort Keys (SORTKEY) for the fact table and any large dimension tables. Explain how your choices will speed up common queries.
In a separate section, briefly explain your reasoning for the chosen SCD (Slowly Changing Dimension) types for the dim_customer and dim_product tables.


