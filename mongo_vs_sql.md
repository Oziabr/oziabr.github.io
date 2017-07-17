# Implementing noSQL designs RDBMS

In this article we're designing **postgresql** DB in a **mongodb** way.

Please note: tables have no indexes, for simplicity's sake.

## Basics

Are described here https://www.mongodb.com/blog/post/6-rules-of-thumb-for-mongodb-schema-design-part-1

### One2Few

It is not only easy to implement

```sql
create table person (name varchar, addresses jsonb);
insert into person values ('Kate Monster', '[{ "street": "123 Sesame St", "city": "Anytown", "cc": "USA" }, { "street": "123 Avenue Q", "city": "New York", "cc": "USA" }]');
insert into person values ('Asha Terner', '[{ "street": "123 Avenue Q", "city": "New York", "cc": "USA" }]');
insert into person values ('Jon Gaucho', '[{ "street": "54 Asame St", "city": "Rome", "cc": "Italy" }]');
```

As an added bonus you'll have wide range of comparison operators, e.g. to check tenants

```sql
select name from person where addresses @> '[{ "street": "123 Avenue Q", "city": "New York", "cc": "USA" }]'::jsonb;
     name     
--------------
 Kate Monster
 Asha Terner
(2 rows)
```

### One2Many

Normally you don't want to do this, because this way you can't ensure consistency.

```sql
create table parts (partno uuid, name varchar, qty int, cost float, price float);
insert into parts values ('96b89add-4a03-4963-a145-2a079ab4cf83', '.4 grommet', 94, 0.94, 3.99);
insert into parts values ('d3a7cb2b-ddee-44d5-b439-8343ff9ac5a1', 'h-lever', 34, 1.40, 5.25);

create table products (name varchar, manufacturer varchar, catalog_number int, parts uuid[]);
insert into products values ('left-handed smoke shifter', 'Acme Corp', 1234, '{96b89add-4a03-4963-a145-2a079ab4cf83, d3a7cb2b-ddee-44d5-b439-8343ff9ac5a1}');
```

Getting data out will also became a bit trickier:

```sql
select parts.name from products, parts where partno = any(products.parts);
    name    
------------
 .4 grommet
 h-lever
(2 rows)
```

### One2Squillions





