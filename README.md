Conceptual Data Model

A customer can place many orders; an order belongs to exactly one customer.
An order has many order items; each order item refers to exactly one product.
An order can have multiple payments (e.g., split tender: gift card + card).
A shipping address can be reused by many orders (e.g., “home”), and customers can save multiple addresses.
We keep price/discount at the order item.

Logical Data Model

Fact Table and Grain
Central fact: FACT_ORDER_ITEM
Grain: one row per order line item at the time of order.
If an order contains three different products, the fact stores three rows. This grain preserves maximum analytical flexibility (e.g., price/discount at the item level, product/category rollups, order-level metrics via degenerate dimensions).
Foreign Keys in the Fact
date_key (order date) → DIM_DATE
ship_date_key (shipment date) → DIM_DATE
customer_key → DIM_CUSTOMER (SCD2)
product_key → DIM_PRODUCT (SCD2)
shipping_address_key → DIM_SHIPPING_ADDRESS
order_status_key → DIM_ORDER_STATUS
sales_channel_key → DIM_SALES_CHANNEL

Degenerate Dimensions (DDs) in the Fact
(Operational identifiers retained on the fact, no separate dimension table)
order_id, order_line_number, invoice_number, promo_code, shipment_tracking_number, payment_reference

Numeric Measures (additive unless noted)
quantity_sold
unit_price (used to derive gross)
gross_item_amount = quantity_sold × unit_price
discount_amount
item_tax_amount
allocated_shipping_amount (line share of order shipping)
net_item_sales_amount = gross − discount
total_item_revenue = net + tax + allocated shipping

Why this grain?
It supports all required analytics—top products, price change impact, regional performance, Avg Order Value by segment—without losing item-level detail or forcing complex allocations later.

Dimensions and Attributes
1) DIM_CUSTOMER — SCD Type 2
Tracks customer profile changes over time for historically correct reporting.
Keys / SCD fields:
customer_key (surrogate PK), customer_id (natural key), valid_from, valid_to (NULL = current), is_current.
Attributes:
full_name, first_name, last_name, email, phone, customer_segment, loyalty_tier, signup_date .
2) DIM_PRODUCT — SCD Type 2
Captures changing product attributes that affect analysis (especially price and category).
Keys / SCD fields:
product_key (surrogate PK), product_id (SKU), valid_from, valid_to, is_current.
Attributes:
product_name, brand, category (single consolidated field per your requirement),
current_list_price, model_number, color, size, is_active, release_date, discontinue_date.
3) DIM_DATE — Standard Calendar (and optional Fiscal)
Provides consistent time slicing and efficient predicate pushdown.
Key: date_key (e.g., YYYYMMDD)
Attributes:
full_date, day, day_of_week_num, day_of_week_name, is_weekend_flag,
week_of_year, month_num, month_name, quarter_num, quarter_name, year, fiscal_month_num, fiscal_quarter_num, fiscal_year.
4) DIM_SHIPPING_ADDRESS — Shipping Location for Analytics
Key: shipping_address_key (surrogate PK)
Attributes:
recipient_name, address_line1, address_line2, city, state_province,
postal_code, country, phone, region.

Physical Data Model

1) Primary Keys & Foreign Keys
PKs: Each dimension uses a surrogate PK (customer_key, product_key, shipping_address_key, etc.). This gives stable, integer join keys and enables SCD2 versioning for Customer/Product.
FKs: FACT_ORDER_ITEM declares FKs to every dimension (Date, Customer, Product, Shipping Address, and optional Order Status / Sales Channel). 
2) Data Types
Keys: BIGINT/SMALLINT/INTEGER for surrogate/foreign keys → fast joins.
Business IDs & codes: VARCHAR (sized for source).
Dates: DATE for valid_from, valid_to, full_date, etc.
Money: DECIMAL(18,2) for prices/amounts (avoids float rounding).
Flags: BOOLEAN for is_current, is_weekend_flag.
3)Distribution Style (DISTSTYLE)
fact_order_item	- KEY - on product_key	Most heavy queries join/group by product; co-locating fact with product minimizes data movement on the largest join.
dim_product - KEY - on product_key	Matches the fact’s dist key → co-location for fast fact↔product joins.
dim_customer - EVEN -	Potentially large; we optimize for product collocation and evenly spread customers to avoid skew.
dim_shipping_address -	EVEN -	Medium/large depending on dedupe; EVEN prevents node skew.
dim_date - ALL -	small and used in almost every query; broadcasting removes redistribution.
dim_order_status -	ALL	- small tables.
dim_sales_channel	- ALL	- small tables.
4)Sort Keys
fact_order_item	COMPOUND (date_key, order_id)	date_key first enables strong time-range pruning; order_id second clusters order lines for faster AOV/order rollups and window functions.
dim_product	COMPOUND (product_id, valid_from)	Speeds point-in-time SCD lookups for the correct product version at order time.
dim_date	(date_key)	Natural access path; keeps lookups instant (even though it’s small).
Other dims-	They’re small; extra sort keys add little value—simpler is better.

Answering Business Requirement
1) Top 10 best-selling products (revenue & quantity, last quarter)
 Groups fact_order_item by dim_product.product_name, filters last quarter via dim_date, orders by SUM(total_item_revenue) and SUM(quantity_sold), LIMIT 10.
 Line-item grain + product dimension = exact counts and revenue per product. The date filter ensures it’s only last quarter, and sorting/limit gives the top 10.

2) Most valuable customers + geographical locations
 Joins fact_order_item → dim_customer and dim_shipping_address, sums total_item_revenue, orders descending.
 Customer identifies “who spent the most”; shipping address supplies country/state/city/postcode for location. Summing revenue per customer + location answers “who” and “where”.

3) How prices changed over time & impact on sales
 Trend query joining fact_order_item to dim_product and dim_date, using AVG(p.current_list_price) with SUM(quantity_sold)/SUM(total_item_revenue) by month (and optional price    bands).
 SCD2 on dim_product ties each sale to the price version valid at order time (valid_from/valid_to). That makes month-over-month price vs. units/revenue comparisons historically  correct.

4) Average Order Value (AOV) by customer segment
 SUM(total_item_revenue) / COUNT(DISTINCT order_id) grouped by dim_customer.customer_segment (and optionally by time from dim_date).
 AOV is revenue per order. Because order_id is a degenerate dimension on the fact, you can do an accurate COUNT(DISTINCT order_id) while segment comes from the customer    dimension.

5) Sales performance by region
 Aggregates SUM(total_item_revenue) and SUM(quantity_sold) grouped by dim_shipping_address.region (and/or country/state/city), filtered by time via dim_date.
 The shipping address provides the geographic roll-up fields; summing revenue/units by those attributes gives regional performance. The date filter scopes the period.

