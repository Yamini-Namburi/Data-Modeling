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



