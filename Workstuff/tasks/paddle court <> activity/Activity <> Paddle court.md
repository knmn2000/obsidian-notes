
Migrations
```sql
ALTER TYPE oms."products_inventory_type_enum" ADD VALUE 'DAILY';
ALTER TYPE oms."products_inventory_type_enum" ADD VALUE 'HOURLY';
ALTER TYPE oms."products_inventory_type_enum" ADD VALUE 'MONTHLY';
ALTER TYPE oms."products_type_enum" ADD VALUE 'ACTIVITY';
```



---
admin dashboard fe -> catalogue

catalogue will first check the category, which we will get while onboarding a product
```
"category": {

"type": "ACTIVITIES"

}
```

then we will insert that data in products table, and also update product pricing table (here, there will be a foreign key towards products table (check TRD))

so in total 3 tables are read/written

---

for the inventory page
- user will fill a form with building id, start date, end date, and activity type
- based on that, we will GET the inventories, and the user will be able to update the inventory


