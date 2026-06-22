**SQL Assignment -3**

### 1 Completed Sales Orders (Physical Items)

Business Problem:  
Merchants need to track only physical items (requiring shipping and fulfillment) for logistics and shipping-cost analysis.

Fields to Retrieve:

*   ORDER\_ID
*   ORDER\_ITEM\_SEQ\_ID
*   PRODUCT\_ID
*   PRODUCT\_TYPE\_ID
*   SALES\_CHANNEL\_ENUM\_ID
*   ORDER\_DATE
*   ENTRY\_DATE
*   STATUS\_ID
*   STATUS\_DATETIME
*   ORDER\_TYPE\_ID
*   PRODUCT\_STORE\_ID

SELECT order\_header.ORDER\_ID,

order\_item.ORDER\_ITEM\_SEQ\_ID,

order\_item.PRODUCT\_ID,

PRODUCT.PRODUCT\_TYPE\_ID,

ORDER\_HEADER.SALES\_CHANNEL\_ENUM\_ID,

order\_header.ORDER\_DATE,

order\_header.ENTRY\_DATE,

order\_item.STATUS\_ID,

order\_status.STATUS\_DATETIME,

ORDER\_TYPE\_ID,

PRODUCT\_STORE\_ID from order\_header left join order\_item on order\_header.order\_id=order\_item.ORDER\_ID

left join product on order\_item.PRODUCT\_ID=product.PRODUCT\_ID left join PRODUCT\_TYPE on product.PRODUCT\_type\_id=product\_type.PRODUCT\_TYPE\_ID

left join ORDER\_STATUS on ORDER\_ITEM.ORDER\_ITEM\_SEQ\_ID=order\_status.ORDER\_ITEM\_SEQ\_ID and order\_item.ORDER\_ID=order\_status.order\_id where product.is\_Variant="Y" and is\_physical="Y" and order\_item.STATUS\_ID = "ITEM\_COMPLETED";

### 2 Completed Return Items

Business Problem:  
Customer service and finance often need insights into returned items to manage refunds, replacements, and inventory restocking.

Fields to Retrieve:

*   RETURN\_ID
*   ORDER\_ID
*   PRODUCT\_STORE\_ID
*   STATUS\_DATETIME
*   ORDER\_NAME
*   FROM\_PARTY\_ID
*   RETURN\_DATE
*   ENTRY\_DATE
*   RETURN\_CHANNEL\_ENUM\_ID

SELECT return\_header.RETURN\_ID,

order\_header.ORDER\_ID,

order\_header.PRODUCT\_STORE\_ID,

return\_status.STATUS\_DATETIME,

order\_header.ORDER\_NAME,

return\_header.FROM\_PARTY\_ID,

return\_header.RETURN\_DATE,

return\_header.ENTRY\_DATE,

return\_header.RETURN\_CHANNEL\_ENUM\_ID

from return\_header left join return\_item on return\_header.RETURN\_ID=return\_item.RETURN\_ID

left join order\_header on return\_item.order\_id=order\_header.order\_id left join return\_status on return\_header.return\_id=return\_status.return\_id where return\_status.status\_id="RETURN\_COMPLETED" and return\_status.return\_item\_seq\_id is null;

### 3 Single-Return Orders (Last Month)

Business Problem:  
The mechandising team needs a list of orders that only have one return.

Fields to Retrieve:

*   PARTY\_ID
*   FIRST\_NAME

select PARTY\_ID,

FIRST\_NAME

from return\_header join return\_item on return\_header.return\_id=return\_item.return\_id join person on PARTY\_ID=FROM\_PARTY\_ID

where return\_date>="2026-05-01" and return\_date<="2026-05-31" group by order\_id having count(order\_id)=1;

### 4 Returns and Appeasements

Business Problem:  
The retailer needs the total amount of items, were returned as well as how many appeasements were issued.

Fields to Retrieve:

*   TOTAL RETURNS
*   RETURN $ TOTAL
*   TOTAL APPEASEMENTS
*   APPEASEMENTS $ TOTAL

select count(RETURN\_item.return\_id) as TOTAL\_RETURNS,

sum(RETURN\_PRICE) as RETURN\_TOTAL,

count(RETUrn\_adjustment\_id) as TOTAL\_APPEASEMENTS,

sum(amount) as APPEASEMENTS\_TOTAL

from return\_item left join return\_adjustment on return\_item.RETURN\_ID=return\_adjustment.RETURN\_ID;

### 5 Detailed Return Information

Business Problem:  
Certain teams need granular return data (reason, date, refund amount) for analyzing return rates, identifying recurring issues, or updating policies.

Fields to Retrieve:

*   RETURN\_ID
*   ENTRY\_DATE
*   RETURN\_ADJUSTMENT\_TYPE\_ID (refund type, store credit, etc.)
*   AMOUNT
*   COMMENTS
*   ORDER\_ID
*   ORDER\_DATE
*   RETURN\_DATE
*   PRODUCT\_STORE\_ID

select return\_header.RETURN\_ID,

return\_header.ENTRY\_DATE,

return\_adjustment.RETURN\_ADJUSTMENT\_TYPE\_ID,

AMOUNT,

COMMENTS,

return\_item.ORDER\_ID,

order\_header.ORDER\_DATE,

return\_header.RETURN\_DATE,

order\_header.PRODUCT\_STORE\_ID

from return\_header left join return\_adjustment on return\_adjustment.RETURN\_ID=return\_header.return\_id

left join return\_item on return\_header.RETURN\_ID=return\_item.return\_id left join order\_header on return\_item.order\_id=order\_header.order\_id;

### 6 Orders with Multiple Returns

Business Problem:  
Analyzing orders with multiple returns can identify potential fraud, chronic issues with certain items, or inconsistent shipping processes.

Fields to Retrieve:

*   ORDER\_ID
*   RETURN\_ID
*   RETURN\_DATE
*   RETURN\_REASON
*   RETURN\_QUANTITY

select ORDER\_ID,

return\_header.RETURN\_ID,

RETURN\_DATE,

RETURN\_REASON\_ID,

RETURN\_QUANTITY from return\_header left join return\_item on return\_header.return\_id=return\_item.return\_id;

### 7 Store with Most One-Day Shipped Orders (Last Month)

Business Problem:  
Identify which facility (store) handled the highest volume of “one-day shipping” orders in the previous month, useful for operational benchmarking.

Fields to Retrieve:

*   FACILITY\_ID
*   FACILITY\_NAME
*   TOTAL\_ONE\_DAY\_SHIP\_ORDERS
*   REPORTING\_PERIOD

select facility.FACILITY\_ID,

FACILITY\_NAME,

count(\*) as TOTAL\_ONE\_DAY\_SHIP\_ORDERS,

"MAY 2026" as REPORTING\_PERIOD

from order\_item\_ship\_group right join facility on order\_item\_ship\_group.FACILITY\_ID=facility.FACILITY\_ID where SHIPMENT\_METHOD\_TYPE\_ID="NEXT\_DAY" group by facility.FACILITY\_ID,

FACILITY\_NAME ;

### 8 List of Warehouse Pickers

Business Problem:  
Warehouse managers need a list of employees responsible for picking and packing orders to manage shifts, productivity, and training needs.

Fields to Retrieve:

*   PARTY\_ID (or Employee ID)
*   NAME (First/Last)
*   ROLE\_TYPE\_ID (e.g., “WAREHOUSE\_PICKER”)
*   FACILITY\_ID (assigned warehouse)
*   STATUS (active or inactive employee)

select facility\_party.PARTY\_ID,

concat(person.first\_name," ",person.last\_name) as NAME,

facility\_party.ROLE\_TYPE\_ID,

facility\_party.FACILITY\_ID,

case when THRU\_DATE is null then "active" else "inactive" end as STATUS from facility\_party join person on facility\_party.PARTY\_ID=person.PARTY\_ID

where role\_type\_id="WAREHOUSE\_PICKER";

### 9 Total Facilities That Sell the Product

Business Problem:  
Retailers want to see how many (and which) facilities (stores, warehouses, virtual sites) currently offer a product for sale.

Fields to Retrieve:

*   PRODUCT\_ID
*   PRODUCT\_NAME (or INTERNAL\_NAME)
*   FACILITY\_COUNT (number of facilities selling the product)
*   (Optionally) a list of FACILITY\_IDs if more detail is needed

select product.PRODUCT\_ID,

PRODUCT\_NAME,

count(\*) as FACILITY\_COUNT from product\_facility join product on product\_facility.PRODUCT\_ID=product.PRODUCT\_ID group by PRODUCT\_ID,

PRODUCT\_NAME;

### 10 Total Items in Various Virtual Facilities

Business Problem:  
Retailers need to study the relation of inventory levels of products to the type of facility it's stored at. Retrieve all inventory levels for products at locations and include the facility type Id. Do not retrieve facilities that are of type Virtual.

Fields to Retrieve:

*   PRODUCT\_ID
*   FACILITY\_ID
*   FACILITY\_TYPE\_ID
*   QOH (Quantity on Hand)
*   ATP (Available to Promise)

select PRODUCT\_ID,

facility.FACILITY\_ID,

FACILITY\_TYPE\_ID,

QUANTITY\_ON\_HAND\_TOTAL,

AVAILABLE\_TO\_PROMISE\_TOTAL from inventory\_item join facility on inventory\_item.FACILITY\_ID=facility.FACILITY\_ID;

### 11 Transfer Orders Without Inventory Reservation

Business Problem:  
When transferring stock between facilities, the system should reserve inventory. If it isn’t reserved, the transfer may fail or oversell.

Fields to Retrieve:

*   TRANSFER\_ORDER\_ID
*   FROM\_FACILITY\_ID
*   TO\_FACILITY\_ID
*   PRODUCT\_ID
*   REQUESTED\_QUANTITY
*   RESERVED\_QUANTITY
*   TRANSFER\_DATE
*   STATUS

select order\_header.ORDER\_ID,

order\_item\_ship\_group.FACILITY\_ID as FROM\_FACILITY\_ID,

ORDER\_FACILITY\_ID as TO\_FACILITY\_ID,

PRODUCT\_ID,

order\_item.QUANTITY as REQUESTED\_QUANTITY,

order\_item\_ship\_grp\_inv\_res.QUANTITY as RESERVED\_QUANTITY,

order\_DATE,

order\_item.STATUS\_id

from order\_header join order\_item on order\_header.ORDER\_ID=order\_item.ORDER\_ID

join order\_item\_ship\_group on order\_item\_ship\_group.ORDER\_ID=order\_item.ORDER\_ID

join order\_item\_ship\_grp\_inv\_res on order\_header.ORDER\_ID=order\_item\_ship\_grp\_inv\_res.ORDER\_ID

where ORDER\_TYPE\_ID="TRANSFER\_ORDER";

### 12 Orders Without Picklist

Business Problem:  
A picklist is necessary for warehouse staff to gather items. Orders missing a picklist might be delayed and need attention.

Fields to Retrieve:

*   ORDER\_ID
*   ORDER\_DATE
*   ORDER\_STATUS
*   FACILITY\_ID
*   DURATION (How long has the order been assigned at the facility)

select order\_header.ORDER\_ID,

ORDER\_DATE,

STATUS\_ID,

FACILITY\_ID,

CREATED\_DATETIME - current\_timestamp from order\_item\_ship\_grp\_inv\_res as oisgir join order\_header on oisgir.ORDER\_ID=order\_header.ORDER\_ID

join order\_item\_ship\_group on oisgir.ORDER\_ID=order\_item\_ship\_group.ORDER\_ID and oisgir.SHIP\_GROUP\_SEQ\_ID=order\_item\_ship\_group.SHIP\_GROUP\_SEQ\_ID

where not exists (select 1 from shipment where primary\_order\_id=oisgir.ORDER\_ID);