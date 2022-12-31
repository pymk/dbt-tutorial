# Set up data for Postgres

The [current dbt tutorial uses BigQuery](https://docs.getdbt.com/docs/get-started/getting-started-dbt-core), but I wanted to test things locally instead.

So this is the guide to help set up the same `jaffle_shop` database using Postgres.

Note: Ensure you are in the project directory (i.e. `~/../dbt-tutorial/jaffle_shop`).

---

First, install [Postgres app](https://postgresapp.com).

In terminal, login to Postgres as sudo:

`psql -U postgres`

The command to list of current databases is `\l`.

Let's, create a user and database for `jaffle_shop`. In my case, the user is `dbt_test` (password: `dbt_test`) and the database is called `jaffle_shop`.

```sql
CREATE USER dbt_test WITH PASSWORD 'adb_test';
CREATE DATABASE jaffle_shop WITH OWNER 'dbt_test';
```

Using `\l`, we can see the new database is created.

Now we can exist sudo with `\q`

Next, let's login as the user we just created:

```sql
psql -U dbt_test jaffle_shop
```

Let's list all available tables (should be none) with `\dt`.

Now, we can create the tables (schema is based on the CSV files in the `data/` folder).

```sql
CREATE TABLE customers
 (id INTEGER, first_name TEXT, last_name TEXT, email TEXT);

CREATE TABLE orders
 (id INTEGER, user_id INTEGER, order_date DATE, status TEXT);

CREATE TABLE payments
 (id INTEGER, order_id INTEGER, payment_method TEXT, amount BigInt);
```

We can run `\dt` again to see a list of newly created tables.

Now we can copy the content of the CSV files into these tables:

```sql
\copy customers(id,first_name,last_name,email) FROM './data/raw_customers.csv' WITH DELIMITER ',' CSV header;

\copy orders(id,user_id,order_date,status) FROM './data/raw_orders.csv' WITH DELIMITER ',' CSV header;

\copy payments(id,order_id,payment_method,amount) FROM './data/raw_payments.csv' WITH DELIMITER ',' CSV header;
```

And a quick sanity check to ensure we have the data:

```sql
SELECT * FROM customers ORDER BY id ASC LIMIT 10;
SELECT * FROM orders ORDER BY id ASC LIMIT 10;
SELECT * FROM payments ORDER BY id ASC LIMIT 10;
```

And that's it. We can quit at anytime with `\q`.

---

The configuration for `~/.dbt/profiles.yml` looks like this:

```yaml
jaffle_shop:
  outputs:    
    dev:
        type: postgres
        threads: 4
        host: localhost
        port: 5432
        user: dbt_test
        pass: dbt_test
        dbname: jaffle_shop
        schema: public
  
  target: dev
```
