SQL Assignment - 1

### 1 New Customers Acquired in June 2023
  
Business Problem:  
The marketing team ran a campaign in June 2023 and wants to see how many new customers signed up during that period.

Fields to Retrieve:

*   PARTY\_ID
*   FIRST\_NAME
*   LAST\_NAME
*   EMAIL
*   PHONE
*   ENTRY\_DATE

SELECT 

  person.party_id, 
 
  first_name, 
  
  last_name, 

  max(
    case when contact_mech_purpose_type_id = "PRIMARY_EMAIL" then info_string end
  ) AS email, 
 
  max(
    case when contact_mech_purpose_type_id = "PRIMARY_PHONE" then contact_number end
  ) AS phone, 
 
  party_role.created_stamp AS entry_date 

FROM 
 
  PERSON 
 
  LEFT JOIN party_contact_mech_purpose ON person.party_id = party_contact_mech_purpose.party_id 
  and (
    party_contact_mech_purpose.CONTACT_MECH_PURPOSE_TYPE_ID in (
      "PRIMARY_EMAIL", "PRIMARY_PHONE"
    )
  ) 
  and (
    party_contact_mech_purpose.THRU_DATE is null 
    or party_contact_mech_purpose.THRU_DATE > now()
  ) 
  
  LEFT JOIN contact_mech ON party_contact_mech_purpose.contact_mech_id = contact_mech.contact_MECH_ID 
 
  LEFT JOIN telecom_Number ON party_contact_mech_purpose.contact_mech_id = telecom_number.contact_MECH_ID 
 
  JOIN party_role ON person.party_id = party_role.party_id 

WHERE 
 
  party_role.created_stamp >= '2023-06-01' 
  AND party_role.created_stamp <= '2026-06-30' 
  AND PARTY_ROLE.role_type_id = "CUSTOMER" 

group by 
  person.party_id, 
  first_name, 
  last_name, 
  entry_date;



Business Problem:  
Merchandising teams often need a list of all physical products to manage logistics, warehousing, and shipping.

Fields to Retrieve:

*   PRODUCT\_ID
*   PRODUCT\_TYPE\_ID
*   INTERNAL\_NAME

### SELECT

### PRODUCT\_ID,

### PRODUCT.PRODUCT\_TYPE\_ID,

### INTERNAL\_NAME

### FROM PRODUCT

### JOIN PRODUCT\_TYPE

### ON PRODUCT.PRODUCT\_TYPE\_ID = PRODUCT\_TYPE.PRODUCT\_TYPE\_ID

### WHERE IS\_VARIANT ="Y"

### AND IS\_PHYSICAL ="Y";

### 3 Products Missing NetSuite ID

Business Problem:  
A product cannot sync to NetSuite unless it has a valid NetSuite ID. The OMS needs a list of all products that still need to be created or updated in NetSuite.

Fields to Retrieve:

*   PRODUCT\_ID
*   INTERNAL\_NAME
*   PRODUCT\_TYPE\_ID
*   NETSUITE\_ID (or similar field indicating the NetSuite ID; may be NULL or empty if missing)

select count(distinct p.product\_id ) from product p left join good\_identification g on p.product\_id=g.product\_id and good\_identification\_type\_ID ="ERP\_ID" AND ID\_VALUE IS NULL;

### 4 Product IDs Across Systems

Business Problem:  
To sync an order or product across multiple systems (e.g., Shopify, HotWax, ERP/NetSuite), the OMS needs to know each system’s unique identifier for that product. This query retrieves the Shopify ID, HotWax ID, and ERP ID (NetSuite ID) for all products.

Fields to Retrieve:

*   PRODUCT\_ID (internal OMS ID)
*   SHOPIFY\_ID
*   HOTWAX\_ID
*   ERP\_ID or NETSUITE\_ID (depending on naming)

**SELECT**

**product\_id,**

**CASE**

**WHEN good\_identification\_type\_id ="SHOPIFY\_PROD\_ID"**

**THEN id\_value**

**END AS shopify\_id,**

**CASE**

**WHEN good\_identification\_type\_id = "ERP\_ID "**

**THEN id\_value**

**END AS ERP\_id,**

**PRODUCT\_ID AS HOTWAX\_id**

**FROM GOOD\_IDENTIFICATION;**

### 5 Completed Orders in August 2023

Business Problem:  
After running similar reports for a previous month, you now need all completed orders in August 2023 for analysis.

Fields to Retrieve:

*   PRODUCT\_ID
*   PRODUCT\_TYPE\_ID
*   PRODUCT\_STORE\_ID
*   TOTAL\_QUANTITY
*   INTERNAL\_NAME
*   FACILITY\_ID
*   EXTERNAL\_ID
*   FACILITY\_TYPE\_ID
*   ORDER\_HISTORY\_ID
*   ORDER\_ID
*   ORDER\_ITEM\_SEQ\_ID
*   SHIP\_GROUP\_SEQ\_ID

### SELECT

### ORDER\_ITEM.PRODUCT\_ID,

### PRODUCT.PRODUCT\_TYPE\_ID,

### ORDER\_HEADER.PRODUCT\_STORE\_ID,

### SUM(ORDER\_ITEM.QUANTITY) AS TOTAL\_QUANTITY,

### ORDER\_HEADER.ORDER\_NAME AS INTERNAL\_NAME,

### ORDER\_ITEM\_SHIP\_GROUP.FACILITY\_ID,

### ORDER\_HEADER.EXTERNAL\_ID,

### FACILITY.FACILITY\_TYPE\_ID,

### ORDER\_HISTORY.ORDER\_HISTORY\_ID,

### ORDER\_HEADER.ORDER\_ID,

### ORDER\_ITEM.ORDER\_ITEM\_SEQ\_ID,

### ORDER\_ITEM\_SHIP\_GROUP.SHIP\_GROUP\_SEQ\_ID

### FROM ORDER\_HEADER

### JOIN ORDER\_ITEM

### ON ORDER\_HEADER.ORDER\_ID = ORDER\_ITEM.ORDER\_ID

### JOIN ORDER\_ITEM\_SHIP\_GROUP

### ON ORDER\_HEADER.ORDER\_ID = ORDER\_ITEM\_SHIP\_GROUP.ORDER\_ID

### LEFT JOIN ORDER\_HISTORY

### ON ORDER\_HEADER.ORDER\_ID = ORDER\_HISTORY.ORDER\_ID

### LEFT JOIN PRODUCT

### ON ORDER\_ITEM.PRODUCT\_ID = PRODUCT.PRODUCT\_ID

### LEFT JOIN FACILITY

### ON ORDER\_ITEM\_SHIP\_GROUP.FACILITY\_ID = FACILITY.FACILITY\_ID

### WHERE ORDER\_HEADER.STATUS\_ID LIKE "ORDER\_COMPLETED"

### AND ORDER\_HEADER.ORDER\_DATE >= "2023-08-01"

### AND ORDER\_HEADER.ORDER\_DATE <= "2023-08-31"

### GROUP BY

### ORDER\_ITEM.PRODUCT\_ID,

### PRODUCT.PRODUCT\_TYPE\_ID,

### ORDER\_HEADER.PRODUCT\_STORE\_ID,

### INTERNAL\_NAME,

### ORDER\_ITEM\_SHIP\_GROUP.FACILITY\_ID,

### ORDER\_HEADER.EXTERNAL\_ID,

### FACILITY.FACILITY\_TYPE\_ID,

### ORDER\_HISTORY.ORDER\_HISTORY\_ID,

### ORDER\_HEADER.ORDER\_ID,

### ORDER\_ITEM.ORDER\_ITEM\_SEQ\_ID,

### ORDER\_ITEM\_SHIP\_GROUP.SHIP\_GROUP\_SEQ\_ID;

### 

### 6 Newly Created Sales Orders and Payment Methods

Business Problem:  
Finance teams need to see new orders and their payment methods for reconciliation and fraud checks.

Fields to Retrieve:

*   ORDER\_ID
*   TOTAL\_AMOUNT
*   PAYMENT\_METHOD
*   Shopify Order ID (if applicable)

### SELECT

### order\_header.ORDER\_ID,

### max\_amount AS TOTAL\_AMOUNT,

### PAYMENT\_METHOD\_TYPE\_ID,

### PAYMENT\_METHOD\_ID,

### EXTERNAL\_ID AS Shopify\_Order\_ID

### FROM ORDER\_HEADER

### LEFT JOIN order\_payment\_preference

### ON ORDER\_HEADER.ORDER\_ID = order\_payment\_preference.ORDER\_ID

### WHERE order\_headeR.STATUS\_ID ="ORDER\_CREATED";

### 7 Payment Captured but Not Shipped

Business Problem:  
Finance teams want to ensure revenue is recognized properly. If payment is captured but no shipment has occurred, it warrants further review.

Fields to Retrieve:

*   ORDER\_ID
*   ORDER\_STATUS
*   PAYMENT\_STATUS
*   SHIPMENT\_STATUS

**SELECT**

**order\_header.ORDER\_ID,**

**order\_header.STATUS\_ID,**

**order\_payment\_preference.STATUS\_ID AS payment\_status,**

**SHIPMENT.STATUS\_ID AS shipment\_status**

**FROM order\_header**

**LEFT JOIN ORDER\_PAYMENT\_PREFERENCE**

**ON order\_header.ORDER\_ID = order\_payment\_preference.ORDER\_ID**

**LEFT JOIN shipment**

**ON order\_header.ORDER\_ID = shipment.PRIMARY\_ORDER\_ID;**

### 8 Orders Completed Hourly

Business Problem:  
Operations teams may want to see how orders complete across the day to schedule staffing.

Fields to Retrieve:

*   TOTAL ORDERS
*   HOUR

### SELECT

### count(\*) AS total\_orders,

### extract(hour FROM status\_datetime) AS hour

### FROM order\_status

### WHERE STATUS\_ID LIKE "ORDER\_COMPLETED"

### GROUP BY extract(hour FROM status\_datetime)

### ORDER BY extract(hour FROM status\_datetime);

### 9 BOPIS Orders Revenue (Last Year)

Business Problem:  
BOPIS (Buy Online, Pickup In Store) is a key retail strategy. Finance wants to know the revenue from BOPIS orders for the previous year.

Fields to Retrieve:

*   TOTAL ORDERS
*   TOTAL REVENUE

SELECT
count(*) AS total_bopis_orders_items,
sum(UNIT_PRICE * order_item.quantity) AS revenue

FROM order_item_ship_group_assoc

JOIN order_item_ship_group
ON order_item_ship_group_assoc.ORDER_ID = order_item_ship_group.ORDER_ID
AND order_item_ship_group_assoc.SHIP_GROUP_SEQ_ID = order_item_ship_group.SHIP_GROUP_SEQ_ID

JOIN order_item
ON order_item_ship_group_assoc.ORDER_ID = order_item.ORDER_ID
AND order_item_ship_group_assoc.ORDER_ITEM_SEQ_ID = order_item.ORDER_ITEM_SEQ_ID

JOIN order_header
ON Order_header.ORDER_ID = order_item.order_id

WHERE SHIPMENT_METHOD_TYPE_ID LIKE "STOREPICKUP" and SALES_CHANNEL_ENUM_ID="WEB_SALES_CHANNEL"
AND year(ORDER_DATE) = year(current_date)-1;

### 10 Canceled Orders (Last Month)

Business Problem:  
The merchandising team needs to know how many orders were canceled in the previous month and their reasons.

Fields to Retrieve:

*   TOTAL ORDERS
*   CANCELATION REASON

### SELECT

### count(DISTINCT ORDER\_ID) AS total\_orders,

### change\_reason

### FROM order\_status

### WHERE STATUS\_ID LIKE "ITEM\_CANCELLED"

### AND STATUS\_DATETIME BETWEEN "2026-05-01" AND "2026-05-31"

### GROUP BY CHANGE\_REASON;

### 11 Product Threshold Value

Business Problem The retailer has set a threshild value for products that are sold online, in order to avoid over selling.

Fields to Retrieve:

*   PRODUCT ID
*   THRESHOLD

**SELECT PRODUCT\_ID,MINIMUM\_STOCK from product\_facility ;**
