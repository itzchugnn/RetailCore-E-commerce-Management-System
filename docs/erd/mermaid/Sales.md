```mermaid
erDiagram
    CUSTOMER ||--o{ SALES_ORDER : "places (1..1 : 0..N)"
    SHOPPING_CART |o--o| SALES_ORDER : "converted_from (0..1 : 0..1)"
    SALES_ORDER |o--o{ SALES_ORDER : "replaces_order (0..1 : 0..N)"

    SALES_ORDER ||--|{ ORDER_LINE : "contains (1..1 : 1..N)"
    PRODUCT_VARIANT ||--o{ ORDER_LINE : "sold_sku (1..1 : 0..N)"

    SALES_ORDER ||--|| ORDER_ADDRESS : "ships_to (1..1 : 1..1)"
    SALES_ORDER ||--|{ ORDER_STATUS_HISTORY : "tracks_status (1..1 : 1..N)"
    ACCOUNT |o--o{ ORDER_STATUS_HISTORY : "changed_by (0..1 : 0..N)"

    SALES_ORDER {
        bigint OrderID PK
        varchar OrderNumber UK
        bigint CustomerID FK
        bigint CartID FK,UK
        varchar OrderType
        bigint ParentOrderID FK
        varchar OrderStatus
        datetimeoffset OrderDate
        decimal SubtotalAmount
        decimal DiscountAmount
        decimal TaxAmount
        decimal ShippingFee
        decimal ShippingDiscountAmount
        decimal TotalAmount
        nvarchar CustomerNote
        datetimeoffset CreatedAt
        datetimeoffset UpdatedAt
        datetimeoffset DeliveredAt
        datetimeoffset ClosedAt
    }
    ORDER_LINE {
        bigint OrderLineID PK
        bigint OrderID FK
        bigint VariantID FK
        varchar SKU
        nvarchar ProductName
        nvarchar VariantName
        nvarchar VariantAttributesSnapshot
        decimal UnitPrice
        decimal VatRate
        decimal TaxAmount
        int OrderedQty
        int CancelledQty
        decimal LineGrossAmount
        decimal LineProductDiscount
        decimal AllocatedOrderDiscount
        decimal LineNetAmount
        int RefundedQty
        decimal AccumulatedRefundedAmount
    }
    ORDER_ADDRESS {
        bigint OrderAddressID PK
        bigint OrderID FK,UK
        nvarchar RecipientName
        varchar RecipientPhone
        nvarchar Province
        nvarchar District
        nvarchar Ward
        nvarchar StreetAddress
        nvarchar BuyerCompanyName
        varchar BuyerTaxCode
        nvarchar BuyerCompanyAddress
        varchar BuyerInvoiceEmail
    }
    ORDER_STATUS_HISTORY {
        bigint StatusHistoryID PK
        bigint OrderID FK
        varchar PreviousStatus
        varchar NewStatus
        datetimeoffset ChangedAt
        bigint ChangedBy FK
        nvarchar ChangeReason
    }
```
