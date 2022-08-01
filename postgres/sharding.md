# psqlshard
This document covers the steps to create an postgresql sharded cluster with postgres_fdw inbuilt extension.

## Create Extension
Run it on main server (shard1)

```sql
CREATE EXTENSION postgres_fdw;
```

## Create table customers partitioned by registered date

```sql
CREATE TABLE customers (
  id INT NOT NULL,
  name VARCHAR(30) NOT NULL,
  registered DATE NOT NULL
)
PARTITION BY RANGE (registered);
```

## add partitions

```sql
CREATE TABLE customers_2021
    PARTITION OF customers
    FOR VALUES FROM ('2021-01-01') TO ('2022-01-01');
CREATE TABLE customers_2020
    PARTITION OF customers
    FOR VALUES FROM ('2020-01-01') TO ('2021-01-01');
```

## Insert some test values in shard1

```sql
INSERT INTO customers (id, name, registered) VALUES (1, 'James', '2020-05-01');
INSERT INTO customers (id, name, registered) VALUES (2, 'Mary', '2021-03-01');
```

## create table in shard2

```sql
CREATE TABLE customers_2019 (
    id INT NOT NULL,
    name VARCHAR(30) NOT NULL,
    registered DATE NOT NULL);
```

## Create Shard2 server in Shard1

```sql
CREATE SERVER shard2 FOREIGN DATA WRAPPER postgres_fdw OPTIONS (host '<hostaddr>',port '5000',dbname 'postgres')
```

## Add user to access it

```sql
CREATE USER MAPPING FOR postgres SERVER shard2 OPTIONS (user 'postgres', password '****');
```

## Create the FOREIGN TABLE in Shard1

```sql
CREATE FOREIGN TABLE customers_2019
PARTITION OF customers
FOR VALUES FROM ('2019-01-01') TO ('2020-01-01')
SERVER shard2;
```

## Insert some test values to verify shard

```sql
INSERT INTO customers (id, name, registered) VALUES (3, 'Robert', '2019-07-01');
INSERT INTO customers (id, name, registered) VALUES (4, 'Jennifer', '2019-11-01');
```
