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


Conceptual Diagram
    CUSTOMER ||--o{ ORDER : places
    ORDER ||--o{ ORDER_ITEM : contains
    ORDER_ITEM }o--|| PRODUCT : refers_to
    ORDER ||--|| PAYMENT : has
    ORDER ||--|| SHIPPING_ADDRESS : ships_to

    CUSTOMER {
        string customer_id
        string name
        string email
    }

    ORDER {
        string order_id
        date order_date
    }

    ORDER_ITEM {
        string order_item_id
        int quantity
        decimal unit_price
    }

    PRODUCT {
        string product_id
        string product_name
        string category
        decimal price
    }

    PAYMENT {
        string payment_id
        string method
        decimal amount
    }

    SHIPPING_ADDRESS {
        string address_id
        string country
        string city
        string zip_code
    }

    Logical start schema
erDiagram
    %% ========= STAR SCHEMA =========
    FACT_SALES }o--|| DIM_CUSTOMER : fk_customer_sk
    FACT_SALES }o--|| DIM_PRODUCT  : fk_product_sk
    FACT_SALES }o--|| DIM_DATE     : fk_date_sk
    FACT_SALES }o--|| DIM_LOCATION : fk_location_sk
    FACT_SALES }o--|| DIM_PAYMENT  : fk_payment_sk

    %% -------- FACT --------
    FACT_SALES {
        bigint  sales_id PK
        string  order_number        "DD"
        string  invoice_number      "DD"
        bigint  customer_sk         "FK"
        bigint  product_sk          "FK"
        int     date_sk             "FK"
        bigint  location_sk         "FK"
        bigint  payment_sk          "FK"
        int     quantity_sold
        decimal unit_price
        decimal discount_amount
        decimal shipping_cost
        decimal extended_price
        decimal order_total
    }

    %% -------- DIMENSIONS --------
    DIM_CUSTOMER {
        bigint  customer_sk PK
        string  customer_id         "BK"
        string  first_name
        string  last_name
        string  email
        string  phone_number
        date    effective_date      "SCD2"
        date    expiry_date         "SCD2"
        boolean is_current          "SCD2 flag"
    }

    DIM_PRODUCT {
        bigint  product_sk PK
        string  product_id          "BK"
        string  product_name
        string  brand
        string  category_l1
        string  category_l2
        string  category_l3
        decimal price
        date    effective_date      "SCD2"
        date    expiry_date         "SCD2"
        boolean is_current          "SCD2 flag"
    }

    DIM_DATE {
        int     date_sk PK
        date    full_date
        int     day_of_month
        int     month
        string  month_name
        int     quarter
        int     year
        string  day_of_week_name
        boolean is_weekend
    }

    DIM_LOCATION {
        bigint  location_sk PK
        string  country
        string  state
        string  city
        string  zip_code
    }

    DIM_PAYMENT {
        bigint  payment_sk PK
        string  payment_method_code
        string  payment_method_desc
        boolean is_third_party
    }

    
