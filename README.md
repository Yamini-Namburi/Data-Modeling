erDiagram
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
