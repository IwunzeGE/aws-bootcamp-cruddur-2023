# Week 4 — Postgres and RDS

Although we'd be launching the RDS instance from the CLI, Lets take a walk on how to use the console too

![rds1](https://user-images.githubusercontent.com/110903886/227605795-55fca5b3-1d1f-43f8-9bb5-85e021fecf6f.png)

![rds2](https://user-images.githubusercontent.com/110903886/227605867-78c84c29-efa1-481a-a046-5cd0d13543d3.png)

![rds3](https://user-images.githubusercontent.com/110903886/227605897-6d9c32eb-30c0-4cd7-8db2-4e3f28125fea.png)

- Select the template - We'd use free tier although it restricts us access to some config options
- Setup database-1
- Set up password if you're not managing the credentials with AWS Secrets Manager. 
**Note: AWS Secrets Manager cost $1 to use.**

- Choose the right instance configuration that fits your need. Although we'd be limited to the db.t3.micro and db.t4g.micro free tier instance.

![rds instance config](https://user-images.githubusercontent.com/110903886/227606167-bad6e2b9-d292-4072-be25-db28138bb0f4.png)

- Allocate storage and enable autoscalinng
- Under the connectivity section, we'd allow public access although it's not the best in terms of secuirity. But there's always a trade cost with secuirity. Despite being public, the secuirity groups would serve as a check medium.

![rds4](https://user-images.githubusercontent.com/110903886/227606333-93ab1962-e542-4b1c-ad4d-1acb279087bd.png)

- With AWS it's okay to use the default vpc but with some other cloud provider, we might want to use another one.

![rds5](https://user-images.githubusercontent.com/110903886/227607338-28b071c5-1866-4ae1-ba12-1ed157402544.png)

- It is important to enable automated backups in production and enable deletion protection

![rds6](https://user-images.githubusercontent.com/110903886/227607602-e7338550-030a-429d-803c-fe40060445a6.png)
 

Now, lets provisioning an RDS instance with db named 'cruddur' from the CLI by running the code below.

```
aws rds create-db-instance \
  --db-instance-identifier cruddur-db-instance \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --engine-version  14.6 \
  --master-username root \
  --master-user-password huEE33z2Qvl383 \
  --allocated-storage 20 \
  --availability-zone ca-central-1a \
  --backup-retention-period 0 \
  --port 5432 \
  --no-multi-az \
  --db-name cruddur \
  --storage-type gp2 \
  --publicly-accessible \
  --storage-encrypted \
  --enable-performance-insights \
  --performance-insights-retention-period 7 \
  --no-deletion-protection
```
![cli code](https://user-images.githubusercontent.com/110903886/227614009-b1b3b1ef-3544-46f8-b133-8d28324fb514.png)

- Confirm from the console to see if it worked.

![cli code check](https://user-images.githubusercontent.com/110903886/227614048-f8024b3f-98b3-42f0-8f97-7b6a98c2abb3.png)

Connect to psql via the psql client cli tool remember to use the host flag to specific localhost
`psql -U postgres --host localhost`

![login postgres](https://user-images.githubusercontent.com/110903886/227617142-5c051ef5-66e9-4f22-bfa3-bcf6a9e213c7.png)


### Common PSQL commands:

```
\x on -- expanded display when looking at data
\q -- Quit PSQL
\l -- List all databases
\c database_name -- Connect to a specific database
\dt -- List all tables in the current database
\d table_name -- Describe a specific table
\du -- List all users and their roles
\dn -- List all schemas in the current database
CREATE DATABASE database_name; -- Create a new database
DROP DATABASE database_name; -- Delete a database
CREATE TABLE table_name (column1 datatype1, column2 datatype2, ...); -- Create a new table
DROP TABLE table_name; -- Delete a table
SELECT column1, column2, ... FROM table_name WHERE condition; -- Select data from a table
INSERT INTO table_name (column1, column2, ...) VALUES (value1, value2, ...); -- Insert data into a table
UPDATE table_name SET column1 = value1, column2 = value2, ... WHERE condition; -- Update data in a table
DELETE FROM table_name WHERE condition; -- Delete data from a table
```

### Import Script

We'll create a new SQL file called `schema.sql` and we'll place it in `backend-flask/db`

The command to import:

`psql cruddur < db/schema.sql -h localhost -U postgres`

### Add UUID Extension

We are going to have Postgres generate out UUIDs. We'll need to use an extension called:

```
CREATE EXTENSION "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```










