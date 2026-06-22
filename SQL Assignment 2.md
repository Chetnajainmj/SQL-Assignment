# SQL Assignment - 2

### 1 Shipping Addresses for October 2023 Orders

Business Problem:  
Customer Service might need to verify addresses for orders placed or completed in October 2023. This helps ensure shipments are delivered correctly and prevents address-related issues.

Fields to Retrieve:

*   ORDER\_ID
*   PARTY\_ID (Customer ID)
*   CUSTOMER\_NAME (or FIRST\_NAME / LAST\_NAME)
*   STREET\_ADDRESS
*   CITY
*   STATE\_PROVINCE
*   POSTAL\_CODE
*   COUNTRY\_CODE
*   ORDER\_STATUS
*   ORDER\_DATE

select order\_header.ORDER\_ID,

ORDER\_ROLE.PARTY\_ID,

postal\_address.TO\_NAME,

postal\_address.ADDRESS1,

postal\_address.CITY,

postal\_address.STATE\_PROVINCE\_GEO\_ID,

postal\_address.POSTAL\_CODE,

postal\_address.COUNTRY\_GEO\_ID,

order\_header.status\_id,

order\_header.ORDER\_DATE from order\_header

join order\_contact\_mech on order\_header.ORDER\_ID=order\_contact\_mech.ORDER\_ID join postal\_address

on order\_contact\_mech.CONTACT\_MECH\_ID=postal\_address.CONTACT\_MECH\_ID

join order\_role on order\_header.ORDER\_ID=order\_role.order\_id

where CONTACT\_MECH\_PURPOSE\_TYPE\_ID like "SHIPPING\_LOCATION" and role\_type\_id like "SHIP\_TO\_CUSTOMER" and order\_header.order\_date>="2023-10-1" and order\_header.order\_date<="2023-10-31";

### 2 Orders from New York

Business Problem:  
Companies often want region-specific analysis to plan local marketing, staffing, or promotions in certain areas—here, specifically, New York.

Fields to Retrieve:

*   ORDER\_ID
*   CUSTOMER\_NAME
*   STREET\_ADDRESS (or shipping address detail)
*   CITY
*   STATE\_PROVINCE
*   POSTAL\_CODE
*   TOTAL\_AMOUNT
*   ORDER\_DATE
*   ORDER\_STATUS

select order\_header.ORDER\_ID,

postal\_address.TO\_NAME,

postal\_address.ADDRESS1,

postal\_address.CITY,

postal\_address.STATE\_PROVINCE\_GEO\_ID,

postal\_address.POSTAL\_CODE,

order\_header.GRAND\_TOTAL,

order\_header.status\_id,

order\_header.ORDER\_DATE from order\_header

join order\_contact\_mech on order\_header.ORDER\_ID=order\_contact\_mech.ORDER\_ID join postal\_address

on order\_contact\_mech.CONTACT\_MECH\_ID=postal\_address.CONTACT\_MECH\_ID

where CONTACT\_MECH\_PURPOSE\_TYPE\_ID like "SHIPPING\_LOCATION"

and city like "NewYork";

### 3\. Top-Selling Product in New York

Business Problem:  
Merchandising teams need to identify the best-selling product(s) in a specific region (New York) for targeted restocking or promotions.

Fields to Retrieve:

*   PRODUCT\_ID
*   INTERNAL\_NAME
*   TOTAL\_QUANTITY\_SOLD
*   CITY / STATE (within New York region)
*   REVENUE (optionally, total sales amount)

SELECT product.PRODUCT\_ID,

product.INTERNAL\_NAME,

sum(order\_item.quantity) as TOTAL\_QUANTITY\_SOLD,

postal\_address.CITY,

sum(order\_item.UNIT\_PRICE\*order\_item.quantity) as revenue

from order\_item right join product on order\_item.PRODUCT\_ID=product.PRODUCT\_ID

left join order\_contact\_mech on order\_contact\_mech.order\_ID=order\_item.ORDER\_ID

left join postal\_address on postal\_address.CONTACT\_MECH\_ID=order\_contact\_mech.CONTACT\_MECH\_ID

where postal\_address.city="New York" and CONTACT\_MECH\_PURPOSE\_TYPE\_ID="SHIPPING\_LOCATION"

group by product.PRODUCT\_ID,product.INTERNAL\_NAME,postal\_address.CITY order by TOTAL\_QUANTITY\_SOLD desc limit 1;

### 4 Store-Specific (Facility-Wise) Revenue

Business Problem:  
Different physical or online stores (facilities) may have varying levels of performance. The business wants to compare revenue across facilities for sales planning and budgeting.

Fields to Retrieve:

*   FACILITY\_ID
*   FACILITY\_NAME
*   TOTAL\_ORDERS
*   TOTAL\_REVENUE
*   DATE\_RANGE

select facility.FACILITY\_ID,

facility.FACILITY\_NAME,

count(distinct order\_header.order\_id) as TOTAL\_ORDERS,

sum(GRAND\_TOTAL)as TOTAL\_REVENUE,

concat(min(ORDER\_DATE),"-",max(ORDER\_DATE)) as date\_range from facility left join order\_item\_ship\_group ON facility.FACILITY\_ID=order\_item\_ship\_group.FACILITY\_ID

left join order\_header on order\_item\_ship\_group.ORDER\_ID=order\_header.ORDER\_ID group by facility.FACILITY\_ID,

facility.FACILITY\_NAME;

### 5 Lost and Damaged Inventory

Business Problem:  
Warehouse managers need to track “shrinkage” such as lost or damaged inventory to reconcile physical vs. system counts.

Fields to Retrieve:

*   INVENTORY\_ITEM\_ID
*   PRODUCT\_ID
*   FACILITY\_ID
*   QUANTITY\_LOST\_OR\_DAMAGED
*   REASON\_CODE (Lost, Damaged, Expired, etc.)
*   TRANSACTION\_DATE

SELECT inventory\_item.INVENTORY\_ITEM\_ID,

inventory\_item.PRODUCT\_ID,

inventory\_item.FACILITY\_ID,

abs(quantity\_on\_hand\_diff )as QUANTITY\_LOST\_OR\_DAMAGED,

reason\_enum\_id,

effective\_date from inventory\_item\_detail join inventory\_item on inventory\_item.INVENTORY\_ITEM\_ID=inventory\_item\_detail.INVENTORY\_ITEM\_ID

where REASON\_ENUM\_ID ="VAR\_DAMAGED" or REASON\_ENUM\_ID ="VAR\_LOST" GROUP BY inventory\_item.INVENTORY\_ITEM\_ID,

inventory\_item.PRODUCT\_ID,

inventory\_item.FACILITY\_ID,

effective\_date;

### 6 Low Stock or Out of Stock Items Report

Business Problem:  
Avoiding out-of-stock situations is critical. This report flags items that have fallen below a certain reorder threshold or have zero available stock.

Fields to Retrieve:

*   PRODUCT\_ID
*   PRODUCT\_NAME
*   FACILITY\_ID
*   QOH (Quantity on Hand)
*   ATP (Available to Promise)
*   REORDER\_THRESHOLD
*   DATE\_CHECKED

SELECT product.PRODUCT\_ID,

product.PRODUCT\_NAME,

product\_facility.FACILITY\_ID,

inventory\_item.QUANTITY\_ON\_HAND\_TOTAL,

inventory\_item.AVAILABLE\_TO\_PROMISE\_TOTAL,

product\_facility.REORDER\_POINT,

inventory\_item.LAST\_UPDATED\_STAMP from

inventory\_item join product\_facility on inventory\_item.INVENTORY\_ITEM\_ID=product\_facility.INVENTORY\_ITEM\_ID

join product on product\_facility.PRODUCT\_ID=product.PRODUCT\_ID where AVAILABLE\_TO\_PROMISE\_TOTAL<REORDER\_POINT or AVAILABLE\_TO\_PROMISE\_TOTAL<=0;

### 7 Retrieve the Current Facility (Physical or Virtual) of Open Orders

Business Problem:  
The business wants to know where open orders are currently assigned, whether in a physical store or a virtual facility (e.g., a distribution center or online fulfillment location).

Fields to Retrieve:

*   ORDER\_ID
*   ORDER\_STATUS
*   FACILITY\_ID
*   FACILITY\_NAME
*   FACILITY\_TYPE\_ID

select order\_header.ORDER\_ID,

order\_header.STATUS\_ID,

FACILITY.FACILITY\_ID,

FACILITY.FACILITY\_NAME,

FACILITY.FACILITY\_TYPE\_ID

FROM ORDER\_HEADER LEFT JOIN order\_item\_ship\_group ON ORDER\_HEADER.ORDER\_ID=order\_item\_ship\_group.ORDER\_ID

join facility on order\_item\_ship\_group.FACILITY\_ID=facility.FACILITY\_ID where order\_header.STATUS\_ID="ORDER\_CREATED"

OR order\_header.STATUS\_ID="ORDER\_APPROVED";

### 8 Items Where QOH and ATP Differ

Business Problem:  
Sometimes the Quantity on Hand (QOH) doesn’t match the Available to Promise (ATP) due to pending orders, reservations, or data discrepancies. This needs review for accurate fulfillment planning.

Fields to Retrieve:

*   PRODUCT\_ID
*   FACILITY\_ID
*   QOH (Quantity on Hand)
*   ATP (Available to Promise)
*   DIFFERENCE (QOH - ATP)

select PRODUCT\_ID,

FACILITY\_ID,

QUANTITY\_ON\_HAND\_TOTAL,

AVAILABLE\_TO\_PROMISE\_TOTAL,

QUANTITY\_ON\_HAND\_TOTAL - AVAILABLE\_TO\_PROMISE\_TOTAL as difference

from inventory\_item ;

### 9 Order Item Current Status Changed Date-Time

Business Problem:  
Operations teams need to audit when an order item’s status (e.g., from “Pending” to “Shipped”) was last changed, for shipment tracking or dispute resolution.

Fields to Retrieve:

*   ORDER\_ID
*   ORDER\_ITEM\_SEQ\_ID
*   CURRENT\_STATUS\_ID
*   STATUS\_CHANGE\_DATETIME
*   CHANGED\_BY

select ORDER\_ID,

ORDER\_ITEM\_SEQ\_ID,

shipment.SHIPMENT\_ID,

STATUS\_ID,

LAST\_MODIFIED\_DATE,

LAST\_MODIFIED\_BY\_USER\_LOGIN from shipment

join order\_shipment on shipment.shipment\_id=order\_shipment.shipment\_id

### 10 Total Orders by Sales Channel

Business Problem:  
Marketing and sales teams want to see how many orders come from each channel (e.g., web, mobile app, in-store POS, marketplace) to allocate resources effectively.

Fields to Retrieve:

*   SALES\_CHANNEL
*   TOTAL\_ORDERS
*   TOTAL\_REVENUE
*   REPORTING\_PERIOD

SELECT SALES\_CHANNEL\_ENUM\_ID,

count(\*) as TOTAL\_ORDERS,

sum(GRAND\_TOTAL) as TOTAL\_REVENUE,

max(order\_date) from order\_header group by SALES\_CHANNEL\_ENUM\_ID;